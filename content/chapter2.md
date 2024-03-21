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
컨테이너는 가상 머신과 마찬가지로 가상 IP 주소를 할당 받는다.  
172.17.0.x 의 IP를 순차적으로 할당
![컨테이너IP](/img/컨테이너IP.png)
- 컨테이너 셸에서 `ifconfig` 명령어로 ip 확인
- 아무런 설정을 하지 않았다면 이 컨테이너는 외부에서 접근할 수 없다.

> 외부에 컨테이너를 노출하기 위해서는 eth0의 IP와 포트를 호스트의 IP와 포트에 바인딩해야 한다.

`$ docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04`
- -p 옵션: 컨테이너의 포트와 호스트의 포트를 바인딩한다.
  - [호스트의 포트]:[컨테이너의 포트] 
  - [호스트의 특정아이피]:[호스트의 포트]:[컨테이너의 포트]
  - 여러개 가능
    - ex) `-p 192.168.0.100:7777:80 -p 3306:3306`

> 아파치 서버 설치해서 연결되는지 보기  

    apt-get update
    apt-get install apache2 -y
    service apache2 start
(AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message) 에러 무시해도됨
- 웹브라우저에서 localhost:80 을 치면 아파치 페이지가 뜬다. (172.17.0.2:80 아님; 조심)
![아파치연결](/img/아파치연결성공.png)

---


### 컨테이너 애플리케이션 구축
대부분의 서비스는 여러 에이전트나 데이터베이스 등과 연결되어 동작하는게 일반적.  
이런 서비스를 **컨테이너화** 할 때 여러 개의 애플리케이션을 한 컨테이너에 설치할 수도 있다.  

> 하지만 컨테이너에 애플리케이션을 **하나만 동작**시키면
- 독립성 보장
- 애플리케이션 버전 관리, 소스코드 모듈화 쉬워짐
- 도커 공식 홈페이지에서도 권장하는 구조
- 한 컨테이너에 프로세스 하나만 실행하는 것이 **도커의 철학**


> 데이터베이스 + 워드프레스 웹 서버 컨테이너 연동

	$ docker run -d \
	> --name wordpressdb \
	> -e MYSQL_ROOT_PASSWORD=password \
	> -e MYSQL_DATABASE=wordpress \
	> mysql:5.7

: mysql 데이터베이스 컨테이너 생성  

	$ docker run -d \
	> -e WORDPRESS_DB_HOST=mysql \
	> -e WORDPRESS_DB_USER=root \
	> -e WORDPRESS_DB_PASSWORD=password \
	> --name wordpress \
	> --link wordpressdb:mysql \
	> -p 80 \
	> wordpress

: 워드프레스 웹 서버 컨테이너 생성
- -p 80 : 호스트의 포트중에 하나와 컨테이너의 80번 포트가 연결됨(호스트 포트는 뭔지 모름)  
`docker ps` 로 확인  
![호스트의아이피](/img/호스트의ip확인.png)  

`dorker port wordpress` 로도 확인 가능 (port만)  

    $ docker port wordpress
    80/tcp -> 0.0.0.0:1110

![워드프레스연결](/img/워드프레스연결.png)


- `-d` 옵션: Detached 모드로 컨테이너를 실행
  - -i -t가 컨테이너 내부로 진입하도록 attach 가능한 상태로 실행하는거라면,  
	  -d는 Detached 모드로 컨테이너를 실행합니다. 
  - 컨테이너 내부에서 프로그램이 하나의 터미널을 차지하는 포그라운드로 실행되어야 한다.
    - ubuntu, centos는 `-d` 옵션으로 컨테이너를 `run`하면 포그라운드로 `/bin/bash` 가 실행되는데 사용자 입력을 받지 않으므로 바로 종료되고 포그라운드로 실행되고있는 프로그램이 없으므로 컨테이너가 종료됨
    - mysql은 mysqld라는 하나의 터미널을 차지하는 프로그램이 포그라운드로 실행됨.
  - 컨테이너를 백그라운드에서 동작하는 애플리케이션으로써 실행되도록 합니다.  


		컨테이너는 각기 하나의 모니터를 기본적으로 가지고 있다고 생각하면 이해하기 쉽습니다.
		여러개의 터미널을 열어 동일한 컨테이너에 동시에 attach해보면 이를 바로 알 수 있습니다.
		(마치 듀얼모니터 연결 후 디스플레이 옵션 > "복제" 로 설정한거 같음)


