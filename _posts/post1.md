---
layout: post
title: "Oracle Linux 7.9 설치"
subtitle: "Oracle Linux 7.9를 설치하는 방법"
date: 2022-12-20 10:00:00 +0900
---

가끔 테스트를 진행하다 보면 Oracle 데이터베이스를 설치해야 하는데 CentOS나 Ubuntu에도 설치는 가능하지만 
설치 준비사항을 충족하는데 꽤 시간이 걸린다. Oracle 데이터베이스 단독 설치의 경우 Oracle Linux를 이용하면 편하게 준비를 할 수 있다.

# Oracle Linux 다운로드
[Oracle Linux ISO Download](https://yum.oracle.com/oracle-linux-isos.html)에 접속하면 원하는 버전의 Full ISO를 
다운로드 받을 수 있다. 이번 포스트에서는 7.9를 설치할 예정이다.

만약, 디스크 공간이 넉넉하지 않고 Minimal Install만 진행할 예정이라면 Boot ISO를 받아도 상관없으나, 
설치과정에서 Repository의 주소를 타이핑해야하는 귀찮음이 존재한다.

# Vritual Machine 생성
최근 사내에 vSphere 서버가 생겨 요즘은 vSphere로 VM을 생성한다. 다른 VM서버도 양식은 비슷하니 참고하여 만들면 된다. 
아무리 테스트 목적의 서버지만 CPU는 2개 이상, 메모리는 4 GB정도가 적당하다. 그리고 위에서 다운로드 받은 ISO를 CD/DVD드라이브에 
설정하고 **전원을 켤 때 연결**을 잊지 말자.

![새 가상 시스템의 하드웨어 설정](https://user-images.githubusercontent.com/100823210/208561487-8be059de-d663-48de-bddb-1c091d75f1e8.png){: width="100%" height="100%"}

# 설치 진행
기본적인 방법은 RHEL/CentOS와 거의 같다. 

### 1.설치언어 선택
필자는 영어로 진행하는것을 선호한다. 대부분의 소프트웨어가 그렇지만 영어권에서 만들어지기 때문에 영어로 된 설명을 보는것이 정확한 경우가 많기 때문이다. (한글로 선택해도 되지만 선택하는 시간에 비해 얻을 수 있는건 많지 않다)

![설치 언어 선택](https://user-images.githubusercontent.com/100823210/208582677-4ef5791f-f3f3-4cbf-8999-fd4b303b5a4b.png){: width="100%" height="100%"}

### 2.KDUMP 끄기
테스트 시스템에서 대부분 KDUMP는 불필요하다. 괜한 리소스 낭비하지 말고 끄자.

![KDUMP 끄기](https://user-images.githubusercontent.com/100823210/208582872-8b3b9b49-38c6-464d-a1e2-d3fc3d026246.png){: width="100%" height="100%"}

### 3.NETWORK & HOSTNAME 설정
다른 설정을 들어가기 전에 NETWORK 설정부터 진행한다. 이유는 이후 DATE설정할때 NTP가 자동으로 ON되기 때문이다. 공유기나 NETWORK의  DHCP 설정이 되어 있는 경우 OFF을 ON으로 전환한다.
그리고 가급적이면 HOSTNAME도 미리 설정하여 나중의 수고를 줄여주는 것이 좋다.

![NETWORK & HOST NAME](https://user-images.githubusercontent.com/100823210/208582980-728eba0d-620d-4660-891d-04a684b750c4.png){: width="100%" height="100%"}

### 4.DATE & TIME 설정
NETWORK를 설정했다면 자동으로 ON되어 있을것이다. 마우스를 이용하여 한국의 중심부를 찍으면, 시간대가 Asia/Seoul로 변경된다.

![DATE & TIME](https://user-images.githubusercontent.com/100823210/208583077-50ee69f2-66d0-444b-a76f-cbeea27f4a21.png){: width="100%" height="100%"}

### 5.설치위치 선택
이미 선택이 되어있지만 들어가서 DONE을 누르고 나온다. 수행하지 않을 경우, 설치가 진행되지 않는다.

![image](https://user-images.githubusercontent.com/100823210/208583219-1e5380cc-97e1-491c-b1bb-e0ebcb962226.png){: width="100%" height="100%"}

### 6.설치 시작
필수적인 설정을 마치면 아래와 같이 "Begin Installation" 버튼이 활성화 된다. 누르고 진행한다.

![Begin Installation](https://user-images.githubusercontent.com/100823210/208583279-514181e7-62f7-49db-8e86-84bc894e61b5.png){: width="100%" height="100%"}

### 7.ROOT PASSWORD
운영환경이면 USER CREATION을 해서 user를 생성한 다음 sudo 명령을 통하여 root 권한을 행사하는것이 낫다. 그러나, 테스트환경에서 특별히 필요없기 때문에 root의 비밀번호를 설정한다.
![Install progressing](https://user-images.githubusercontent.com/100823210/208583321-653a05fe-c697-4565-b6bb-6fae1d705601.png){: width="100%" height="100%"}

만약, root의 비밀번호를 root로 동일하게 설정하는 경우 DONE버튼을 두번 눌러야 진행이 된다.
![root password](https://user-images.githubusercontent.com/100823210/208583391-8f35ab9f-8856-4ba5-b2f2-fc9daaa58143.png){: width="100%" height="100%"}{: width="100%" height="100%"}

### 8.설치 완료
설치가 완료되면 아래와 같이 "Reboot" 버튼이 활성화된다. 누르면 재부팅이 된다.
![install completed](https://user-images.githubusercontent.com/100823210/208584979-f2cb4700-59b8-45c9-9fa5-5fe955e8686b.png){: width="100%" height="100%"}

# 마치며
이로서 Oracle Linux 7.9의 설치과정을 마쳤다. Redhat 계열의 Linux이기 때문에 RHEL/CentOS와 별반 차이가 없다.
단지 오라클이 제공하는 몇 가지 설치과정의 편의가 있기때문에 가끔 Oracle Linux를 선택하게 된다.
