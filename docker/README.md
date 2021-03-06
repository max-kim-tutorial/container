# 필요에 의해 배우는 Docker

얕게..얕게...

## 컨테이너를 왜 쓰는가?

**여러 서버 인스턴스를 가상화 기술을 통해 하나의 머신 안에 여러개의 독립적인 컨테이너 영역을 만들어 배포하고 구동한다**

컨테이너는 일종의 추상화 레이어라는 생각이 들었음. 직접 인프라 인스턴스를 세팅하고 조율하는 것은(베어메탈까지 갈 필요도 없고 AWS 콘솔만 생각해봐도) 하드웨어단, 인프라단의 복잡하고 오래 걸리는 과정이지만 가상화를 하면 쉽게 가상화 컨테이너라는 레이어 속에서 설정과 run/stop/restart들을 쉽게 소프트웨어의 영역에서, 관리자/애플리케이션과 더 가까운 영역에서 할 수 있게 된다. 하드웨어 설정의 영역을 소프트웨어로 좀 더 가져온 것

- 도커는 OS 컨테이너를 구축해주는 컨테이너화 도구이다. 소프트웨어를 패키징하여(말아) 어느 하드웨어의 어느 OS 환경에서도 동일하게 동작하도록 만든다
- 대충의 과정

  - 파일 토대로 이미지 생성(직접 머신을 세팅할 필요는 없다)
  - 도커 레지스트리에 올리고
  - 해당 이미지를 각 서버 구동 환경에서 Pull 받아서 Container로 구동한다(run)
  - start/rester/stop

- 개빠르다 : 가상화 기술 중 컨테이너 구축이 가장 빠르고 용량도 적다 -> 그래서 쓰고 버리기 매우 편하다
- 인프라에 깔리는 환경과 애플리케이션의 형상을 이미지로 기록해 간편하게 붙이고 떼고 구동한다 -> 그래서 쓰고 버리기 매우 편하다
- VM과 컨테이너는 어떻게 다른가? : 둘 다 **가상화**된 운영환경을 제공하는 서비스이다.
  - 가상머신은 Hypervisor가 여러개의 VM을 관리한다. 게스트 OS를 가상화된 파트마다 모두 설치한다
  - 컨테이너는 Hypervisor 필요 없이 컨테이너 런타임이 다 처리한다. 도커 엔진은 HostOs를 쪼개서 각 컨테이너가 필요로 하는 OS와의 차이를 파악한 후 부족한 부분만 수집하여 공유하는 부분만 HostOS를 사용한다. 게스트 OS를 컨테이너마다 깔지 않아 가볍다

## Docker Container

- 도커 이미지 : 컨테이너 생성에 필요한 요소. 가상머신을 생성할때 사용하는 iso파일같은 개념
  - 여러 계층으로 된 바이너리 파일
  - 이미지가 컨테이너를 정의한다 `저장소 이름/이미지이름:태그`
- 도커 컨테이너 : 이미지로 컨테이너를 생성하면 해당 이미지의 목적에 맞는 파일이 들어있는 파일시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있음
  - 생성될 때 사용된 도커 이미지의 종류에 따라 알맞는 설정과 파일을 가지고 있음. 도커 이미지의 목적에 맞도록 사용하는 것이 일반적
- 이미지와 컨테이너의 관계는 1:N

### 컨테이너 구동

컨테이너에 접속시 기본 사용자는 root이고 호스트 이름은 무작위의 16진수 해쉬값

```bash
docker run -i -t ubuntu:14.04 # 이미지 이름으로 컨테이너 생성하기, 로컬 도커 엔진에 이미지가 존재하지 않으면 도커 허브에서 받음

exit # 컨테이너 내부에 들어와있는 상태에서 배시 셸을 종료하고 컨테이너를 정지시킴
docker images # 도커 엔진에 존재하는 이미지의 목록 출력

docker create -i -t --name mycentos centos:7 # 도커 create는 run과는 다르게 컨테이너를 생성하면서 컨테이너로 접속하지 않음
docker start mycentos # 컨테이너 구동
docker attach mycentos # 컨테이너 내부로 들어감

docker ps # 지금까지 정지되지 않은 컨테이너 목록 출력

docker rm angry_morse # 컨테이너 삭제, 삭제후 복구 불가, 삭제하기 전에 정지해야함
docker container prune # 모든 컨테이너 삭제
```

