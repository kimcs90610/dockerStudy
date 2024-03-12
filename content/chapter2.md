# 2장 도커 엔진
## 도커 이미지와 컨테이너
도커 엔진에서 사용하는 기본 단위는 이미지와 컨테이너이며, 이 두가지가 도커 엔진의 핵심
### 도커 이미지
- 컨테이너를 생성할 때 필요한 요소
- 가상머신을 생성할때 사용하는 iso파일과 비슷한 개념???
- 여러 개의 계층으로 된 바이너리 파일로 존재
- 도커 명령어로 내려받을 수 있음(별도 설치 X)

> [저장소 이름]/[이미지 이름]:[태그]

    docker.io/ubuntu:14.04 (도커 허브 저장소 명시)
    ubuntu:latest (저장소 생략)

- **저장소(Repository)** 이름: 이미지가 저장된 장소.
  - 생략할 경우 기본적으로 제공되는 이미지 저장소인 도커 허브(Docker Hub)의 공식 이미지를 뜻한다. (도커허브: 약간 npm같은 개념인 듯)
	- docker.io(도커허브) 외에 gcr.io(구글 클라우드 컨테이너 레지스트리)도 있음 
- **이미지** 이름: 이미지가 어떤 역할을 하는지 나타냄.
  - 위 예시는 우분투 컨테이너를 생성하기 위한 이미지라는 것을 알 수 있음.
- **태그** : 버전을 명시, 생략시 latest

### 도커 컨테이너
- 이미지로 컨테이너를 생성하면 해당 이미지의 목적에 맞는 파일시스템과 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성된다. 이것이 바로 **도커 컨테이너**
- 컨테이너는 이미지를 읽기 전용으로 사용.
  - 이미지에서 변경된 사항만 컨테이너 계층에서 저장하므로 원래 이미지에는 아무 영향X
- 생성된 컨테이너에서 어떤 애플리케이션을 설치하거나 삭제해도 다른 컨테이너와 호스트에 영향 X
  - 우분투 이미지로 두개의 컨테이너를 생성한 후, 각각 MySQL, 아파치 웹서버를 설치해도 각 컨테이너에 영향X 호스트에도 영향X

## 도커 컨테이너 다루기
도커 컨테이너의 기초적인 사용법

### 컨테이너 생성
- 도커 버전 확인

		$ docker -v
		Docker version 25.0.3, build 4debf41

- 컨테이너 생성 + 실행

		$ docker run -i -t ubuntu:latest
		Unable to find image 'ubuntu:latest' locally
		latest: Pulling from library/ubuntu
		bccd10f490ab: Pull complete
		Digest: sha256:77906da86b60585ce12215807090eb327e7386c8fafb5402369e421f44eff17e
		Status: Downloaded newer image for ubuntu:latest
		root@154c2226de70:/#

	- -i -t 옵션: 컨테이너와 상호 입출력을 가능하게 함 ( 2.2.5 참고 )
		- -i : 상호입출력
		- -t : tty(TeleTYpewriter) 활성화
		- 두 옵션 모두 사용해야 셸을 정상적으로 사용 가능
	- ubuntu 이미지가 로컬에 존재하지않으므로 도커 허브에서 자동으로 내려받음
	- `root@154c2226de70:/#` 셸의 사용자와 호스트의 이름이 변경된 것이 컨테이너 내부에 들어와 있다는 것을 나타냄
	- 컨테이너의 기본사용자는 **root** 이고 호스트 이름은 **154c2226de70** 무작위 16진수 해시값

- 컨테이너 빠져나오기

		root@154c2226de70:/# exit
		exit
	- 컨테이너를 정지 시키고 빠져나옴.
	- 정지하지 않고 빠져나오기 : Ctrl + P,Q
		- 애플리케이션 개발 목적일때 많이 씀 

- 이미지 내려받기

		$ docker pull centos:7
		7: Pulling from library/centos
		Digest: sha256:be65f488b7764ad3638f236b7b515b3678369a5124c47b8d32916d6487418ea4
		Status: Image is up to date for centos:7
		docker.io/library/centos:7
	- centos:7 이미지를 내려 받음

- 도커 엔진에 존재하는 이미지의 목록 확인

		$ docker images
		REPOSITORY                 TAG       IMAGE ID       CREATED        SIZE
		ubuntu                     latest    ca2b0f26964c   13 days ago    77.9MB
		docker/welcome-to-docker   latest    c1f619b6477e   4 months ago   18.6MB
		centos                     7         eeb6ee3f44bd   2 years ago    204MB
		ubuntu                     14.04     13b66b487594   2 years ago    197MB
	- 방금 내려받은 centos:7 이미지와, ubuntu:14.04 가 있는 것 확인 가능

