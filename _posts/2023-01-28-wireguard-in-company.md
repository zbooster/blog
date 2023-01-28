---
layout: post
title: "사내 WireGuard VPN 구축하기"
subtitle: ""
date: 2023-01-28 15:00:00 +0900
---
<img src='https://user-images.githubusercontent.com/100823210/215249301-1d4beb30-16ea-448d-814c-b66a6296f030.png' align='center'>

꼭 한달만에 다시쓰는 글이다. 최근 프로젝트에 나가있어 VPN으로 사내에 접속할 일이 많았는데 기존에 구축해놓았던 Softether VPN의 속도가 느려서 그 대안으로 찾아본 것이 WireGuard 이다.


# 왜 WireGuard를 선택했는가?
### 1. Softether VPN서버의 불만
Softether를 구축하면서 편리했던 점은 설치가 쉽고 GUI로 관리가 가능했기 때문이고 딱히 클라이언트를 설치하지 않아도 연결을 할 수 있기 때문이였다. 그 때 WireGaurd도 선택지에 있었으나 선택하지 않은 이유는 클라이언트 설치가 필요했고 접속을 위해 Key를 배포해야 했었기 때문이다. (윗 분들은 그런거 싫어한다.)

그러나, 요즘 vSphere내 테스트 머신에 접속하여 작업하는데 설치 스크립트 수행 중간에 접속이 끊겨 먹통이 되는 현상이 나타났다. 처음에는 별 생각이 없었지만, 현재하는 업무 특성상 계속 설치작업이 일어나기 때문에 슬슬 짜증이 났다.

접속이 끊기는 패턴을 확인해보면, 설치 스크립트 같이 다른 실행파일을 fork해서 수행하고 결과를 얻는 과정에서 끊어졌다. Windows에 내장된 IPSec방식이 아닌 Client설치를 하고 VPN을 연결해서 수행하면 좀 더 나은모습을 보였지만 그래도 끊기기는 마찬가지였다.

### 2. WireGuard Easy의 발견
사실 이 프로젝트의 발견이 결정적이였다. 이전에 테스트할때는 Linux에 명령을 수행해서 키를 만들고 설정파일을 만들어 배포하는 방식이 너무 번거롭다고 느껴졌는데 Docker 이미지만 실행하면 설치가 끝나고 신규 사용자를 등록하는 과정은 GUI로 실행하면 된다.

# WireGaurd 
기존에 사용하던 Docker 서버가 있다면 3번부터 하면 된다.

### 1. VM서버 준비
애초에 Docker만 하나 설치하여 운영할 생각이므로 사내 VM서버에 Ubuntu를 최소 설치하여 준비했다. 자세한 설치과정은 인터넷에 많이 나와있기 때문에 여기서 설명하지는 않겠다.

### 2. Docker 설치
이 과정도 [DockerHub - WireGuard Easy](https://hub.docker.com/r/weejewel/wg-easy)에 들어가면 설명되어 있다. root 유저로 전환할 수 있는 유저로 수행하거나, root 유저로 수행하면 된다. (이 과정에서도 기존 Softether는 끊기더라...)
```bash
$ curl -sSL https://get.docker.com | sh
$ sudo usermod -aG docker $(whoami)
$ exit
```


### 3. 도메인 준비 및 포트포워딩 
외부에서 이 VPN서버를 찾아오려면 회사의 Public IP나 이와 연결된 도메인이 필요하다. 나는 기존에 사용하던 Softether에서 제공해주는 DDNS 주소를 사용했다. 그리고, 포트포워딩은 KT를 사용중이기 때문에 아래와 같이 51820/UDP, 51821/TCP를 등록했다.
![tempsnip](https://user-images.githubusercontent.com/100823210/215251233-56de1cca-45ee-4486-9b4c-16c4af41191b.png){: width="100%" height="100%"}

### 4. wg-easy 도커 이미지 실행
준비가 다 되었다면, 아래 2가지만 채워서 docker 명령을 수행하면 된다. 
- 🚨YOUR_SERVER_IP: 앞서 준비한 도메인이나 Public IP
- 🚨YOUR_ADMIN_PASSWORD: 관리자 GUI에 접속할 때 사용할 비밀번호


```bash
docker run -d \
  --name=wg-easy \
  -e WG_HOST=🚨YOUR_SERVER_IP \
  -e PASSWORD=🚨YOUR_ADMIN_PASSWORD \
  -v ~/.wg-easy:/etc/wireguard \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  --restart unless-stopped \
  weejewel/wg-easy

```
이제 설치는 모두 끝났다.

### 5. 관리자 페이지 접속
앞서 설정한 <도메인 이름>:51821에 접속하면 아래와 같이 GUI화면이 나온다. 만약 이 부분에서 실패한다면, 포트포워딩이나 DDNS 설정을 더 살펴봐야 한다. 

![wg-easy - Admin](https://user-images.githubusercontent.com/100823210/215251734-ea5199c2-5fef-4667-89f9-80786369b21f.png){: width="100%" height="100%"}

docker 이미지를 실행할 때, 입력했던 패스워드를 넣고 들어가면 Client를 추가할수 있는 화면이 나온다. New나 New Client를 누르고 간단히 이름을 입력하면 클라이언트가 만들어진다. 

![wg-easy - New Client](https://user-images.githubusercontent.com/100823210/215252005-564ce956-2c41-4047-a24a-6c60b8cb5d6e.png){: width="100%" height="100%"}

만들어지면 우측에 아이콘이 생기는데 Windows에서 접속할 예정이므로 아래 화살표를 가진 아이콘을 클릭하면 conf 파일을 받는다.
![image](https://user-images.githubusercontent.com/100823210/215252191-99192a44-0281-43e0-8e74-c75ff9f3f410.png){: width="100%" height="100%"}

### 6. Windows 클라이언트 설치 및 접속
다시 WireGaurt 홈페이지의 Installation 메뉴어 가면 Windows용 설치파일을 찾을 수 있다. 다운로드 받아 설치하면 된다.

![image](https://user-images.githubusercontent.com/100823210/215252242-59ed0274-22ea-490f-86d3-7608cae64f27.png)

이후 클라이언트를 실행하고 아래와 같은 화면이 나오면 Add Tunnel으 눌러 아까 다운로드 받은 conf 파일을 등록하면 된다. 그리고 Activate를 누르면 접속이 완료된다.

![tempsnip](https://user-images.githubusercontent.com/100823210/215252524-ccbc77ad-89fe-4f67-8748-8bd1a273ab8c.png){: .align-center}

# 설치 후기
VPN을 WireGuard로 변경한 이후에 설치 스크립트 수행 시, 접속이 끊기는 부분은 완전히 사라졌다. 좀 더 테스트를 해보고 사내 다른 사용자들에게 전파할 예정이다.

WireGaurd에 가장 큰 불만은 메뉴얼이 잘 되어있지 않아 설정하는 부분이 다소 어렵게 느껴진다는것이다. 홈페이지에 들어가서 Quick Start를 보면 나름 쉽게 설명한다고 동영상이나 커맨드를 써놓았지만, 처음 사용하는 사용자 입장에서는 더 혼란스러울 수 있다. 이 글이 입문자들에게 도움이 되었으면 좋겠다.

# 레퍼런스
- [WireGaurd 홈페이지](https://www.wireguard.com/)
- [DockerHub - WireGuard Easy](https://hub.docker.com/r/weejewel/wg-easy)