컨테이너도 가상머신과 마찬가지로 가상 IP 주소를 할당받으며, 기본적으로 도커는 컨테이너에 `172.17.0.x` IP를 순차적으로 할당. 외부의 컨테이너에 애플리케이션을 노출하기 위해서는 `eth()`의 IP와 포트를 호스트의 IP와 포트에 바인딩해야 함.

```bash
ifconfig # 각 컨테이너에 접속한 채로 실행하면 어떤 ip가 바인딩되어있는지 알 수 있다.
docker run -i -t --name mywebserver -p 192.168.0.1000:7777:80 ubuntu:14.04 # 호스트에서, P옵션으로 컨테이너의 포트를 호스트의 포트와 바인딩
```

`[호스트의 포트]:[컨테이너의 포트]` : 호스트의 포트를 컨테이너의 포트로 연결. 여러개의 포트를 개방하고 싶다면 -p를 여러번 쓴다. 아파치를 예로 들어보면 172 대역을 가진 컨테이너의 NAT IP와 80번 포트로 서비스하므로, 여기에 접근하려면 172.17.0.x:80의 주소로 접근해야함.

```shell
호스트의 80번 포트 -> 80번 포트는 컨테이너의 80번 포트로 포워딩 -> 웹 서버 접근
```

컨테이너 애플리케이션을 하나만 동작시키면 컨테이너 간의 독립성을 보장함과 동시에 애플리케이션의 버전 관리, 소스코드 모듈화가 더욱 쉬워지고, 애플리케이션을 분리시켜 동작시키면 도커 이미지를 관리하고 컴포넌트의 독립성을 유지하기 쉬움. 도커에서는 이걸 권장. 한 컨테이너의 프로세스 하나만 실행하는 것이 도커의 철학.. 컨테이너는 쓰고 버리기 쉽기 때문에 이런 세팅이 가능함(가령 데이터베이스와 서버 어플리케이션)

`detached -d` : attach 가능한 컨테이너는 bash를 통해 접속 가능하지만 detach는 그렇지 않음. 입출력이 없는 상태로 컨테이너가 실행되며, 컨테이너 내부에서 프로그램이 터미널을 차지하는 포그라운드로 실행되어 사용자의 입력을 받지 않음. 이때 detached는 터미널에서 프로그램이 돌아야함. `-i -t`와 반대, 한 컨테이너를 하나의 애플리케이션으로만 사용할때 사용하는 방법

```shell
docker run -d --name wordpress -e WORDPRESS_DB_HOST=mysql -e WORDPRESS_DB_USER=root -e WORDPRESS_DB_PASSWORD=password mysql:5.7 -p 80 --link wordpressdb:mysql
# 컨테이너 실행하는데 이름, 환경변수, 호스트의 포트 하나와 컨테이너의 80번 포트 연결,
# Link는 내부 IP에 대한 컨테이너의 alias, NAT API가 아니라 alias로 컨테이너끼리 소통한다(현재 deprecated-bridge사용)
```

### 도커 볼륨

컨테이너를 없애면, 컨테이너 계층에 저장되어있던 데이터베이스의 정보가 삭제되고 컨테이너를 삭제하면 데이터를 복구할 수 없는데 컨테이너는 갈아치우기가 너무 쉬움. 컨테이너의 데이터를 영속적으로 사용하기 위해서는 도커 볼륨을 사용함

```shell
docker run --name wordpressdb_hostvolume -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=wrodpress -v /home/wordpress_db:/var/lib/mysql mysql:5.7
# DB 컨테이너, -v 옵션으로 호스트의 /home과 컨테이너의 /var/lib 디렉토리를 공유.
# 동기화되는 게 아니라 완전히 같은 디렉토리를 사용한다고 한다.
# 컨테이너의 파일이 호스트로 복사된 것이라고 할 수 있따
```

이미지에 원래 존재하던 디렉토리에 호스트의 볼륨을 공유하면 컨테이너의 디렉토리 자체가 덮어씌워짐. -v 옵션을 통한 호스트 볼륨 공유는 호스트의 디렉토리를 컨테이너의 디렉토리에 마운트. 디렉토리 찾아갈때만 다른듯