- `-e` 옵션: 컨테이너 내부의 환경변수 설정
  - `echo $MYSQL_ROOT_PASSWORD` -> password 출력

- `exec` 명령어: 컨테이너 내부에서 명령어를 실행한 뒤 그 결괏값을 반환 받는다.
  - `-i -t` 옵션을 추가하여 `/bin/bash` 를 실행하면 상호 입출력이 가능
    - `docker exec -i -t wordpressdb /bin/bash`
  - `exec`로 컨테이너에 들어와서 exit를 해도 컨테이너가 종료되지않는다.
    - mysqld 프로세스는 여전히 포그라운드로 동작중이기때문

- `--link` 옵션: 컨테이너에 별명(alias)으로 접근하도록 설정     
  - `--link wordpressdb:mysql`  
  워드프레스 웹 서버 컨테이너에서 mysql 이라는 호스트명으로 접근할 수 있다.
  - deprecated 되었다고 함.   
	**도커 브릿지(bridge)** 네트워크를 사용하면 동일한 기능을 손쉽게 사용할 수 있으므로 2.2.7.2절 ㄱㄱ

---

### 도커 볼륨
도커 이미지로 컨테이너를 생성하면 이미지는 읽기 전용이 되며 컨테이너의 변경 사항만 별도로 저장해서 각 컨테이너의 정보를 보존한다.  
이미 생성된 이미지는 어떠한 경우로도 변경되지 않으며, 컨테이너 계층에 원래 이미지에서 변경된 파일시스템 등을 저장합니다.  

예) **이미지**에 mysql을 실행하는 데 필요한 애플리케이션 파일이 들어있다면 **컨테이너** 계층에는 워드프레스에서 쓴 로그인 정보나 게시글 등과 같이 데이터베이스를 운용하면서 쌓이는 데이터가 저장됩니다.
> 치명적인 단점: 컨테이너를 삭제하면 컨테이너 계층에 저장돼 있던 데이터베이스의 정보도 삭제됨.

도커의 컨테이너는 생성과 삭제가 매우 쉬우므로 실수로 컨테이너를 삭제하면 데이터를 복구할 수 없게 됨.  

> 이를 방지하기 위해 가장 활용하기 쉬운 방법이 바로 도커 **볼륨**
- 호스트와 볼륨을 공유하는 방법
- 볼륨 컨테이너를 활용하는 방법
- 도커가 관리하는 볼륨을 생성하는 방법  

이 있다.

#### 호스트 볼륨 공유
---
일단 mysql 데이터베이스 컨테이너와 워드 프레스 웹 서버 컨테이너를 생성해보자.  


mysql 컨테이너 생성  

	$ docker run -d \
	> --name wordpressdb_host \
	> -e MYSQL_ROOT_PASSWORD=password \
	> -e MYSQL_DATABASE=wordpress \
	> -v C:/dockerVolume/wordpress_db:/var/lib/mysql \
	> mysql:5.7


- `-v` 옵션: 디렉터리를 공유하는 옵션 
  - [호스트의 공유 디렉터리]:[컨테이너의 공유 디렉터리]  

> 컨테이너의 `/var/lib/mysql`디렉터리는 MySQL이 데이터베이스의 데이터를 저장하는 기본 디렉터리입니다.  
> 미리 /home/wordpress_db 디렉터리를 생성하지 않았어도 도커가 자동으로 생성해줍니다.

> 윈도우에서 git bash를 이용할 경우 `-v /home/wordpress_db:/var/lib/mysql `  
> docker: Error response from daemon: mkdir C:\Program Files\Git\home: Access is denied.  
> 에러 발생... 관리자권한도 안통함.. 그냥 풀경로 적어줘야될듯..

---

워드프레스 컨테이너 생성

	$ docker run -d \
	> -e WORDPRESS_DB_PASSWORD=password \
	> --name wordpress_hostvolume \
	> --link wordpressdb_hostvolume:mysql \
	> -p 80 \
	> wordpress

> `ls c:/dockerVolume/wordpress_db` 볼륨 연결 확인  

![볼륨연결확인](/img/볼륨연결확인.png)
![볼륨연결확인](/img/볼륨연결확인2.png)