- 컨테이너 생성

		$ docker create -i -t --name chansooContainer centos:7
		a77dcb2522c3bcdaef454090517ff865ef8687ddc725a07fe347c982233b64f9
	- 무작위 16진수 해시값은 컨테이너의 고유 ID
	- 너무 길어서 일반적으로 앞의 12자리만 사용
	- `docker inspect` 명령어로 ID 확인 가능
	- run 명령어와는 다르게 컨테이너 내부로 들어가지 않는다. (생성만 하니까)

- 컨테이너 시작

		$ docker start chansooContainer
		chansooContainer

- 컨테이너 내부로 들어가기

		$ docker attach chansooContainer
		[root@a77dcb2522c3 /]#

> run 명령어
>> `docker pull` (이미지없을때) > `docker create` > `docker start` > `docker attach` (-i -t옵션을 사용했을 때)

> create 명령어
>> `docker pull` (이미지없을때) > `docker create`

	보통 run 명령어를 더 많이 사용함.

	컨테이너를 대상으로 하는 모든 명령어는 컨테이너의 이름 대신 ID를 쓸 수 있다.
	(id의 앞 2~3자만 입력해도 됨)
	(단, 중복시 에러 발생하므로 적절히 4~5자까지 써주자)
	ex) docker start a77dc 

### 컨테이너 목록 확인
생성한 컨테이너의 목록 확인

- `docker ps` 정지되지 않은 컨테이너의 목록 

![docker-ps](/img/docker-ps.png)

- `docker ps -a` 모든 컨테이너 목록

![docker-ps-a](/img/docker-ps-a.png)

CONTAINER ID: 컨테이너에게 자동으로 할당되는 고유한 ID  
IMAGE: 컨테이너를 생성할 때 사용된 이미지의 이름  
COMMAND: 컨테이너가 시작될 떄 실행될 명령어  
- 대부분의 이미지에 미리 내장돼 있음
- centos와 ubuntu는  /bin/bash 커맨드가 내장돼 있다.
- run이나 create 맨 뒤에 입력하면 커맨드를 덮어 씌울 수 있다.  
ex) `docker run -i -t ubuntu:14.04 echo hello world!!`

CREATED: 컨테이너가 생성되고 난 뒤 흐른 시간  
STATUS: 컨테이너의 상태  
- 실행중: Up
- 종료: Exited
- 일시중지: Pause  

PORTS: 컨테이너가 개방한 포트와 호스트에 연결한 포트를 나열
- 컨테이너를 생성할 때 외부에 노출하도록 설정하지 않았으므로 아무것도 출력 X  

NAMES: 컨테이너의 고유한 이름
- `--name` 옵션으로 이름을 설정하지 않으면 도커 엔진이 임의로 이름을 설정함
- 변경 가능  
ex) `docker rename chansooContainer renameContainer`

### 컨테이너 삭제
삭제한 컨테이너는 복구할 수 없으므로 신중해야함  
실행중인 컨테이너는 삭제할 수 없다.

- `docker rm chansooContainer`

		$ docker rm chansooContainer
		Error response from daemon: cannot remove container "/chansooContainer": container is running: stop the container before removing or force remove

- exit 후 삭제 가능

		$ docker attach chansooContainer
		[root@a77dcb2522c3 /]# exit
		exit

		$ docker rm chansooContainer
		chansooContainer

- stop 후 삭제 가능

		$ docker stop sleepy_goldstine
		sleepy_goldstine

		$ docker rm sleepy_goldstine
		sleepy_goldstine

- -f 옵션으로 강제 삭제 가능

		$ docker rm -f vibrant_einstein
		vibrant_einstein

- 연습용으로 생성한 컨테이너가 너무 많아 일일이 삭제하기 귀찮은 경우 `prune` 명령어로 모든 컨테이너 삭제

		$ docker container prune
		WARNING! This will remove all stopped containers.
		Are you sure you want to continue? [y/N]

- `docker ps -a -q` 를 이용한 모든 컨테이너 삭제

		$ docker stop $(docker ps -a -q)
		$ docker rm $(docker ps -a -q)

	- `-q`는 컨테이너ID만 출력하는 옵션

### 컨테이너를 외부에 노출