볼륨 컨테이너를 사용해서 `--volumes-from 옵션` 호스트의 볼륨을 여러 컨테이너와 공유하는 방법도 가능하고, 도커 자체에서 제공하는 볼륨 기능을 활용해 데이터 보존도 가능. `docker volume create`

쓰다버려야 하므로 도커 컨테이너 자체에서 데이터를 가지고있는 것은 사실상 지양하는게 좋긴 하다. 호스트 볼륨이나 볼륨 컨테이너를 사용해서 책임을 위임하는게 좋다

### 도커 네트워크

`docker0 default 브리지` : 도커가 설치된 호스트에서 ifconfig 등으로 넽워크 인터페이스 확인하면 실행중인 컨테이너 수만큼 veth(virtual eth)로 시작하는 인터페이스가 생성된다. 이 veth는 각 컨테이너의 eth0과 연결이 되어있어 호스트-컨테이너간 통신, 컨테이너-컨테이너간 통신을 가능케 한다. 일종의 네트워크 브릿지(버스같기도 함)를 형성한다. 사용자의 선택에 따라 기본으로 설정된 브리지 말고 다른 네트워크 드라이브를 갈아끼울 수도 있다.

```sh
docker network create --driver bridge mybridge # 새로운 브리지 타입의 네트워크
```

네트워크를 호스트로 설정하면 호스트의 네트워크 환경을 그대로 사용할 수 있다. 이 경우는 호스트 머신에서 설정한 호스트 이름도 컨테이너가 물려받으므로 컨테이너의 호스트 이름도 16진수가 아니라 도커 엔진이 설치된 호스트 머신의 호스트 이름이 된다. 이렇게 설정하면 컨테이너 내부의 애플리케이션을 별도의 포트포워딩 없이 바로 서비스가 가능하다.

```sh
docker run -i -t --name network_host --net host ubuntu:14.04 # --net 옵션으로 host를 갖다 지정
docker run -i -t --name network_host --net none ubuntu:14.04 # --net 옵션으로 none을 지정 이러면 연결 단절 localhost만 사용
# --net으로 다른 컨테이너 이름을 입력하면 다른 컨테이너의 네트워크 네임스페이스 환경을 공유(내부 IP, MAC Address)
docker run -i -t -d --name network_container_2 --net container:network_container_1 ubuntu:14.04
```

브리지 타입 넽워크와 `--net-alias` 명령어를 같이 사용하면 클라이언트사이드 로드밸런싱(?이라고 해야하나)이 가능하다.

```sh
docker run -i -t -d --name network_alias_container1 --net mybridge --net-alias alicek106 ubuntu:14.04
docker run -i -t -d --name network_alias_container2 --net mybridge --net-alias alicek106 ubuntu:14.04
docker run -i -t -d --name network_alias_container3 --net mybridge --net-alias alicek106 ubuntu:14.04
# 이렇게 하는 경우 IP 끝자리만 다른 주소가 할당됨 172.18.0.3 -> 172.18.0.4 -> ...
```

이렇게 만들어놓고 alicek106으로 핑을 보내면, 3개 컨테이너 중 하나로 핑을 보내는데, 이때 라운드로빈으로 컨테이너 결정됨. 도커 엔진에 내장된 DNS(...내장하고 있군!)가 alias라는 호스트 이름을 설정한 컨테이너로 변환한다.

## Docker 이미지

도커는 기본적으로 도커 허브라는 중앙 이미지 저장소에서(apt-get이나 yum이나 brew처럼) 이미지를 내려받는다. 도커 계정을 가지고 있다면 누구든지 이미지를 올리고 내려받을 수 있기 때문에 이미지 공유도 쉽다. bare한 이미지는 도커 허브에서 제공하고(ubuntu:14.04, centos:7) 다른 사람들이 올려놓은 이미지도 있음(Apache Tomcat, Hadoop) 이미지를 만들지 않아도 내려받을 수 있는게 많음

컨테이너 내용을 이미지로 만들어야 하는 경우도 있음. 다음과 같이. 컨테이너의 현 상태에서 변경사항을 이미지로(스냅샷) 만듬