> 컨테이너를 삭제해도 해당 디렉터리의 데이터는 그대로 남아 있는 것을 확인할 수 있다.  
> 컨테이너의 `/var/lib/mysql` 디렉터리는 호스트의 `C:/dockerVolume/wordpress_db` 디렉터리와 동기화 되는 것이 아니라  
 **완전히 같은 디렉터리**이다.  

	원래 존재하지 않던 디렉토리를 -v 옵션으로 호스트에 디렉터리를 생성하고
	결과적으로 컨테이너의 파일이 호스트의 디렉터리로 복사된 것이다.

그렇다면 호스트에 이미 디렉터리와 파일이 존재하고 컨테이너에도 존재할 때 두 디렉터리를 공유하면 어떻게 될까?  

	$ docker run -it --name volume_dummy alicek106/volume_test
	root@84bf49a80147:/# ls /home/testdir_2/
	test

- /home/testdir_2/ 디렉터리에 test 파일이 있는 컨테이너이다.

![덮어씌워진디렉터리](/img/덮어씌워진디렉터리.png)

-  호스트의 파일들로 덮어씌워진 컨테이너의 디렉터리
> 정확히 말하면 -v 옵션을 통한 호스트 볼륨 공유는 호스트의 디렉터리를 컨테이너의 디렉터리에 **`마운트`** 한다.  

---
---
---

디렉터리 단위의 공유뿐 아니라 단일 파일 단위의 공유도 가능하며, 동시에 여러 개의 -v 옵션을 쓸 수도 있습니다.  

	echo hello >> c:/dockerVolume/hello && echo hello2 >> c:/dockerVolume/hello2  

	$ docker run -it --name file_volume -v c:/dockerVolume/hello:/hello -v c:/dockerVolume/hello2:/hello2 ubuntu:14.04
	root@9cee81a2fb40:/# cat hello && cat hello2
	hello
	hello2

- echo '문자열' >> 파일 :  파일에 문자열을 추가 (파일이 없으면 생성하고 추가)
- cat 파일 : 파일의 내용을 출력  
---
---
---

#### 볼륨 컨테이너
---
볼륨을 사용하는 두번째 방법은 -v 옵션으로 볼륨을 사용하는 컨테이너를 다른 컨테이너와 공유하는 것입니다.  
컨테이너를 생성할 때 --volumes-from 옵션을 설정하면 -v 또는 --volume 옵션을 적용한 컨테이너의 볼륨 디렉터리를 공유할 수 있습니다.  
그러나 이는 직접 볼륨을 공유하는 것이 아닌 -v 옵션을 적용한 컨테이너를 통해 공유하는 것입니다.

아래 예제는 위에서 생성한 volume_overide 컨테이너에서 볼륨을 공유받는 경우입니다.  
앞에서 생성한 `volume_overide 컨테이너`는 /home/testdir_2 디렉터리를 호스트와 공유하고 있으며,  
이 컨테이너를 `볼륨 컨테이너`로서 volumes_from_container 컨테이너에 다시 공유하는 것입니다.
![볼륨컨테이너공유](/img/볼륨컨테이너공유.png)

--volumes-from 옵션을 적용한 컨테이너와 볼륨 컨테이너 사이의 관계를 나타내면 다음 그림과 같습니다.

![볼륨컨테이너구조](/img/볼륨컨테이너구조.png)

여러 개의 컨테이너가 동일한 컨테이너에 --volumes-from 옵션을 사용함으로써 볼륨을 공유해 사용할 수도 있습니다.  
이러한 구조를 활용하면 호스트에서 볼륨만 공유하고 별도의 역할을 담당하지 않는 일명 `'볼륨 컨테이너'` 로서 활용하는 것도 가능합니다.  
즉, 볼륨을 사용하려는 컨테이너에 `-v` 옵션 대신 `--volumes-from` 옵션을 사용함으로써 볼륨 컨테이너에 연결해 데이터를 간접적으로 공유받는 방식입니다.  

---

#### 도커 볼륨

도커 자체에서 제공하는 볼륨 기능  

`docker volume create` 명령으로 볼륨 생성

	$ docker volume create --name myvolume
	myvolume

`docker volume ls` 명령으로 볼륨 리스트 확인

	$ docker volume ls
	DRIVER    VOLUME NAME
	local     myvolume

