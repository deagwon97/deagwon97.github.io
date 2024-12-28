---
layout: post
thumbnail: 992707c2-23ce-4df8-ad30-3bb5229c4e58
title: "Start: Downloading VS Code Server Too Slow!"
createdAt: 2023-07-26 11:20:35.048000
updatedAt: 2023-07-26 11:20:35.048000
category: "기타"
---

필자는 대부분의 개발을 remote server → remote container 에서 수행한다.  VS code의 remote-ssh 익스텐션과 remote-container 익스텐션을 활용하면, 이런 작업을 대부분 자동화 할 수 있다.

 예를 들어 미리 실행되고 있는 container내부에 VSCode로 접속하기 위해서는 이 Container에  VS code Server를 설치해야한다. 이런 일련의 설치과정은 remote-container라는 익스텐션이 자동으로 수행한다.

### 문제 상황
하지만, 가끔 아래와 같이 6d9b74a70ca9c7733b29f0456fd8195364076dda 라는 파일을 다운받을 때, 시간이 매우 오래 걸리기도 한다.

``````bash
.
.
.
.
[5639 ms] 
[5639 ms] 
[5640 ms] Start: Run in container: test -d /root/.vscode-server/bin/6d9b74a70ca9c7733b29f0456fd8195364076dda
[5655 ms] 
[5655 ms] 
[5655 ms] Exit code 1
[5655 ms] Start: Run in container: test -d /vscode/vscode-server/bin/linux-x64/6d9b74a70ca9c7733b29f0456fd8195364076dda
[5670 ms] 
[5670 ms] 
[5670 ms] Exit code 1
[5670 ms] Installing VS Code Server for commit 6d9b74a70ca9c7733b29f0456fd8195364076dda
[5671 ms] Start: Downloading VS Code Server
[5671 ms] 6d9b74a70ca9c7733b29f0456fd8195364076dda linux-x64 stable
``````

하루 자고 일어나서 다시 시도해보면, 잘 동작하기 때문에 MS의 사소한 버그라고 생각하고 그날은 개발을 쉬었다.

하지만, 급하게 개발을 해야할 때는 꽤나 곤란한 상황이 발생하기 때문에 미리 해결책을 찾아보았다.

필자가 이 문제를 직면한 날로부터 28일 전에 똑같은 문제를 겪은 분이 있었다.


<img alt="image" src="/images/992707c2-23ce-4df8-ad30-3bb5229c4e58"/>


[https://github.com/microsoft/vscode-remote-release/issues/6948](https://github.com/microsoft/vscode-remote-release/issues/6948)

한국에서 이런 일이 종종 발생하는 것으로 보인다. 이슈로 등록되어 있지만, 마땅한 해결책은 찾을 수가 없었다.

### 해결책(임시)
미봉책으로 remote 서버의 vscode-server 파일을 container에 복사하여 접속한다.

참고로 필자는 dockerfile, build context, docker-compose.yml 파일 경로는 다음처럼 설정한다.

``````
/home/user/srv
└── project
    ├── .devcontainer
    │   ├── devcontainer.json
    │   └── docker-compose.yml
		├── .gitignore
		├── .git
    │   └── ...
    │  
    ├── .env
    ├── Dockerfile
    ├── README.md
    └── src
        └── project source files ..
``````

이제 remote 서버의 vscode-server 파일을 복사하여 project 폴더 아래에 둔다. 

``````bash
sudo cp -r /home/user/.vscode-server/bin /home/user/srv/project/.vscode-server/bin
``````

다음처럼 .vscode-server 폴더가 생겼다.

``````
/home/user/srv
└── project
    ├── .devcontainer
    │   ├── devcontainer.json
    │   └── docker-compose.yml
		├── .gitignore
		├── .git
    │   └── ...
    │  
    ├── .vscode-server
    │   └── bin
    │       ├── 3b889b090b5ad5793f524b5d1d39fda662b96a2a
    │       │   ├── LICENSE
    │       │   ├── bin
    │       │   │   ├── code-server
    │  
    ├── .env
    ├── Dockerfile
    ├── README.md
    └── src
        └── project source files ..
``````

이제 기존의 Dockerfile를 수정하여 ``/root/.vscode-server/bin`` 경로에 ``./.vscode-server/bin`` 를 복사한다. (만약 root user가 아니라 다른 user를 사용한다면 경로를 바꿔야한다.)

``````docker

FROM base-image

.
.
.

 
COPY ./.vscode-server/bin /root/.vscode-server/bin

.
.
``````

이후 다시 이미지를 빌드하고  Remote-Containers: Open Folder in Container.. 로 devcontainer에 접속하면


<img alt="image" src="/images/8fa9cdca-c6aa-4561-ae68-b7e3ac947cf0"/>


``````bash
[181319 ms] 
[181319 ms] 
[181409 ms] Start: Run in Host: docker inspect --type container 0ed74ff3f7cfb1a59a27f7b6e9acdf9d6f17df136306d490db35c63701ed2048
[181502 ms] Start: Run in Host: /home/-/.vscode-server/bin/6d9b74a70ca9c7733b29f0456fd8195364076dda/node /home/-/.vscode-remote-containers/dist/dev-containers-cli-0.241.3/dist/spec-node/devContainersSpecCLI.js read-configuration --workspace-folder /home/ubuntu/srv/- --log-level debug --log-format json --config /home/ubuntu/srv/-/.devcontainer/devcontainer.json --mount-workspace-git-root true
[181775 ms] Start: Run in Host: docker-compose version --short
[181844 ms] Start: Inspecting container
[181844 ms] Start: Run in Host: docker inspect --type container 0ed74ff3f7cfb1a59a27f7b6e9acdf9d6f17df136306d490db35c63701ed2048
.
.
.[187728 ms] [13:25:17] Renamed to /root/.vscode-server/extensions/esbenp.prettier-vscode-9.5.0
[187736 ms] [13:25:17] Extracting completed. esbenp.prettier-vscode
[187740 ms] [13:25:17] Extension installed successfully: esbenp.prettier-vscode
[187744 ms] Extension 'esbenp.prettier-vscode' v9.5.0 was successfully installed.
[391533 ms] Start: Run in container: cat /proc/951/environ
``````

성공적으로 접속할 수 있다! 급할 때는 이렇게 vscode-server파일을 복사하여 사용할 수 있다. 하지만, 근본적인 해결책은 아니다.

### reference
- https://github.com/microsoft/vscode-remote-release/issues/6948