```sh
docker run -i -t --name commit_test ubuntu:14.04 # 컨테이너 생성
>> echo test_first! >> first # 기존의 이미지로부터 변경 사항 만듬
# 호스트로 빠져나온 후
docker commit -a "alicek106" -m "first commit" commit_test commit_test:first # commit_test:first라는 이미지 생성
# -a는 author, -m은 커밋 메시지, docker images로 생성된 이미지 호스트에서 확인
```

`이미지 레이어 구조` : 컨테이너의 변경사항을 커밋할때 컨테이너에서 변경된 사항만 레이어로 저장하고, 레이어를 포함해 새로운 이미지를 생성한다. 실제 크기는 첫번째 레이어로부터 incremental됨. 각 레이어(변경사항)은 sha256 체크섬을 가진다.

이미지를 사용중인 컨테이너가 있다면 이미지를 지울 수 없다. 삭제되는 이미지의 부모 이미지가 존재하지 않아야 해당 이미지의 파일이 실제로 삭제된다. 이미지를 단일 바이너리 파일로 저장할때는 docker save를 사용한다. load를 통해 다시 도커 엔진으로 이미지를 저장한다

```sh
docker save -o ubuntu_14_04.tar ubuntu:14.04
docker load -i ubuntu_14_04.tar
```

다른 도커 엔진에 이미지를 배포하고 싶을때는 도커 허브 이미지 저장소에 올리거나, 도커 사설 레지스트리에 이미지 저장소를 만들어 푸쉬한다.

## Dockerfile

개발한 애플리케이션을 컨테이너화-> 셋업 -> 이미지화 하고 싶으면 이미지로 컨테이너 생성한다음에 그 위에 뭐 셋업하고 이미지로 커밋해서 사용한다. 이 방법을 사용하면 애플리케이션이 동작하는 환경을 구성하기 위해 일일히 수작업을 컨테이너에서 패키지 설치하고 소스코드를 복제하고 등등을 한번 이상은 해줘야하기 때문에 귀찮다.

도커는 이 작업을 소스코드단에서 할 수 있게 build명령어를 제공하며, 완성된 이미지를 생성하기 위해 컨테이너에 설치해야 하는 패키지, 추가해야하는 소스코드, 실행해야 하는 명령어와 셸스크립트 등을 하나의 파일에 기록해두면 도커는 이 파일을 읽어 컨테이너에서 작업을 수행 한 후 이미지로 만들어낸다. 이러한 작업을 기록한 파일이 dockerfile이다. dockerfile을 사용하면 직접 컨테이너를 생성하고 이미지로 커밋해야 하는 번거로움을 덜고, 깃과 같은 개발 도구를 통해 애플리케이션의 빌드와 배포를 자동화한다.

생성한 이미지를 도커 허브를 배포할 때 이미지 자체를 배포하는 대신 도커파일을 배포해놓을 수도 있다. 도커 허브에 올려져 있는 이미지는 Dockerfile을 함게 제공하는 경우가 많다. 애플리케이션에 필요한 패키지등을 명확히 하고, 이미지 생성을 자동화하고 쉽게 배포할 수 있다는 점에서 이점이 있다.

### 도커파일 예제 - 기본 명령어

명령어는 일반적으로 대문자 표기

```dock
FROM ubuntu:14.04
MAINTAINER alicek106
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFOREGROUND
```

- FROM : 생성할 이미지의 베이스. 무조건 한번 이상 입력. docker run에서 이미지 이름 사용하는 거랑 같음
- MAINTAINER: 이미지 생성한 사람 정보. 요즘은 Label로 표현
- LABEL : 이미지에 메타데이터 추가 키:값. docker inspect시 나오는 정보
- RUN: 이미지를 만들기 위해 컨테이너 내부에서 명령어 실행.
  - 이미지를 빌드할때 별도의 입력을 받아야 하는 RUN이 있다면 build는 이를 오류로 간주
  - 사용할 셸/ 명령줄 인자/명령줄 인자
  - JSON처럼 쌍타옴표