볼륨을 생성할 때 플러그인 드라이버를 설정해 여러 종류의 스토리지 백엔드를 쓸 수 있다.
- `nfs`(Network File System), `aws efs`(Amazon Elastic File System), `azure file` 등의 다양한 플러그인 드라이버가 있다.

예제는 기본적으로 제공되는 드라이버인 `local`을 사용.  

	$ docker run -i -t --name myvolume_1 \
	> -v myvolume:/root/ \
	> ubuntu:14.04
	root@6d8eaa7d9951:/# echo hello, volume! >> /root/volume

- /root 디렉터리에 `volume` 파일 생성  

컨테이너를 빠져나와 새로운 컨테이너를 생성해 볼륨안의 데이터가 공유되는지 확인해보자.

	$ docker run -it --name myvolume_2 \
	> -v myvolume:/root/ \
	> ubuntu:14.04
	root@311623fc0e9a:/# cat /root/volume
	hello, volume!

- 결과를 보면 같은 파일인 volume이 존재하는 것을 확인할 수 있다.


docker volume 명령어로 생성한 볼륨은 아래 그림과 같은 구조로 활용된다.
![도커볼륨사용구조](/img/볼륨사용구조.png)

docker inspect 명령어를 사용하면 myvolume 볼륨이 실제로 어디에 저장되는지 알 수 있습니다.

	$ docker inspect --type volume myvolume
	[
			{
					"CreatedAt": "2024-03-20T03:19:00Z",
					"Driver": "local",
					"Labels": null,
					"Mountpoint": "/var/lib/docker/volumes/myvolume/_data",
					"Name": "myvolume",
					"Options": null,
					"Scope": "local"
			}
	]

