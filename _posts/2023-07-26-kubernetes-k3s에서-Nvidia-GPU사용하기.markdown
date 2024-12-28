---
layout: post
thumbnail: 9a468452-2fe2-4d76-bbe3-5bd95053ff88
title: "[kubernetes] k3s에서 Nvidia GPU사용하기"
createdAt: 2023-07-26 11:21:54.943000
updatedAt: 2023-07-26 11:21:54.943000
category: "쿠버네티스"
---



 운영중인 k3s 클러스터에는 상당히 오래된 gtx 1050 라는 nvidia gpu가 달려있다. VRAM이 무려 4GB 로 아주 작고 귀엽지만, 가벼운 딥러닝 모델을 추론용으로 사용할 수 있기 때문에 이번 기회에 이 gpu를 k3s 클러스터에 연결하고자 한다.

<img alt="image" src="/images/9a468452-2fe2-4d76-bbe3-5bd95053ff88"/>
[Image From: [https://developer.nvidia.com/blog/announcing-containerd-support-for-the-nvidia-gpu-operator/](https://developer.nvidia.com/blog/announcing-containerd-support-for-the-nvidia-gpu-operator/)]

 ### 1. nvidia gpu 확인

 내장 그래픽 카드로 UHD Graphics 620과  Nvidia GPU인 GTX 1050 Mobile 을  가지고 있다. 우선 lshw를 통해서 운영체제가 GPU를 인식하고 있는지 확인한다.

``````bash
$ sudo lshw -C display
``````

``````bash
*-display                 
       description: VGA compatible controller
       product: UHD Graphics 620
       vendor: Intel Corporation
       physical id: 2
       bus info: pci@0000:00:02.0
       logical name: /dev/fb0
       version: 07
       width: 64 bits
       clock: 33MHz
       capabilities: pcxxxxxxxxx
       configuration: depth=xx driver=xx latency=0 resolution=1920,1080
       resources: ixxxxxxxxx
  *-display
       description: 3D controller
       product: GP107M [**GeForce GTX 1050 Mobile**]
       vendor: **NVIDIA** Corporation
       physical id: 0
       bus info: pci@0000:03:00.0
       version: a1
       width: 64 bits
       clock: 33MHz
       capabilities: pxxxxxx
       configuration: driver=nvidia latency=0
       resources: irq:1xxxxxxxxxxxx
``````

### 2. nvidia-driver 설치

이제 nvidia-driver를 설치한다. 현재 시점에서 사용할 수 있는 nvidia-driver는 다음 명령어를 통해 확인할 수 있다.

``````bash
$ apt search nvidia-driver
``````

``````bash
# graphics display 이외의 기능을 담은 드라이버 패키지(커널 드라이버, cuda 드라이버, 유틸)
nvidia-headless-510/jammy-updates,jammy-security 510.85.02-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage
# graphics display 이외의 기능을 담은 드라이버 패키지(커널 드라이버, cuda 드라이버, 유틸)
nvidia-headless-510-server/jammy-updates,jammy-security 510.85.02-0ubuntu0.22.04.1 amd64
  NVIDIA headless metapackage

# 모든 드라이버 패키지(커널 드라이버, 2d/3d xorg 드라이버, cuda 드라이버, 유틸)
nvidia-driver-510/jammy-updates,jammy-security,now 510.85.02-0ubuntu0.22.04.1 amd64 [installed]
  NVIDIA driver metapackage
# 모든 드라이버 패키지(커널 드라이버, 2d/3d xorg 드라이버, cuda 드라이버, 유틸)
nvidia-driver-510-server/jammy-updates,jammy-security 510.85.02-0ubuntu0.22.04.1 amd64
  NVIDIA Server Driver metapackage

# server가 붙은 드라이버와 그렇지 않은 지원기간 이외에 큰 차이가 없다.
``````

``````bash
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install nvidia-driver-510 nvidia-dkms-510

$ sudo reboot # 드라이버가 gpu를 인식하기 위해서는 재부팅이 필요하다.
``````

 재부팅 후 nvidia-smi 커맨드를 실행하면, GPU에 대한 간략한 정보를 확인할 수 있는데, 이 정보가 나타난다는 것은 성공적으로 nvidia driver가 설치되었다는 뜻이다.

``````bash
$ nvidia-smi

Sun Sep  4 02:33:11 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.85.02    Driver Version: 510.85.02    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:03:00.0 Off |                  N/A |
| N/A   42C    P8    N/A /  N/A |      4MiB /  4096MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1001      G   /usr/lib/xorg/Xorg                  4MiB |
+-----------------------------------------------------------------------------+
``````

우리가 사용하는 k3s는 Container Runtime으로 containerd를 사용한다. 

(도커를 contianer runtime으로 쓸 수도 있지만, 여러가지 이유로 인해 k3s의 기본설정인 containerd를 사용하고 있다. 두 runtime의 차이와 containrd를 사용하는 이유에 대해서는 다른 포스트에서 다룰 예정이다.)

아무튼, k3s가 nvidia gpu를 사용하기 위해서는 

1. nvidia-container-toolkit이 설치
2. containerd의 low-level runtimes에 nvidia-container-runtime 추가
3. k3s가 선택적으로 이 nvidia-contianer-runtime을 사용가능

이 수행되어야 한다.

하나씩 진행해 보자.

### 4. nvidia-container-toolkit 설치

toolkit을 설치하는 과정은 경우에 따라 변경될 수 있다. nvidia 공식문서를 참고하여 설치하는 것을 추천한다.

- [https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#containerd](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#containerd)

``````bash
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
    && curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey | sudo apt-key add - \
    && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

$ sudo apt-get update \
    && sudo apt-get install -y nvidia-container-toolkit
``````

### 3. containerd 설정파일 수정

이제  containerd가 NVIDIA Container Runtime를 사용할 수 있도록 ``/etc/containerd/config.toml`` 파일을 수정한다.

``````dhall
<--/etc/containerd/config.toml-->
...
privileged_without_host_devices = false
        base_runtime_spec = ""
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
-            SystemdCgroup = false
+            SystemdCgroup = true
+       [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
+          privileged_without_host_devices = false
+          runtime_engine = ""
+          runtime_root = ""
+          runtime_type = "io.containerd.runc.v1"
+          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
+            BinaryName = "/usr/bin/nvidia-container-runtime"
+            SystemdCgroup = true
    [plugins."io.containerd.grpc.v1.cri".cni]
    bin_dir = "/opt/cni/bin"
    conf_dir = "/etc/cni/net.d"
...
``````

containerd를 재시작한다.

``````bash
$ sudo systemctl restart containerd
``````

### 5. containerd 에서 gpu 사용 확인

containerd에서 gpu가 정상적으로 인식되는지 확인한다.

``````bash
$ sudo ctr image pull docker.io/nvidia/cuda:11.0.3-base-ubuntu20.04
$ sudo ctr run --rm -t \
    --runc-binary=/usr/bin/nvidia-container-runtime \
    --env NVIDIA_VISIBLE_DEVICES=all \
    docker.io/nvidia/cuda:11.0.3-base-ubuntu20.04 \
    cuda-11.0.3-base-ubuntu20.04 nvidia-smi
>>>
Sat Sep  3 17:40:08 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.85.02    Driver Version: 510.85.02    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:03:00.0 Off |                  N/A |
| N/A   42C    P8    N/A /  N/A |      4MiB /  4096MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
``````

이제 containerd가 nvidia-container-runtime을 사용하여 container를 생성하고 관리할 수 있다. 하지만, 아직 k3s는 nvidia-contianer-runtime을 사용하지 못한다. 

### 5. k3s의 contianer runtimes에 nvidia-container-runtime을 추가

 다음 단계는 k3s의 container runtime을 nvidia-container-runtime를 사용하는 containerd로 바꾸는 것이다.

``/var/lib/rancher/k3s/agent/etc/containerd/config.toml`` 파일에  ``[plugins.cri.containerd.runtimes."nvidia"]`` 설정과 ``[plugins.cri.containerd.runtimes."nvidia".options]`` 설정을 추가한다.

``````dhall
<--/var/lib/rancher/k3s/agent/etc/containerd/config.toml-->

[plugins.opt]
  path = "/var/lib/rancher/k3s/agent/containerd"

[plugins.cri]
  stream_server_address = "127.0.0.1"
  stream_server_port = "10010"
  enable_selinux = false
  enable_unprivileged_ports = true
  enable_unprivileged_icmp = true
  sandbox_image = "rancher/mirrored-pause:3.6"

[plugins.cri.containerd]
  snapshotter = "overlayfs"
  disable_snapshot_annotations = true

[plugins.cri.cni]
  bin_dir = "/var/lib/rancher/k3s/data/577968fa3d58539cc4265245941b7be688833e6bf5ad7869fa2afe02f15f1cd2/bin"
  conf_dir = "/var/lib/rancher/k3s/agent/etc/cni/net.d"

[plugins.cri.containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"

[plugins.cri.containerd.runtimes.runc.options]
  SystemdCgroup = true

+[plugins.cri.containerd.runtimes."nvidia"]
+  runtime_type = "io.containerd.runc.v2"
+[plugins.cri.containerd.runtimes."nvidia".options]
+  BinaryName = "/usr/bin/nvidia-container-runtime"
``````

파일을 수정한 후 k3s를 재시작하면 k3s는 ``nvidia-container-runtime`` 을 사용하는 containerd를 선택적으로 사용할 수 있다.

``````bash
$ sudo systemctl restart k3s
``````

### 6.  kubernetes nvidia plugin 배포

k8s nvidia plugin은 다음과 같은 역할을 수행한다.

- 클러스터의 각 노드들에 달려있는 gpu수를 파악
- gpu의 상태를 지속적으로 트래킹
- k8s클러스터 내 contianer가 gpu를 사용할수 있도록 함

 [https://github.com/NVIDIA/k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin) 을 참고하면 간단히 daemonset을 배포하여 설치할 수도 있고, helm을 통해서 다양한 커스텀이 가능한 것으로 보인다.

 필자는 현재 특별한 커스텀이 필요한 상황이 아님으로 git에 올라온 daemonset을 조금 수정하여 plugin을 설치하였다.

우선 gpu가 달려있는 노드에 gpu=nvidia라는 레이블을 달아준다.

``````bash
# node에 label 추가
$ kubectl label nodes [gpu가 장착된 노드명 1] gpu=nvidia
``````

이제 git에 올라온 경로대로  DaemonSet을 생성하는 yaml을 파일을 다운받고,

``````bash
$ curl -O https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.12.3/nvidia-device-plugin.yml
``````

gpu가 달려있는 노드에 plugin이 배포될 수 있도록 nodeSelector를 추가한다.

``````yaml
nodeSelector:
  gpu: nvidia
``````

``````yaml
# nvidia-device-plugin.yml

# Copyright (c) 2019, NVIDIA CORPORATION.  All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nvidia-device-plugin-daemonset
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: nvidia-device-plugin-ds
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: nvidia-device-plugin-ds
    spec:
+      nodeSelector:
+          gpu: nvidia
      tolerations:
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      # Mark this pod as a critical add-on; when enabled, the critical add-on
      # scheduler reserves resources for critical add-on pods so that they can
      # be rescheduled after a failure.
      # See https://kubernetes.io/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/
      priorityClassName: "system-node-critical"
      containers:
      - image: nvcr.io/nvidia/k8s-device-plugin:v0.12.3
        name: nvidia-device-plugin-ctr
        env:
          - name: FAIL_ON_INIT_ERROR
            value: "false"
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: device-plugin
            mountPath: /var/lib/kubelet/device-plugins
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
``````

``````bash
$ kubectl apply -f nvidia-device-plugin.yml
``````

마지막으로 RuntimeClass를 생성하여 pod이 nvidia runtime을 선택할 수 있도록 설정한다.

``````yaml
# nvidia-runtime-class.yml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  # The name the RuntimeClass will be referenced by.
  # RuntimeClass is a non-namespaced resource.
  name: "nvidia"
# The name of the corresponding CRI configuration
handler: "nvidia"
``````

``````bash
$ kubectl apply -f nvidia-runtime-class.yml
``````

이제 모든 준비를 마쳤다.

### 6.  테스트 용 GPU pod 생성

gpu를 사용하는 pod을 생성해보자.

``````yaml
# test-gpu-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: gpu
spec:
  restartPolicy: Never
  runtimeClassName: "nvidia" # nvidia runtime 사용
  nodeSelector: 
    gpu: nvidia # gpu가 장착된 node에만 배포
  containers:
    - name: gpu
      image: "nvidia/cuda:11.4.1-base-ubuntu20.04"
      command: [ "/bin/bash", "-c", "--" ]
      args: [ "while true; do sleep 30; done;" ]
``````

``````bash
> kubectl apply -f test-gpu-pod.yml
> kubectl exec -it gpu -- nvidia-smi
Wed Sep 28 03:03:22 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.85.02    Driver Version: 510.85.02    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:03:00.0 Off |                  N/A |
| N/A   39C    P8    N/A /  N/A |      4MiB /  4096MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
``````

성공적으로 pod이 올라갔다!

 k3s에 gpu를 연결하는데 생각보다 시간이 오래걸렸다. 

 대부분의 내용을 [https://itnext.io/enabling-nvidia-gpus-on-k3s-for-cuda-workloads-a11b96f967b0](https://itnext.io/enabling-nvidia-gpus-on-k3s-for-cuda-workloads-a11b96f967b0) 이 블로그에서 참고하였는데, k3d에서 배포한 config.toml 파일이 필자가 사용하는 버전에는 사용할 수 없었다. k3s git에 올라온 이슈를 확인하던 중  ([https://github.com/k3s-io/k3s/issues/4070](https://github.com/k3s-io/k3s/issues/4070)) 최근 버전이 올라와 있어, 이를 참고하여 배포하였다.

 사실 아직 nvidia container runtime, k8s nvidia plugin, cuda(특히, image는 cuda:11.4.1를 사용하는데 실제 nvidia-smi에서 잡히는 cuda version은 로컬에 설치된 11.6으로 나오는 이유?), 등에 대해 명확하게 이해하지 못했다. 기회가 된다면 각 요소들이 어떤 역할을 수행하며, 왜 필요한지에 대해 공부해 볼 예정이다.

 또한 괜찮은 아이디어가 떠오른다면 이 gpu를 활용한 인공지능 토이 프로젝트를 진행해 볼 예정이다.

## Reference

- [https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#containerd](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#containerd)
- [https://askubuntu.com/questions/1262401/what-is-the-nvidia-server-driver](https://askubuntu.com/questions/1262401/what-is-the-nvidia-server-driver)
- [https://itnext.io/enabling-nvidia-gpus-on-k3s-for-cuda-workloads-a11b96f967b0](https://itnext.io/enabling-nvidia-gpus-on-k3s-for-cuda-workloads-a11b96f967b0)
- [https://github.com/k3s-io/k3s/issues/4070](https://github.com/k3s-io/k3s/issues/4070)
- [https://www.samsungsds.com/kr/insights/docker.html](https://www.samsungsds.com/kr/insights/docker.html)
- [https://developer.nvidia.com/blog/announcing-containerd-support-for-the-nvidia-gpu-operator/](https://developer.nvidia.com/blog/announcing-containerd-support-for-the-nvidia-gpu-operator/)