- ADD: 파일을 이미지에 추가. 추가하는 파일은 도커파일이 위치한 디렉터리(컨텍스트)에서 가져옴
- WORKDIR : 명령어를 실행할 디렉터리. cd 명령어
- EXPOSE: 이미지에서 노출할 포트 설정. 포트포워딩 되는건 아니고, 컨테이너의 80번 포트를 사용할 것을 나타냄. EXPOSE는 컨테이너를 생성하는 run 명령어에서 모든 노출된 컨테이너의 포트를 호스트에 퍼블리시하는 run -P와 사용
- CMD : 컨테이너가 시작될 때마다 실행할 명령어, DockerFile에서 한번만 사용 가능. 이 명령어는 컨테이너가 실행될때 자동으로 아파치 서버 실행한다는 얘기
- 이미지 생성 : `docker build -t mybuild:0.0 ./` , -t 옵션은 생성될 이미지의 이름. -t를 사용하지 않으면 16진수 형태로 이미지가 저장되므로 이름을 지정해주는게 좋다. build 명령어 끝에는 DockerFile이 저장된 경로 입력. 외부 url도 가능
  - 생성된 이미지로 컨테이너 실행 `docker run -d -P --name my server mybuild:0.0` -P옵션은 이미지에 설정된 EXPOSE의 모든 포트를 호스트에 연결하도록 설정함. EXPOSE로 지정하는 것은 컨테이너의 포트이고, 노출된 포트를 호스트에서 사용 가능한 포트(꼭 80은 아니고)에 연결. 호스트의 IP와 호스트와 연결된 포트를 통해 컨테이너의 웹 서버에 접근이 가능해짐

### 빌드 컨텍스트

**빌드 컨텍스트**(도커파일이 있는 디렉토리)를 읽어들임: 도커파일의 이미지에 파일을 추가할때 사용함.

- build 명령어의 맨 마지막에 지정된 위치에 있는 파일을 **전부** 포함. DockerFile이 위치한 곳에는 이미지 빌드에 필요한 파일만 있는 것이 바람직
- 루트 디렉토리와 같은 곳에서 이미지를 빌드하지 않도록 주의해야함
- 단순 파일뿐 아니라 하위 디렉토리를 전부 포함하게 되므로 빌드에 불필요한 파일이 포함된다면 빌드 속도가 느려지고, 호스트의 메모리를 지나치게 점유하게 됨
- .dockerIgnore라는 파일을 작성하면 빌드시 이 파일에 명시된 이름의 파일을 컨텍스트에서 제외
- 각 스텝의 명령어가 수행될때마다 **컨테이너가 하나씩 생성되며 이를 이미지로 커밋함.** 도커파일에서 명령어 한줄이 실행될때마다 이전 Step에서 생성된 이미지에 의해 새로운 컨테이너가 생성되며, DockerFile에 적힌 명령어를 수행하고 다시 새로운 이미지 레이어로 저장. (진짜 손으로 셋업하는 것처럼 + 캐시)
- 따라서 이미지의 빌드가 완료되면 DockerFile의 명령어 줄 수만큼의 레이어가 존재하게 되며, 중간에 컨테이너도 같은 수만큼 생성되고 삭제.

### **캐시 이미지 빌드**

한번 이미지 빌드를 마치고 난 뒤 다시 같은 빌드를 진행하면 이전의 이미지 빌드에서 사용했던 캐시를 사용(Using Cache)

- 이미지 빌드 중 오류가 발생했을 때는 build 명령어가 중지되며, 이미지 레이어 생성을 위해 마지막으로 생성된 임시 컨테이너가 삭제되지 않은 채로 남게 된다.
- git clone등을 사용해 이미지를 빌드하려고 할 경우, RUN에 대한 이미지 레이어를 계속 캐시로 사용하기 때문에(?) 실제 깃 저장소에서 변동이 일어나도 매번 빌드를 할 때마다 고정된 소스코드를 사용함(clone을 하려고 하지 않음. 명령어는 똑같으므로). 이럴때 --no-cache사용

### 멀티 스테이지 빌드

```sh
FROM golang
ADD main.go /root
WORKDIR /root
RUN go build -o /root/mainApp /root/main.go

FROM alpine:latest
WORKDIR /root
COPY --from=0 /root/mainApp .
CMD ["./mainApp"]
```