- `Driver` : 볼륨이 스는 드라이버
- `Labels` : 볼륨을 구분하는 라벨 (사용자 정의 메타데이터)
- `Mountpoint` : 해당 볼륨이 실제로 호스트의 어디에 저장됐는지를 의미
  - Windows 운영체제에서는 wsl 내에 존재
  - Windows에서 해당 디렉터리 확인하고 싶으면 참고 [Windows운영체제에서 /var/lib/docker 찾기](https://velog.io/@ette9844/Windows10-%EC%97%90%EC%84%9C-varlibdocker-%EC%B0%BE%EA%B8%B0)


`docker volume create` 명령을 별도로 수행하지 않고 `-v` 옵션만으로 수행하도록 설정 할 수 있다.

	$ docker run -i -t --name volume_auto \
	> -v /root ubuntu:14.04  
해당 명령 수행 후

	$ docker volume ls
	DRIVER    VOLUME NAME
	local     e90870e5bae2e8b4b7c161ca3350ac70c50c3f1fff6eeb1a7956d09e41052111
	local     myvolume

목록을 조회하면 이름이 무작위 16진수 형태인 볼륨이 자동으로 생성된 것을 볼 수 있다.

 - `docker container inspect volume_auto` 로 어떤 볼륨이 mount됐는지 확인 가능.(Mounts, Source 항목)
  
도커 볼륨을 사용하고 있는 컨테이너를 삭제해도 볼륨이 자동으로 삭제되지는 않는다.
 - `docker volume prune` 명령어로 사용되지 않는 볼륨을 한꺼번에 삭제 가능

---

`-v` 옵션 대신 `--mount` 옵션을 사용해도 기능은 같다.  
도커 볼륨 마운트(type=volume) : `--mount type=volume,source=myvolume,target=/root`  
호스트의 경로 마운트(type=bind) : `--mount type=bind,source=c:/dockerVolume/wordpress_db,target=/home/testdir`

---

### 도커 네트워크
#### 도커 네트워크 구조

![도커네트워크구조](/img/도커네트워크구조.png)

- `veth` : 호스트에 생성되는 네트워크 인터페이스
- `docker0` : `veth`인터페이스와 `호스트의 eth0`을 이어주는 브리지  
  
---

도커는 각 컨테이너에 외부와의 네트워크를 제공하기 위해 컨테이너마다 가상 네트워크 인터페이스를 호스트에 생성하며 이 인터페이스의 이름은 `veth`로 시작한다.  
 - 컨테이너가 생성될 때 도커 엔진이 자동으로 생성한다.
 - `veth`의 v는 virtual을 의미한다. 즉 virtual ethernet 이라는 의미.
 - 도커가 설치된 호스트(리눅스)에서 ifconfig로 컨테이너 수만큼 veth인터페이스가 생성된다.

veth인터페이스 뿐만 아니라 `docker0` 이라는 브리지도 존재하는데 docker0 브리지는 각 veth 인터페이스와 바인딩돼 호스트의 eth0 인터페이스와 이어주는 역할을 한다.

---

> 윈도우에서는 Hyper-V Virtual Ethernet Default Switch IP를 사용중.  
> Docker for Desktop은 WSL(Windows Subsystem for Linux)에서 돌아가고있어서..  
> docker0이나 veth를 직접적으로 확인은 불가하다...

`brctl show docker0` 명령어를 이용해 docker0 브리지에 veth이 실제로 바인딩됐는지 알 수 있다.  
  

---

#### 도커 네트워크 기능

컨테이너를 생성하면 기본적으로 docker0 브리지를 통해 외부와 통신할 수 있는 환경을 사용할 수 있지만 사용자의 선택에 따라 여러 네트워크 드라이버를 쓸 수도 있습니다.

> 도커가 자체적으로 제공하는 네트워크 드라이버  
  - 브리지(bridge), 호스트(host), 논(none), 컨테이너(container), 오버레이(overlay)  
  
> 서드파티 플러그인 솔루션
  - weave, flannel, openvswitch 등..  


> 네트워크 목록 확인

	$ docker network ls
	NETWORK ID     NAME      DRIVER    SCOPE
	37f7ac294c2e   bridge    bridge    local
	d6cd99e1a051   host      host      local
	d930b9d027a2   none      null      local

> 네트워크의 자세한 정보 확인 (예시는 bridge)

	$ docker network inspect bridge
	[
			{
					"Name": "bridge",
					"Id": "37f7ac294c2e98186ca4b12db7600b3c2390b91f537732019153274ae061818e",
					"Created": "2024-03-21T00:48:33.798977483Z",
					"Scope": "local",
					"Driver": "bridge",
					"EnableIPv6": false,
					"IPAM": {
							"Driver": "default",
							"Options": null,
							"Config": [
									{
											"Subnet": "172.17.0.0/16",
											"Gateway": "172.17.0.1"
									}
							]
					},
					"Internal": false,
					"Attachable": false,
					"Ingress": false,
					"ConfigFrom": {
							"Network": ""
					},
					"ConfigOnly": false,
					"Containers": {},
					"Options": {
							"com.docker.network.bridge.default_bridge": "true",
							"com.docker.network.bridge.enable_icc": "true",
							"com.docker.network.bridge.enable_ip_masquerade": "true",
							"com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
							"com.docker.network.bridge.name": "docker0",
							"com.docker.network.driver.mtu": "1500"
					},
					"Labels": {}
			}
	]

- 브리지 네트워크는 컨테이너를 생성할 때 자동으로 연결되는 docker0 브리지를 활용
- 172.17.0.x 대역을 순차적으로 할당 (Config항목의 Subnet, Gateway 참고)
- 브리지 네트워크를 사용중인 컨테이너의 목록 확인가능
---

#### 브리지 네트워크
기본적으로 존재하는 docker0을 사용하는 브리지 네트워크가 아닌 새로운 브리지 타입의 네트워크를 생성할 수 있다.

> 새로운 브리지 네트워크 생성  

	$ docker network create --driver bridge mybridge
	793c0c261fe1ca13f251f9815e528870538405df850c14a8d73627039c345913

> --net 옵션으로 네트워크 설정  

	$ docker run -i -t --name mynetwork_container --net mybridge ubuntu:14.04

> 새로운 IP 대역이 할당된 것 확인 ( 172.18 대역 )  

	root@0f28ee850579:/# ifconfig
	eth0      Link encap:Ethernet  HWaddr 02:42:ac:12:00:02
						inet addr:172.18.0.2  Bcast:172.18.255.255  Mask:255.255.0.0
						UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
						RX packets:12 errors:0 dropped:0 overruns:0 frame:0
						TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
						collisions:0 txqueuelen:0
						RX bytes:1112 (1.1 KB)  TX bytes:0 (0.0 B)

	lo        Link encap:Local Loopback
						inet addr:127.0.0.1  Mask:255.0.0.0
						UP LOOPBACK RUNNING  MTU:65536  Metric:1
						RX packets:0 errors:0 dropped:0 overruns:0 frame:0
						TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
						collisions:0 txqueuelen:1000
						RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

> 유동적으로 네트워크 연결 및 끊기 가능

	$ docker network disconnect mybridge mynetwork_container
	$ docker network connect mybridge mynetwork_container

 - none 네트워크, host 네트워크 같은 특별한 네트워크 모드에서는 disconnect 사용 불가능
   - 특정 IP 대역을 갖는 브리지, 오버레이 네트워크 등에서만 사용가능하다.

> disconnect 시 eth0 네트워크 인터페이스가 사라진 것을 확인 할 수 있다.

	root@0f28ee850579:/# ifconfig
	lo        Link encap:Local Loopback
	          inet addr:127.0.0.1  Mask:255.0.0.0
	          UP LOOPBACK RUNNING  MTU:65536  Metric:1
	          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
	          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
	          collisions:0 txqueuelen:1000
	          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)		

