---
layout: post
thumbnail: ab739486-6263-4eff-95e5-80a76c6b1c15
title: "[Linux] exit 은 절대로 /etc/profile.d  속에 있으면 안 된다."
createdAt: 2023-07-26 11:24:27.737000
updatedAt: 2023-07-26 11:24:27.737000
category: "리눅스"
---

<img alt="image" src="/images/ab739486-6263-4eff-95e5-80a76c6b1c15"/>
 
 작업을 마무리하고 테스트를 위해서 서버에 다시 접속하려고 하는데, 다음과 같은 문제를 마주했다.

``````bash
user@local ~% ssh server
user@local ~%#...?
``````

지금까지 잘 들어갈 수 있었던 서버가 갑자기 막혔다. 하지만, 뭔가 이상했다. 보통 서버가 죽거나, 인증 상의 오류가 있다면 터미널이 pending 되다가 어떤 메시지를 띄워준다. 하지만, 위의 예시처럼, 서버에 접속하자마자 로컬로 돌아왔다. 마치 서버에 들어가자마자 ``exit`` 커맨드를 실행한 것처럼 동작했다.

### 무엇을 하고 있었나?

 서버는 여러 명의 사용자가 상시로 접속한다. 이 사용자 중 특정 사용자에게 개인화된 배너를 보여주는 기능을 만들고 있었다.

### 어떤 방법을 선택했는가? - /etc/profile

 처음에는 motd를 알아보았다. 하지만, CentOS의 MOTD는 정적이기 때문에 모든 사용자에게 동일한 배너를 띄운다. 사용자 별로 개인화된 메시지를 보낼 수는 없었다.

이때, 모든 사용자가 로그인할 때, 항상 ``/etc/profile`` 스크립트가 실행되며, 이 스크립트는 ``/etc/profile.d`` 폴더 아래에 있는 모든 ``*.sh`` 파일을 실행한다는 것을 알게되었다. 이를 바탕으로  ``/etc/profile.d`` 폴더 아래에 다음과 같은 ``message.sh`` 파일을 추가해 개인화된 배너를 만들었다. **여기서 문제가 발생했다.** 아래 코드를 살펴보자.

``````bash
#!/bin/bash
user_list="user1 user2 user3 user4 ..."
user_name=``whoami``

is_in=0
for user in $user_list
do
    if [ $user = $user_name ]
    then
        is_in=1
        break
    fi
done
if [ $is_in = 1 ]
then
    exit
fi

echo ""
echo "안녕하세요 $user_name 님 반갑습니다."
echo "~~를 제출하셔야합니다. ~으로 ~을 보내주세요."
echo ""
``````

### 문제의 원인

``/etc/profile`` 의 동작 방식을 알고 있는 사람들은 위 스크립트를 보고 깜짝 놀랐을 것이다. 

``````bash
if [ $is_in = 1 ]
then
    exit