2개의 FROM을 통해 2개의 이미지 명시. 각각 불러온 이미지들 사이에서 파일을 공유하고 복사하는 것이 가능함. 첫번째 FROM문에서 빌드한 go 애플리케이션을 다른 이미지에 가져와서 복사함. 이렇게하면 용량을 크게 줄일 수 있는데, 반드시 필요한 실행 파일만 최종 이미지 결과물에 포함시킬 수 있음. 빌드에 golang이 전부 있을 필요보다는 필요한 것들만 있으면 좋음. -> 각 명령어 레이어마다 컨테이너를 만들어 격리하기 때문에 이런게 가능함 각 컨테이너의 존재는 명령어 실행 순서와 어느정도 독립적임

멀티스테이지 빌드를 사용하면 Dockerfile은 2개 이상의 이미지를 사용할 수 있고, 각 이미지는 먼저 FROM에 명시된 순서대로 순번이 붙음. 이를 활용하면 여러개의 이미지를 사용해 멀티 스테이지 빌드를 활용할 수 있고, 결과적으로 마지막으로 FROM된 이미지 위에서 만들어진 이미지로 나감

### 이외 다른 명령어들

- ENV : 환경변수 설정. 설정한 환경변수는 $ENV_NAME, ${ENV_NAME}의 형태로 사용이 가능. 이미지로 빌드된 컨테이너 내부에서도 사용이 가능
- VOLUME: 빌드된 이미지로 컨테이너를 생성했을 때 호스트와 공유할 컨테이너 내부의 디렉토리 설정. 컨테이너의 디렉토리 지정
- ARG: build 명령어를 실행할때 추가로 입력을 받아서 Dockerfile 내에서 사용될 변수의 값을 설정. 환경변수처럼 미리 지정하지 않고 명령어 실행 런타임에 받는 변수
- USER : 컨테이너 내에서 사용될 사용자 계정의 이름이나 UID 설정. 해당 사용자 권한으로 실행. 루트 권한이 필요하지 않다면 USER로 사용하는게 좋음
- ONBUILD: 빌드된 이미지를 기반으로 하는 다른 이미지가 DockerFile로 생성될때 실행할 명령어. 이 이미지 자체가 빌드될때는 실행되지 않다가, 이 이미지를 기반으로 새로운 이미지를 만들때 (이 이미지가 FROM이 될때) 만 실행. 즉 **자식 이미지에서만 실행** -> 가장 먼저 실행
  - 접두어 명렁어로, RUN ADD등 이미지가 빌드될 때 수행되어야 하는 각종 DockerFile의 명령어를 나중에 빌드될 이미지를 위해 미리 저장해놓는게 가능
  - 이미지가 빌드하거나 활용할 소스코드를 ONBUILD ADD로 추가해서 좀더 깔끔하게 사용가능
- STOP SIGNAL : 컨테이너 정지시 사용될 시스템 콜
- HEALTHCHECK : 이미지로부터 생성된 컨테이너에서 동작하는 애플리케이션의 상태를 체크하도록 설정. 이렇게 설정하고 빌드하면 docker ps 실행시 컨테이너에 STATUS 정보가 추가
  ```
  HEALTHCHECK --interval=1m --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1
  ```
- COPY와 ADD의 차이 : COPY는 로컬의 파일만 이미지에 추가할 수 있고, ADD는 외부 URL 및 tar파일에서도 추가할 수 잇다는 점이 다름. COPY가 ADD에 포함됨
- CMD와 entrypoint의 차이 : 커맨드가 동일하게 컨테이너가 수행할 명령을 지정한다는 점에서 같으나, entrypoint는 커맨드를 인자로 받아 사용할 수 있는 스크립트의 역할을 할 수 있다는 점에서 다름. entrypoint를 설정하면 run시에 스크립트를 넣어줄 수 있다. 맨 마지막에 있는 cmd를 인자로 삼는다
  ```sh
  docker run -i -t --entrypoint="echo" --name yes_entrypoint ubuntu:14.04 /bin/bash # /bin/bash가 cmd
  docker run -i -t --name entrypoint_sh --entrypoint="/tesh.sh" ubuntu:14.04 /bin/bash # 엔트리포인트와 쉘스크립트 연결
  # 실행할 스크립트는 컨테이너 내부에 존재해야 하며(COPY, ADD로 사전에 세팅)
  ```