> 서브넷, 게이트웨이, IP할당 범위 임의로 설정가능  
> 
	$ docker network create --driver=bridge \
	> --subnet=172.72.0.0/16 \
	> --ip-range=172.72.0.0/24 \
	> --gateway=172.72.0.1 \
	> my_custom_network
	b0d3bbafd43cee440eea462a3575f60e5837d0d4377d8805428669a2673143bc
---
#### 호스트 네트워크
네트워크를 호스트로 설정하면 호스트의 네트워크 환경을 그대로 쓸 수 있다.  
> `--net host` 옵션 사용  

	docker run -i -t --name network_host --net host ubuntu:14.04

- 컨테이너 내부의 애플리케이션을 별도의 포트 포워딩 없이 바로 서비스할 수 있다.  
    - ex) 호스트 모드를 쓰는 컨테이너에서 아파치 웹 서버를 구동한다면 호스트의 IP와 컨테이너의 아파치 웹 서버 포트인 80으로 바로 접근할 수 있다.
    - 안되는... 질문필요
  


---
#### 논 네트워크
none은 말 그대로 아무런 네트워크를 쓰지 않는 것.

> `--net none` 옵션 사용

	docker run -i -t --name network_none --net none ubuntu:14.04

- ifconfig로 확인하면 eth0이 없고 lo 인터페이스밖에 없는것을 알 수 있다.

---
#### 컨테이너 네트워크
다른 컨테이너의 네트워크 네임스페이스 환경을 공유할 수 있다.  
> 공유되는 속성  

- 내부IP
- 네트워크 인터페이스의 맥(MAC) 주소

> `--net container:[컨테이너이름]` 옵션 사용

	$ docker run -i -t -d --name network_container_1 ubuntu:14.04
	29310ea7d0996f0778e45471cb46517631ea0a0cd32c9f9c03dd6cdd4bed1544

	$ docker run -i -t -d --name network_container_2 --net container:network_container_1 ubuntu:14.04
	1551f28dd1036db7e82c60f7cf5a44cc66a208499ec6c6b8672616056418c14a

- 다른 컨테이너의 네트워크 환경을 공유하면 내부 IP를 새로 할당받지 않으며, veth 도 생성되지 않는다.
- `network_container_2` 컨테이너의 네트워크 관련된 사항은 전부 `~_1`과 같게 설정된다.
---

-i -t -d 옵션을 함께 사용하면 컨테이너 내부에서 셸을 실행하지만 내부로 들어가지 않으며 컨테이너도 종료되지 않는다.  
테스트용으로 컨테이너를 생성할 때 유용하게 쓸 수 있다.

---

  
#### 브리지 네트워크와 --net-alias

브리지 타입의 네트워크와 run 명령어의 --net-alias 옵션을 함께 쓰면 특정 호스트 이름으로 컨테이너 여러 개에 접근할 수 있다.

> `--net-alias chanchan`로 컨테이너 생성

	$ docker run -i -t -d --name network_alias_container1 --net mybridge --net-alias chanchan ubuntu:14.04
	1b076abb7ef526187c841dbf48d87ca509b794f0fa2158bc804c06b633e4a217

	$ docker run -i -t -d --name network_alias_container2 --net mybridge --net-alias chanchan ubuntu:14.04
	26c0fec6e8de117d3dc3c0b7493b7a258b06f31bbce7d44a9d18b2bff27a25c1

	$ docker run -i -t -d --name network_alias_container3 --net mybridge --net-alias chanchan ubuntu:14.04
	e0b5517def665295afb0fac841c86be3f016d53f16f57b7a2c5c61db5b10d172

