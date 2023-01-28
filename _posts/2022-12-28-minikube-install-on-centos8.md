---
layout: post
title: "CentOS 8에 minikube 설치"
subtitle: "CentOS 8에 minikube 설치"
date: 2022-12-28 14:00:00 +0900
---
Docker의 유료화 이후로 자주 사용하던 docker-compose가 손이 가질 않는다. 마침 회사에서도 Kubenates를 공부하라고 하기에 minikube를 설치하고 테스트를 수행해보려고 합니다.


# 준비사항
- minikube를 설치할 Virtual Machine (2 CPUs, 2 GB Memory, 20 GB 이상의 Disk)

# podman 설치
podman은 docker의 대안으로 Redhat 계열에서 많이 사용하고 있습니다. 매우 유사하지만 deamon없이 동작한다는 점에서 다릅니다. 

설치방법은 [podman 홈페이지 - 설치](https://podman.io/getting-started/installation)에 잘 설명되어 있다. CentOS의 경우 아래 명령으로 손쉽게 설치가 가능합니다.
```bash
yum inatll -y podman
```

# minikube 설치
[minikube 홈페이지 - 시작](https://minikube.sigs.k8s.io/docs/start/)에 가면 아래와 같이 OS에 따른 설치방법을 제공합니다. CentOS에 설치할 예정이므로 Linux, x86_64, Stable, RPM package를 선택합니다. (Binary를 받아도 되나 CentOS이므로 RPM을 선택했습니다.)

![minikube - install](https://user-images.githubusercontent.com/100823210/209761607-b3ace868-b7b3-483c-8779-7b9245976222.png){: width="100%" height="100%"}

가이드에 나와 있는대로 수행합니다.
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
sudo rpm -Uvh minikube-latest.x86_64.rpm
```

# minikube 시작 
minikube를 시작하면 다음과 같이 알아서 podman을 찾아 올라옵니다. 최초 시작할때는 필요한 image도 자동으로 받아오고 아기자기한 이모티콘도 터미널에 나타납니다.

저는 root 유저로 시작했기 때문에 --force 옵션을 추가 했습니다.
```bash
minikube start --force
```
![minikube - start](https://user-images.githubusercontent.com/100823210/209762951-5dc5c025-f547-4765-aaf9-1fd0984f7a12.png){: width="100%" height="100%"}

설치가 끝나면 아래와 같이 .bash_profile에 alias를 등록해주고 pod들을 확인해 봅니다.
```bash
echo 'alias kubectl="minikube kubectl --"' >> .bash_profile
. .bash_profile
kubectl get po -A
```
![minikube - pod](https://user-images.githubusercontent.com/100823210/209763344-fda901dc-a0c8-4bdf-b18e-df66b8d96105.png){: width="100%" height="100%"}

# 레퍼런스
- [minikube - 문서](https://minikube.sigs.k8s.io/docs/)