## 도커 이미지의 크기, 최적화

Docker를 아무렇게나 작성하면 저장 공간을 불필요하게 차지하는 이미지나 레이어가 너무 많은 이미지가 생성된다. 최적화의 방향으로는 크게 더 작은 Base Image를 사용하는 것과, 최종 결과물에 불필요한 파일들이 포함되지 않도록 하는 것 정도. 그리고 도커의 이미지 레이어 구조를 이해해야하는 것.

- 가벼운 베이스 이미지 사용 : centos, ubuntu 기반으로 만들게 되면 도커 환경에서는 불필요한 패키지나 파일들이 포함되어있어 용량이 클수밖에 없음. 도커를 위한 OS, Alpine같은 것도 있음. 도커를 위한 리눅스 배포판도 있음 -> 근데 이거는 필요에 따라서
- Multi stage build 활용 : 빌드에만 필요한 작업을 나눠서 빌드하고, 추후 최종 컨테이너 이미지에는 포함시키지 않는 방법

```doc
FROM node:10-alpine AS build
WORKDIR /app
COPY app /app
RUN npm install && npm run build # 의존성 설치와 빌드

FROM node:10-alpine # 새로운 이미지, 딱 build 결과물만 서빙하게 된다
WORKDIR /app
RUN npm install -g webserver.local
COPY --from=build /app/build ./build
EXPOSE 3000
CMD webserver.local -d ./build
```

- 일반적으로 변경 가능성이 가장 적은 레이어부터 변경 가능성이 높은 레이어 순으로 배치하는게 팁. 그래야 분리가 용이해짐
- Dockerfile에서 RUN 명령을 개별로 실행시 실행이 끝날때마다 중간 이미지 레이어가 생성된다. 체이닝으로 RUN 명령을 실행하면 한개만 만들어지기 때문에 RUN을 체이닝하는것도 팁
  - 이전 레이어에 남아있는 변경사항의 경우에도, 파일을 삭제했을 경우에도, 이미지에 용량이 그대로 남아있을 수 있음
  - `RUN mkdir /test && allocate -l 100m /test/dummy && rm /test/dummy` : 체이닝하지 않은 경우 test/dummy만큼의 용량이 이미지에 반영됨
  - 도커의 쌓이는 레이어 구조는, 여러 이미지들이 동일한 의존성을 필요로 할때 동일한 의존성을 먼저 빌드한후 새로운 이미지를 생성함으로써 병렬적으로 용량을 아낄 수 있기도 함
- docker export, import를 사용하면 여러개의 도커 레이어를 하나로 합쳐서 사용할 수 있지만 히스토리를 삭제하기때문에 권장되지는 않음. 너무 많은 레이어가 있는 경우 모든 레이어를 하나로 줄여준다.
- 소스는 왠만해서는 최종 결과물 위에서 빌드하지 말고, 이미지에 넣는 식으로 하면 빌드 도구나 의존성이 차지하는 공간을 줄일 수 있음
- 패키지 관리자도 정리 `apt-get clean` 비대하게 깔리지는 않았는지 체크하고, 다운로드 파일이나 패키지 리스트 파일도 지워야함
- 파일 다운로드 한 후 쓰임이 끝났으면 삭제할 것 이때 체이닝 사용해서 같이 정리
- `docker history 이미지:버전 --no-trunc` 이 명령어로 레이어마다의 용량을 볼 수 있다.
- ADD보다는 wget이나 curl을 통해 외부 파일을 다운받고, 압축파일을 채이닝으로 동시에 지운다
- 개발용 패키지 유의 : node에서 npm install하면 의존성 모듈과 더불어 devdependencies까지도 설치하므로 npm install시에는 이런 의존성을 설치할 필요가 없을 수도 있다. 하나의 이미지에서 빌드나 테스트를 수행한 후 빌드파일만 옮기거나..
- docerignore 활용해서 컨텍스트에서 애초에 안올라가게끔 하기

## 심화

### 도커 스웜

### 도커 컴포즈

## References
