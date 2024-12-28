---
layout: post
thumbnail: 064595aa-143e-4729-973e-71a70beb460a
title: "[네트워크]외부에서 Private Network에 접속할 방법이 없을까?  Reverse SSH Port Forwarding을 통해 방법을 찾아보자."
createdAt: 2023-07-26 10:41:21.316000
updatedAt: 2023-07-26 10:41:21.316000
category: "기타"
---


<img alt="image" src="/images/b09d16b6-55d5-4a00-abca-1e9c168a8d4f"/>

 회사 데스크탑을 집에서 SSH로 접근할 수 있다면 집에서도 재택을 더 효율적으로 할 수 있지 않을까 하는 생각이 들었다.(?)
 
 하지만 외부에서 사내 데스크탑에 접속하는 것은 쉽지 않다. 공유 오피스 관리팀에 포트 포워딩을 요청 해야 하며, 중간에 공유기가 있다면 그 공유기에도 추가 설정을 해야하고, 따로 요금을 지불해야 할 수도 있다.


 전에 누군가 Reverse SSH와 개인 서버만 있다면 이 문제를 쉽게 해결할 수 있다고 했던 말이 생각나 찾아봤다.
 

<img alt="image" src="/images/f8dbf960-2533-4eac-a44e-c6459d81c2e1"/>

### Prerequirements
- Company Desktop
  - ssh 서버가 열려있는 사내 컴퓨터.
  - 여기서는 22번 포트가 열려있다고 가정.
  - 외부 네트워크에 접속하는 것은 가능.
  - 외부 네트워크가 사내 데스크탑의 특벙 포트에 접속하는 것은 불가능.
- Cloud Server
  - ssh 서버가 열려있는 개인 클라우드 서버
  - 사내 망과 집 모두 이 클라우드 서버에 접속가능


<img alt="image" src="/images/064595aa-143e-4729-973e-71a70beb460a"/>

### 1. 사내 데스크탑으로 클라우드 서버에 접속
 
RemoteForward 옵션을 통해서 Cloud Server가 Company Desktop으로 reverse ssh 접속이 가능하도록 설정한다. 

``````txt
# Company Desktop
# ~/.ssh/config

Host cloud_server
    HostName <cloud_server_ip>
    Port <cloud_server_ssh_port>
    User <cluoud_user>
    IdentityFile <cloud_ssh_key_path>
    RemoteForward 5555 localhost:22
``````

``RemoteForward 5555 localhost:22`` 라인은 Cloud 의 5555가 Company Desktop 의 localhost:22에 Reverse Port Forwaring한다는 의미이다.

nohup으로 background에서 실행되도록 설정하였다. systemctl에 데몬으로 등록하여 Company Desktop이 켜져있을 때는 항상 접근 가능하도록 할 예정이다.

``````bash
>> nohup ssh cloud_server & 
``````
### 2. & 3. ProxyJump를 통해 클라우드를 경유하여 사내 데스크탑 접속
정말 간단하다. Home의 ssh config파일에 클라우드 서버의 ssh 접속 정보와, company_desktop의 접속 정보를 입력하고 ``PorxyJump``를 추가하면 된다.
``````
# Home
# ~/.ssh/config
Host cloud_server
	HostName  <cloud_server_ip>
	Port <cloud_server_port>
	User <cloud_server_user>
	IdentityFile <cloud_server_key_path>
	
Host company_desktop
	HostName localhost
	Port 5555
	User company_desktop_user
	ProxyJump cloud_server
``````

이제 집에서 Clode_server를 경유하여 Company desktop에 접근할 수 있다.

``````
# Home
>> ssh company_desktop
``````

### 4. VS code로 접근 테스트

ssh코드는 remote - ssh 플러그인을 통해서 해당 서버에 쉽게 접근이 가능하다.

**1. Remote-SSH 실행**

<img alt="image" src="/images/376be7bb-7664-4ce8-8df8-2cbcdc1214fa"/>

**2. company_desktop 선택**

<img alt="image" src="/images/947bb6c7-86f5-4b29-9c67-5d6712035361"/>

**3. Cloud Server SSH 암호 입력**

<img alt="image" src="/images/b9baa02b-e171-4fa6-a771-0f6b8d8dee7b"/>

**4. Company Desktop SSH 암호 입력**

<img alt="image" src="/images/f5480200-101a-4f5f-a408-7de129d72462"/>

**5. 성공!**

<img alt="image" src="/images/d3d46c44-a728-4f7d-9dcc-d0c5a31a6616"/>

### Reference
- https://blog.devolutions.net/2017/03/what-is-reverse-ssh-port-forwarding/
- https://lovethepenguin.com/how-to-ssh-keys-and-proxyjump-entries-caaec72e7e20
- https://system-monitoring.readthedocs.io/en/latest/ssh.html