> `ping` 요청 전송

	$ docker run -i -t --name network_alias_ping --net mybridge ubuntu:14.04
	root@063d10416c77:/# ping -c 1 chanchan
	PING chanchan (172.18.0.4) 56(84) bytes of data.
	64 bytes from network_alias_container3.mybridge (172.18.0.4): icmp_seq=1 ttl=64 time=0.067 ms

	--- chanchan ping statistics ---
	1 packets transmitted, 1 received, 0% packet loss, time 0ms
	rtt min/avg/max/mdev = 0.067/0.067/0.067/0.000 ms
	root@063d10416c77:/# ping -c 1 chanchan
	PING chanchan (172.18.0.2) 56(84) bytes of data.
	64 bytes from network_alias_container1.mybridge (172.18.0.2): icmp_seq=1 ttl=64 time=0.094 ms

	--- chanchan ping statistics ---
	1 packets transmitted, 1 received, 0% packet loss, time 0ms
	rtt min/avg/max/mdev = 0.094/0.094/0.094/0.000 ms
	root@063d10416c77:/# ping -c 1 chanchan
	PING chanchan (172.18.0.3) 56(84) bytes of data.
	64 bytes from network_alias_container2.mybridge (172.18.0.3): icmp_seq=1 ttl=64 time=0.137 ms

- 컨테이너 3개의 IP로 각각 ping이 전송된 것 확인
- 매번 달라지는 IP를 결정하는 것은 라운드 로빈 방식
- 도커 엔진에 내장된 DNS가 `chanchan` 이라는 호스트 이름을 `--net-alias`옵션으로 설정한 컨테이너로 변환(resolve)하기 때문

> 브리지 네트워크의 --net-alias와 도커 DNS의 작동 구조
![작동구조](/img/브리지네트워크와alias작동구조.png)

도커의 DNS는 호스트 이름으로 유동적인 컨테이너를 찾을 때 주로 사용된다.  

- `--link`
  - 컨테이너의 IP가 변경돼도 별명으로 컨테이너를 찾을 수 있게 DNS에 의해 자동으로 관리된다.  
  (디폴트 브리지 네트워크의 컨테이너 DNS로 관리됨)

- `--net-alias`
  - 도커는 사용자가 정의한 브리지 네트워크에 사용되는 내장 DNS 서버를 가짐
  - 컨테이너의 IP는 DNS 서버에 alias로 지정한 호스트이름으로 등록됨

> dig명령어로 chanchan 호스트 이름 변환 IP 확인

	apt-get update
	apt-get install dnsutils  

---

	root@063d10416c77:/# dig chanchan

	; <<>> DiG 9.9.5-3ubuntu0.19-Ubuntu <<>> chanchan
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11138
	;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

	;; QUESTION SECTION:
	;chanchan.                      IN      A

	;; ANSWER SECTION:
	chanchan.               600     IN      A       172.18.0.2
	chanchan.               600     IN      A       172.18.0.3
	chanchan.               600     IN      A       172.18.0.4

	;; Query time: 1 msec
	;; SERVER: 127.0.0.11#53(127.0.0.11)
	;; WHEN: Thu Mar 21 07:10:36 UTC 2024
	;; MSG SIZE  rcvd: 98

dig chanchan 명령어를 반복 입력해보면 반환되는 IP의 리스트 순서가 모두 다른 것을 알 수 있다.

---

#### MacVLAN 네트워크
호스트의 네트워크 인터페이스 카드를 가상화해 물리 네트워크 환경을 컨테이너에게 동일하게 제공  
컨테이너는 물리 네트워크상에서 가상의 맥 주소를 가지며, 해당 네트워크에 연결된 다른 장치와의 통신이 가능해진다.  
> 기본으로 할당되는 172.17.X.X 대신 네트워크 장비의 IP를 할당받기 때문
- MacVLAN을 사용하려면 적어도 1개의 네트워크 장비와 서버가 필요
- `-d macvlan`옵션은 `--driver macvlan` 과 같다.
- `--ip-range` : MacVLAN을 생성하는 호스트에서 사용할 컨테이너의 IP범위를 입력
---

