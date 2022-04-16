+++
author = "dive-ors"
title =  "Docker Engine 01"
date = "2022-04-012T10:23:26+09:00"
draft = true
tags = [
"docker",
"book",
]
+++

## 도커 이미지와 컨테이너 

도커 엔진에서 사용하는 기본 단위는 이미지와 컨테이너이며, 도커 엔진의 핵심이다.

`도커 이미지`

이미지는 컨테이너를 생성할 때 필요한 요소이며, iso 파일과 비슷한 개념이다.
컨테이너를 생성하고 실행할대 읽기 전용으로 사용되며, 도커 명령어로 내려받을 수 있으므로 별도로 설치할 필요가 없다.

기본적인 형태는 아래와 같다.

`repositoryName/ImageName:Tag` 

ex) `ubuntu:latest`

저장소 이미지는 생략 가능하며, 명시되지 않을 경우 `도커 허브의 공식 이미지` 를 의미한다.    
태그는 이미지의 버전을 의미하며, 기본 값은 latest 로 인식된다. 

`도커 컨테이너`

컨테이너란, 도커 이미지에 맞는 `파일 시스템과 격리된 시스템 자원 및 네트워크` 를 사용할 수 있는 독립된 공간을 의미한다.
대부분의 컨테이너는 도커 이미지의 종류에 따라 알맞은 설정 과 파일을 갖고 있기때문에 이미지의 목적과 맞도록 사용된다. 

> nginx 이미지 로 생성된 컨테이너에는 nginx 설정 및 파일들이 포함되어 있다.

컨테이너는 이미지를 `읽기 전용` 으로 사용한다.
이미지에서 변경된 사항만 컨테이너 계층에 저장하므로, 컨테이너의 변경 사항이 이미지에 영향을 주지 않는다.
또한 위에서 언급했듯, 격리된 시스템 자원을 제공받아, 호스트와 분리되어 있는 구조이다.
따라서 다른 컨테이너에서 어떤 Application 이 설치되거나, 삭제되어도 영향을 받지 않는다.


## 도커 컨테이너 다루기 

#### 컨테이너 생성 

```text
docker run -it ubuntu:14.04 
```

`run` 명령은 컨테이너를 생성하고 실행하는 역할을 한다.
`ubuntu:14.04`은 이미지의 이름이며, -it 는 컨테이너와 상호 입출력을 가능하게 한다. 

run 실행 시, 이미지가 로컬에 없다면 도커 허브에서 자동으로 이미지를 내려받아 컨테이너를 생성 / 실행한다.

이는 2가지 명령어로 풀어서 쓸 수 있다.

```text
docker pull centos:7 
docker create -it --name mycentos centos:7 # 입출력을 위해 it option을 명시해야한다.
```

다만 create 는 생성만 할 뿐, run 명령어와 달리 컨테이너 내부로 들어가지 않는다.

> 실행도 하지 않으므로, start 명령을 통해 컨테이너를 실행해야한다.

내부로 들어가기 위해선 `attach` 명령을 사용하면 된다.

```text
docker start mycentos
docker attach mycentos
```

정리하자면 run 과 create 는 아래처럼 동작한다.

`run`  pull -> create -> start -> (-it 사용시) attach     
`create`  pull -> create    

대부분 컨테이너를 생성과 동시에 시작하기 때문에 run 명령어를 더 많이 사용한다. 


#### 컨테이너 목록 확인 

```text
docker ps 
```

`ps` 명령을 통해 정지되지 않은 컨테이너 목록을 볼 수 있다.
모든 컨테이너를 보기 위해선 `-a` 를 통해 목록을 확인 할 수 있다.

#### 컨테이너 삭제 

```text
docker rm mycentos
```

`rm` 명령을 통해 정지된 컨테이너를 삭제할 수 있다.
실행 중인 컨테이너라면, 정지 후 삭제하거나, `-f` option 을 사용할 수 있다.

만약 모든 컨테이너를 삭제하고 싶다면, `prune` 명령을 사용 할 수 있다.

```text
docker container prune
```

또한 -aq option 을 조합하여 컨테이너를 삭제 할 수 있다.

```text
docker ps -aq | xargs docker rm -f 
```

#### 컨테이너를 외부에 노출

컨테이너는 가상 머신과 마찬가지로 가상 IP 주소를 할당받는다. 
기본적으로 172.17.0.x 의 IP를 순차적으로 할당하는데, 컨테이너를 생성 한 후, ifconfig 명령어로 네트워크 인터페이스를 확인 할 수 있다.

```text
➜  ~ docker run -it --name network_test ubuntu:14.04
root@10057ec2108e:/#
root@10057ec2108e:/# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:696 (696.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

도커의 NAT IP 를 할당받은 eth0 인터페이스와 lo 인터페이스가 있으며, 아무 설정을 하지 않았다면 외부에서 접근할 수 없다.

> 도커가 설치된 호스트에서만 접근할 수 있다.

외부에 컨테이너 Application 을 노출시키기 위해서는 etho 의 IP / Port 를 Host의 Ip / Port 에 바인딩해야한다.

```text
docker run -it --name mywebserver -p 80:80 ubuntu:14.04
```

`-p` option 을 통해 컨테이너의 포트를 호스트의 포트와 바인딩하여 연결할 수 있게 설정한다.

```text
-p ${host-port}:${container-port}
```

만약 Host 의 특정 Ip 로 연결하고 싶다면, 아래처럼 사용할 수 있다.

```text
-p 192.168.0.100:7777:80 
```

또한 여러개의 포트를 외부에 개방하려면 -p option 을 여러번 써서 설정 할 수 있다.

> apache 띄우는 예시 

```text
docker run -it --name ubuntu -p 80:80 ubuntu:latest

apt-get update
apt-get install apache2 -y 
service apache2 start
```

내려받은 ubuntu image 를 통해 port 를 연결하고, apache 를 설치하면 localhost:80 으로 컨테이너에 접근 할 수 있다.

- `1. host Ip 의 80 port 로 접근` 
- `2. 80번 port는 컨테이너의 80 port 로 포워딩`
- `3. 웹서버 접근`