fi
``````

 만약 ``/etc/profile`` 스크립트가 ``/etc/profile.d/*.sh`` 파일들을 실행할 때, 개별 프로세스를 만들어서 실행 했다면, 위의 exit 구문은 문제가 없다. ``user_name``이 ``user_list`` 에 들어있지 않다면 해당 프로세스를 종료하고, 그렇지 않으면 아래 ``echo`` 를 실행한다.

하지만 ``/etc/profile.d/*.sh`` 파일을 실행시키는 ``/etc/profile`` 는 새로운 프로세스를 생성하지 않는다.  파일속 문자열을 읽은 후 그대로 자신의 프로세스에서 코드를 수행한다.

``````bash
for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then 
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done
``````

쉬운 예시를 통해서 살펴보자. 우선 [caller.sh](http://caller.sh)와 [callee.sh](http://callee.sh) 파일을 작성한다.

``````bash
.
├── callee.sh
└── caller.sh
``````

``````bash
#!/bin/bash
# callee.sh

echo "I'm callee"

exit
``````

``````bash
#!/bin/bash
# caller.sh

echo "I'm caller"

sh ./callee.sh

echo "after call"
``````

이제 ``caller.sh`` 파일을 실행한다면 어떤 결과가 나올까?

``````bash
$ sh ./caller.sh 
I'm caller
I'm callee
after call
``````

caller는 새로운 프로세스를 만들어서 callee를 실행했고, callee가 끝난 후(``exit``), ``echo "after call"`` 라인을 실행했다. 

이제 ``/etc/profile`` 처럼 바꿔보자.

``````bash
#!/bin/bash
# caller.sh

echo "I'm caller!"

**callee=./callee.sh
. "$callee"**

echo "after call"
``````

``````bash
#!/bin/bash
# caller.sh

echo "I'm callee"

exit
``````

``caller.sh`` 파일을 실행하면 다음과 같은 결과를 얻는다.

``````bash
$ sh ./caller.sh 
I'm caller
I'm callee
``````

``echo "after call"`` 이 실행되지 않고 caller가 종료되었다! caller는 callee를 새로운 프로세스에서 실행한 것이 아니다. callee일 읽어서 직접 실행했다. 이러면 ``exit``은 callee 프로세스를 종료한다는 뜻이 아닌 caller를 종료하는 명령이 된다.

자 이제 다시 필자가 짠 코드를 확인해 보자.

``````bash
#!/bin/bash
user_list="user1 user2 user3 user4 ..."
user_name=``whoami``

is_in=0
for user in $user_list
do
    if [ $user = $user_name ]
    then
        is_in=1
        break
    fi
done
if [ $is_in = 1 ]
then
    exit
fi

echo ""
echo "안녕하세요 $user_name 님 반갑습니다."
echo "~~를 제출하셔야합니다. ~으로 ~을 보내주세요."
echo ""
``````

그렇다! 필자가 작성한 코드는 ``user_list``에 속하지 않는 사용자는 로그인하는 즉시 ``exit``하는 코드이다! 그리고 저 ``user_list`` 속에는 ``root``도 없었고, ``sudo`` 권한을 가진 사용자가 아무도 없었다. 

 ``/etc/profile.d/message.sh`` 파일을 지울 방법을 찾아봤다. 하지만, 모든 ``root`` 터미널을 종료한 상태였고, ``sudo`` 권한이 없는 사용자가 ``message.sh`` 파일에 접근하는 법 따위는 존재하지 않았다.

 이 운영체제가 보호하는 한, ``root`` 권한 없이 ``message.sh`` 파일을 지울 방법이 생각나지 않았다.

 급하게 USB를 구해서 Ubuntu의 라이브 USB를 만들었다. 그리고 모니터, 키보드, usb를 들고 서버실에 들어갔다. BIOS 모드에서 부팅 디스크를 USB로 바꾼 후, 부팅하면 데이터는 그대로 둔 상태에서 Ubuntu가 부팅된다. 이제 볼륨을 마운트하고, ``root`` 권한으로 ``/etc/profile.d/message.sh`` 파일을 제거했다.

### /etc/profile 왜 이렇게 동작할까?

``/etc/profile`` 는 ``profile.d`` 폴더 속 스크립트를 읽어와, Linux 시스템의 환경 및 기타 커스텀 설정을 수행하는 도구이다.  스크립트 들의 실행결과가 유지돼야 하기 때문에, ``profile``은 새로운 프로세스를 생성하지 않고, 폴더 속 코드를 읽어와서 직접 실행한다.

### 다른 해결방법 1

다음날 찾아보니 ``/etc/profile.d`` 속 스크립트를 실행하지 않고 로그인 하는 방법이 있다고 한다.

``````bash
ssh -t root@host bash --noprofile
# or
su root bash  --noprofile
``````

하지만 이 방법은 root 계정으로 ssh 로그인이 가능할 때, 쓸 수 있는 반쪽짜리 방법이다. 

### 다른 해결방법 2
 centos에서도 안전모드로 들어가는 방법이 존재한다. 다른 부팅 usb를 구할 필요 없이, 노드에 키보드와 모니터만 연결할 수 있다면, 안전모드로 들어가 message.sh를 지울 수 있다. 

### 반성

 필자는 ``/etc/profile.d`` 속에 ``message.sh``를 추가할 때, ``profile`` 이 어떤 방식으로 ``profile.d``를 읽는지 알아보지 않았다. 이 때문에 대형 사고를 쳤다. 서버의 ``/etc`` 폴더를 건드리는 작업은 정말 신중을 가해야 한다. 또한 새로운 도구를 사용할 때, 이 **도구가 왜 만들어졌고**, **어떻게 동작하는 지**를 충분히 이해하고 사용해야 한다.


## Reference

- [https://askubuntu.com/questions/63741/can-i-ssh-into-my-account-without-invoking-profile](https://askubuntu.com/questions/63741/can-i-ssh-into-my-account-without-invoking-profile)
- [https://serverfault.com/questions/434321/when-are-scripts-inside-etc-profile-d-executed](https://serverfault.com/questions/434321/when-are-scripts-inside-etc-profile-d-executed)
- [https://eng.libretexts.org/Bookshelves/Computer_Science/Operating_Systems/Linux_-_The_Penguin_Marches_On_(McClanahan)/02%3A_User_Group_Administration/5.03%3A_System_Wide_User_Profiles/5.03.2_System_Wide_User_Profiles%3A_The_etc-profile.d_Directory](https://eng.libretexts.org/Bookshelves/Computer_Science/Operating_Systems/Linux_-_The_Penguin_Marches_On_(McClanahan)/02%3A_User_Group_Administration/5.03%3A_System_Wide_User_Profiles/5.03.2_System_Wide_User_Profiles%3A_The_etc-profile.d_Directory)
- [https://forums.fedoraforum.org/showthread.php?317937-Never-put-exit-in-a-etc-profile-d-script](https://forums.fedoraforum.org/showthread.php?317937-Never-put-exit-in-a-etc-profile-d-script)
- [https://bugzilla.redhat.com/show_bug.cgi?id=739991](https://bugzilla.redhat.com/show_bug.cgi?id=739991)