### 컨테이너 로깅
#### json-file 로그 사용하기
도커는 컨테이너의 표준 출력과 에러 로그를 별도의 메타데이터 파일로 저장하며 이를 확인하는 명령어를 제공한다.

> `docker logs`명령어로 컨테이너의 표준 출력 확인하기

	$ docker run -d --name mysql_log -e MYSQL_ROOT_PASSWORD=1234 mysql:5.7
	c4fec1ef912dd71e5985b28b5cc85dfb60224dd19e17cb328d55ea43b86542f6

	확인용 mysql 컨테이너 생성 후

	$ docker logs mysql_log

	로그 확인


> 동일한 mysql 컨테이너를 생성하되, -e 옵션을 제외하고 생성해보기  

	$ docker run -d --name mysql_no_passwd mysql:5.7
	79f55f92d50ae049d8934857985c36eeb33575cabe5a8476a215d16fa2cb44d0
- 생성 후 `docker ps`명령어로 실행되지 않은 것을 확인
- `docker start` 명령어로 다시 시작해도 컨테이너는 시작되지 않는다.  
- `docker attach` 명령어도 사용 불가능
  
> 실행 안되는 컨테이너 로그 확인

	$ docker logs mysql_no_passwd
	2024-03-21 08:31:15+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.44-1.el7 started.
	2024-03-21 08:31:15+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
	2024-03-21 08:31:15+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.44-1.el7 started.
	2024-03-21 08:31:15+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
			You need to specify one of the following as an environment variable:
			- MYSQL_ROOT_PASSWORD
			- MYSQL_ALLOW_EMPTY_PASSWORD
			- MYSQL_RANDOM_ROOT_PASSWORD  

> `--tail` 옵션으로 마지막 로그 줄 부터 출력할 줄의 수 설정하기  

	$ docker logs --tail 4 mysql_no_passwd
			You need to specify one of the following as an environment variable:
			- MYSQL_ROOT_PASSWORD
			- MYSQL_ALLOW_EMPTY_PASSWORD
			- MYSQL_RANDOM_ROOT_PASSWORD

> `--since` 옵션으로 특정시간 이후의 로그를 확인하기 (유닉스시간)  

	$ docker logs --since 1710977400 mysql_no_passwd
	2024-03-21 08:31:15+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.44-1.el7 started.
	2024-03-21 08:31:15+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
	2024-03-21 08:31:15+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.44-1.el7 started.
	2024-03-21 08:31:15+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
			You need to specify one of the following as an environment variable:
			- MYSQL_ROOT_PASSWORD
			- MYSQL_ALLOW_EMPTY_PASSWORD
			- MYSQL_RANDOM_ROOT_PASSWORD

- 1710977400 : Thu Mar 21 2024 08:30:00 UTC+0900 (한국 표준시)
- `-t` 옵션으로 타임스탬프를 표시할 수도 있다.
- `-f` 옵션을 써서 로그를 스트림으로 확인할 수 있다.  


> 기본적으로 컨테이너 로그는 JSON 형태로 도커 내부에 저장된다.
 - `/var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log`

> `--log-opt` 옵션으로 컨테이너 json 로그 파일의 크기, 개수 설정 가능  

	`$ docker run -it --log-opt max-size=10k --log-opt max-file=3 --name log-test ubuntu:14.04`  

- syslog, journald, fluentd, awslogs 등의 로깅 드라이버들이 있다.
- 어떠한 설정도 하지 않았다면 위와 같이 json 로그 파일로 저장한다.

--- 

로깅 드라이버는 기본적으로 json-file로 설정되지만 도커 데몬 시작 옵션에서 `--log-driver` 옵션을 써서 기본적으로 사용할 로깅 드라이버를 변경할 수 있다.  
--log-opt 같은 옵션 또한 도커 데몬에 적용함으로써 모든 컨테이너에 일괄적으로 사용할 수 있습니다.  

	DOCKER_OPTS="--log-driver=syslog"  

	DOCKER_OPTS="--log-opt max-size=10k --log-opt max-file=3"

---

#### syslog 로그
유닉스 계열 운영체제에서 로그를 수집하는 오래된 표준 중 하나이다.

syslog 로깅 드라이버는 기본적으로 로컬호스트의 syslog에 저장하므로 운영체제 및 배포판에 따라 syslog 파일의 위치를 알아야 이를 확인할 수 있다. 

