# 도커 설치 가이드

이 글은 Ubuntu에서 도커를 설치하는 방법을 공부하며 정리한 내용입니다.

## 설치전 준비사항

### OS 요구사항

Docker Engine을 설치하려면 다음 Ubuntu 버전 중 하나의 64 비트 버전이 필요합니다.

- Ubuntu Focal 20.04 (LTS)
- Ubuntu Bionic 18.04 (LTS)
- Ubuntu Xenial 16.04 (LTS)

### 기존 도커 삭제

```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

## 도커 설치

도커를 설치하는 방법은 아래와 같으며, 이 중 리포지터리를 사용한 설치로 진행하겠습니다.

- 가장 권장되는 [리포지터리를 사용한 설치](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
- DEB 패키지를 다운로드하여 [수동으로 설치](https://docs.docker.com/engine/install/ubuntu/#install-from-a-package)하고 업그레이드를 완전히 수동으로 관리하는 방법
- 테스트 및 개발 환경을 위한 자동화 된 편의 [스크립트를 사용하여 설치](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)

### 리포지터리 설정

apt가 HTTPS로 리포지터리를 사용하는 것을 허용하기 위한 패키지 설치

```bash
$ sudo apt-get update

$ sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

### 도커 공식 GPG 키 추가

```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add -
```

### 도커 apt 리포지터리 추가

```bash
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

### 도커 CE 설치

```bash
$ sudo apt-get update
$ sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

만약 특정 버전을 설치하려면 아래 방법을 사용하세요.

```bash
# 사용 가능한 버전 확인
$ apt-cache madison docker-ce

# output 확인
docker-ce | 5:18.09.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
docker-ce | 5:18.09.0~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
docker-ce | 18.06.1~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
docker-ce | 18.06.0~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages

# 특정 버전 설치
$ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

## 도커를 사용한 실습

위 내용을 바로 테스트할 수 있도록 [docs/install/Dockerfile](https://github.com/divlook/docker-study/blob/main/docs/install/Dockerfile) 파일을 만들어놓았습니다.

```bash
$ git clone git@github.com:divlook/docker-study.git
$ cd docker-study/docs/install

# 이미지 build
$ docker build -t docker-study/install .

# 컨테이너 생성
$ docker run -it --rm --name docker-study/install docker-study.install bash

# 컨테이너 내부에서 docker를 실행해봅니다.
$ docker -v
```

## 도커 삭제

```bash
$ sudo apt-get purge docker-ce docker-ce-cli containerd.io

# 이미지, 컨테이너, 볼륨 또는 사용자 정의 된 구성 파일은 자동으로 제거되지 않습니다. 모든 이미지, 컨테이너 및 볼륨을 삭제하려면
$ sudo rm -rf /var/lib/docker
```

## 도커 데몬의 드라이버 systemd로 변경

```bash
# docker 설정 폴더 생성
$ mkdir -p /etc/docker

# 도커 데몬 설정 파일 생성
$ cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

# /etc/systemd/system/docker.service.d 생성
sudo mkdir -p /etc/systemd/system/docker.service.d

# 도커 재시작
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 부팅시 시작되게 설정

```bash
# 설정
$ sudo systemctl enable docker

# 해제
$ sudo systemctl disable docker
```

## 참고 사이트

- https://docs.docker.com/engine/install/ubuntu/
- https://docs.docker.com/engine/install/linux-postinstall/
- https://kubernetes.io/ko/docs/setup/production-environment/container-runtimes/#%EB%8F%84%EC%BB%A4
