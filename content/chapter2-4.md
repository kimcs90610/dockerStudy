## 2.4 Dockerfile

### 2.4.1 이미지를 생성하는 방법

개발한 애플리케이션을 컨테이너화하는 방법  
  1. 아무것도 존재하지 않는 이미지(우분투,CentOS 등)로 컨테이너 생성
  2. 애플리케이션을 위한 환경을 설치하고 소스코드 등을 복사해 잘 동작하는지 확인
  3. 컨테이너를 이미지로 커밋(commit)

도커는 위와 같은 일련의 과정을 손쉽게 기록하고 수행할 수 있는 빌드(build) 명령어를 제공한다.  
> Dockerfile
```
컨테이너에 설치해야 하는 패키지, 추가해야 하는 소스코드, 실행해야 하는 명령어, 셸 스크립트 등을 하나의 파일에 기록해 두면 도커는 이 파일을 읽어 컨테이너에서 작업을 수행한 뒤 이미지로 만들어낸다.
```

> 도커 허브에 이미지 대신 Dockerfile을 배포할 수 있다.  
```
배포되는 이미지를 신뢰할 수 없거나 직접 이미지를 생성해서 사용하고 싶다면 도커 허브에 올려져 있는 Dockerfile로 빌드하는 것도 하나의 방법.
```


### 2.4.2 Dockerfile 작성
도커의 명령어는 위에서 아래로 한 줄씩 차례대로 실행됨.  
한줄이 하나의 명령어  
일반적으로 대문자로 작성.

> 명령어  
- `FROM`: 생성할 이미지의 베이스가 될 이미지  
  Dockerfile을 작성할 때 반드시 한  번 이상 입력해야함  
	사용하려는 이미지가 도커에 없다면 자동으로 pull한다.

- `MAINTAINER`: 이미지를 생성한 개발자의 정보  
  도커 1.13.0 버전 이후 사용하지 않음, LABEL로 교체해 표현함  
	(LABEL maintainer "kimcs90610 \<kimcs90610@naver.com\>")

- `LABEL`: 이미지에 메타데이터를 추가.  
  메타데이터는 키:값 형태로 저장되며 docker inspect로 확인가능

- `RUN`: 이미지를 만들기 위해 컨테이너 내부에서 명령어를 실행.  
  예제에서는 apt-get update, apt-get install apache2 실행  
	Dockerfile을 이미지로 빌드하는 과정에서는 별도의 입력이 불가능하기 때문에, 별도의 입력을 받아야 하는 RUN이 있다면 build 명령어는 이를 오류로 간주하고 빌드를 종료한다.  
---
	RUN명령어에 ["bin/bash", "echo hello >> test2.html"] 과 같이 입력하면 /bin/bash 셸을 이용해 echo hello >> test2.html을 실행한다.  

	RUN ["실행 가능한 파일", "명령줄 인자 1", "명령줄 인자 2", ...] 
---

- `ADD`: 파일을 이미지에 추가한다.  
  추가하는 파일은 Dockerfile이 위치한 디렉터리인 컨텍스트(Context)에서 가져온다.  
	(예제에서는 Dockerfile이 위치한 디렉터리의 test.html 파일)  
  배열 형태로 여러개의 파일을 지정할 수있다. (["추가할 파일 이름1", "이름2" ..., "컨테이너에 추가될 위치"])  
	맨 마지막 원소가 컨테이너에 추가될 위치.  

- `WORKDIR`: 명령어를 실행할 디렉터리.   배시 셸에서 cd 명령어를 입력하는 것과 같은 기능이다.  
--- 

WORKDIR을 여러번 사용한것과 cd를 여러번 사용한것과 같다  
/var/www/html  

	WORKDIR /var
	WORKDIR www/html  

---

- `EXPOSE`: 노출될 포트 설정.  
  반드시 바인딩되는것은 아니며 -P 플래그와 함께 사용됨 (뒤에서 다시 설명)

- `CMD`: 컨테이너가 시작될 때마다 실행할 명령어 설정  
  Dockerfile에서 한 번만 사용할 수 있다.  
	배열로 사용가능 ( ["실행 가능한 파일", "명령줄 인자1", "인자2" ... ] )

> 웹 서버 이미지 생성하기

```bash
$ mkdir dockerfile && cd dockerfile
$ echo test >> test.html
--> dockerfile 디렉토리생성 -> 디렉토리로 진입 -> test.html 파일 작성

$ vi dockerfile

FROM ubuntu:14.04
MAINTAINER kimcs90610
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFOREGROUND
```
1. Dockerfile에서 사용할 베이스 이미지 ubuntu:14.04로 설정
2. 이미지 생성한 개발자 정보 kimcs90610으로 설정
3. 메타데이터 목적=연습 으로 설정
4. apt-get update, apt-get install 실행으로 아파치 웹서버 설치
5. test.html 파일을 웹서버 디렉토리와 연결
6. 작업 디렉토리를 /var/www/html로 변경 후 test2.html 파일 생성
7. 컨테이너가 사용할 포트 80으로 설정
8. 시작될 때마다 실행할 명령어 설정(아파치 포그라운드로 실행)


### 2.4.3 Dockerfile 빌드
#### 2.4.3.1 이미지생성

```bash
$ docker build -t mybuild:0.0 ./
```
- `-t`: 생성될 이미지의 이름을 설정  (미사용시 16진수 형태의 이름으로 이미지가 저장됨)
- `./`: Dockerfile이 저장된 경로 ( ./는 현재 디렉토리 )  
   외부 URL도 사용 가능

```bash
[+] Building 39.5s (11/11) FINISHED                                                                      docker:default
 => [internal] load build definition from Dockerfile                                                               0.1s
 => => transferring dockerfile: 295B                                                                               0.0s
 => [internal] load .dockerignore                                                                                  0.1s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load metadata for docker.io/library/ubuntu:14.04                                                    0.0s
 => [1/6] FROM docker.io/library/ubuntu:14.04                                                                      0.1s
 => [internal] load build context                                                                                  0.1s
 => => transferring context: 41B                                                                                   0.0s
 => [2/6] RUN apt-get update                                                                                      21.6s
 => [3/6] RUN apt-get install apache2 -y                                                                          17.1s
 => [4/6] ADD test.html /var/www/html                                                                              0.0s
 => [5/6] WORKDIR /var/www/html                                                                                    0.0s
 => [6/6] RUN ["/bin/bash", "-c", "echo hello >> test2.html"]                                                      0.3s
 => exporting to image                                                                                             0.2s
 => => exporting layers                                                                                            0.2s
 => => writing image sha256:0212aa5bcf757b9f54ad2a39a90d1dca4b1db8f6054af402eabf84db2ff45af9                       0.0s
 => => naming to docker.io/library/mybuild:0.0                                                                     0.0s

View build details: docker-desktop://dashboard/build/default/default/tzxmvjatktnkuedm6lbchts00

What's Next?
  View a summary of image vulnerabilities and recommendations → docker scout quickview
```

> 컨테이너 실행해보기
```bash
$ docker run -d -P --name myserver mybuild:0.0
85fad6a46ded8b2f1df834c467e9ef0352874d8889ae905952a04096bf559083
```
-P 옵션은 EXPOSE로 노출된 포트를 호스트에서 사용 가능한 포트에 차례로 연결하므로  
이 컨테이너가 호스트의 어떤 포트와 연결됐는지 확인할 필요가 있다.  
```bash
$ docker port myserver
80/tcp -> 0.0.0.0:32768
```
![포트연결](/img/포트연결.png)  

> 라벨 활용
```bash
$ docker images --filter "label=purpose=practice"
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
mybuild      0.0       0212aa5bcf75   2 hours ago   221MB
```
- 라벨은 도커 이미지뿐만 아니라 컨테이너, 도커 엔진 등에 메타데이터를 추가할 수 있는 유용한 방법  
- docker ps 에서 --filter 옵션 적용 가능


#### 2.4.3.2 빌드 과정 살펴보기

> 빌드 컨텍스트  

- 이미지를 생성하는 데 필요한 각종 파일, 소스코드, 메타데이터 등을 담고 있는 디렉터리.  
- Dockerfile이 위치한 디렉터리가 빌드 컨텍스트가 된다.
- Dockerfile에서 빌드될 이미지에 파일을 추가할 때 사용됨.  (`ADD`, `COPY`)
- 위 예제는 빌드경로를 `./` 로 지정하여 test.html 파일을 **빌드 컨텍스트**에 추가,  
  `ADD`를 이용하여 test.html을 이미지에 추가함.
- 컨텍스트는 `build` 명령어의 맨 마지막에 지정된 위치에 있는 파일을 전부 포함한다.  
  ( 루트 디렉토리(/) 를 지정하지 않도록 주의해야 함 )
- `.gitignore`와 유사한 `.dockerignore`를 이용하여 제외할 파일 설정 가능

> Dockerfile을 이용한 컨테이너 생성과 커밋

`build` 명령어는 Dockerfile에 기록된 대로 컨테이너를 실행한 뒤 완성된 이미지를 만들어 낸다.  
그렇지만 이미지로 만드는 과정이 하나의 컨테이너에서 일어나는 것은 아니다.  
이미를 빌드할 때 나오는 다음과 같은 출력결과를 통해 이를 어느정도 짐작할 수 있다.  
```bash
 => [1/6] FROM docker.io/library/ubuntu:14.04                                                                      0.1s
 => [internal] load build context                                                                                  0.1s
 => => transferring context: 41B                                                                                   0.0s
 => [2/6] RUN apt-get update                                                                                      21.6s
 => [3/6] RUN apt-get install apache2 -y                                                                          17.1s
 => [4/6] ADD test.html /var/www/html                                                                              0.0s
 => [5/6] WORKDIR /var/www/html                                                                                    0.0s
 => [6/6] RUN ["/bin/bash", "-c", "echo hello >> test2.html"]                                                      0.3s
```
- 각 Step은 Dockerfile에 기록된 명령어에 해당.  
  (`ADD`, `RUN` 등의 명령어가 실행될 때마다 새로운 컨테이너가 하나씩 생성 -> 새로운 이미지로 커밋 )
	
- 이미지 빌드가 완료되면 Dockerfile의 명령어 줄 수만큼의 레이어가 존재.  
  (중간에 컨테이너도 같은 수만큼 생성되고 삭제됨)

> 캐시를 이용한 이미지 빌드  

```bash

$ vi Dockerfile2
FROM ubuntu:14.04
MAINTAINER kimcs90610
LABEL "purpose"="practice"
RUN apt-get update

--> Dockerfile2 생성  ( RUN apt-get update 까지만 작성 )

$ docker build -f Dockerfile2 -t mycache:0.0 ./
[+] Building 0.1s (6/6) FINISHED                                                                         docker:default
 => [internal] load build definition from Dockerfile2                                                              0.0s
 => => transferring dockerfile: 124B                                                                               0.0s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load metadata for docker.io/library/ubuntu:14.04                                                    0.0s
 => [1/2] FROM docker.io/library/ubuntu:14.04                                                                      0.0s
 => CACHED [2/2] RUN apt-get update                                                                                0.0s
 => exporting to image                                                                                             0.0s
 => => exporting layers                                                                                            0.0s
 => => writing image sha256:5307879b97744401483887a94176d201a2a57ef0d785cf164db0c2fbb5a5b0bf                       0.0s
 => => naming to docker.io/library/mycache:0.0                                                                     0.0s
```

- `-f`: Dockerfile의 이름을 지정할 수 있는 옵션  
- `CACHED`라는 출력 내용 확인  
  (이전에 빌드했던 Dockerfile에 같은 내용이 있다면 build 명령어는 이를 새로 빌드하지 않고 같은 명령어 줄까지 이전에 사용한 이미지 레이어를 활용해 이미지를 생성한다.)

- 캐시 기능이 필요하지 않은 경우가 있다  
  (`git clone` 등의 명령어를 사용할때)
	- docker build `--no-cache` 옵션으로 캐시를 사용하지 않을 수 있다
	- docker build `--cache-from` 옵션으로 직접 캐시로 사용도 가능


#### 2.4.3.3 멀티 스테이지를 이용한 Dockerfile 빌드하기

	하나의 Dockerfile 안에 여러 개의 FROM 이미지를 정의함으로써 빌드 완료 시 최종적으로 생성될 이미지의 크기를 줄이는 역할을 한다.  
---

일반적으로 Dockerfile에서 Go 소스코드를 빌드하기 위한 방법
1. Go와 관련된 도구들이 미리 설치된 이미지를 FROM에 명시
2. RUN 명령어로 소스코드를 컴파일

> Dockerfile을 통해 컴파일될 main.go 파일
```go
package main
import "fmt"
func main() {
    fmt.Println("hello world")
}
```
> golang 이미지를 기반으로 main.go를 컴파일하고 출력프로그램을 실행하는 Dockerfile
```dockerfile
FROM golang
ADD main.go /root
WORKDIR /root
RUN go build -o /root/mainApp /root/main.go
CMD ["./mainApp"]
```

> 빌드
```bash
$ docker build . -t go_helloworld

...

$ docker images
REPOSITORY                TAG       IMAGE ID       CREATED          SIZE
go_helloworld             latest    621ef6d7fd6c   57 seconds ago   852MB
```

- 이미지의 크기가 무려 850MB에 달하는 것 확인.  
  (실제 실행 파일의 크기는 매우 작지만 소스코드 빌드에 사용된 각종 패키지 및 라이브러리가 불필요하게 이미지를 차지하고 있다)

- 17.05버전 이상 도커엔진이라면 이미지의 크기를 줄이기 위해 **멀티 스테이지 빌드 사용가능**

> 멀티 스테이지 빌드 사용  
```dockerfile
FROM golang
ADD main.go /root
WORKDIR /rootRUN go build -o /root/mainApp /root/main.go

FROM alpine:latest
WORKDIR /root
COPY --from=0 /root/mainApp .
CMD ["./mainApp"]
```
- 일반적인 Dockerfile과는 다르게 2개의 FROM 사용
- `COPY --from=0` 을 이용해 첫번째 FROM에서 빌드된 이미지의 최종상태를 카피


```bash
$ docker build . -t go_helloworld:multi-stage

---

$ docker images
REPOSITORY                TAG           IMAGE ID       CREATED          SIZE
go_helloworld             multi-stage   1f64bbc89371   47 minutes ago   9.27MB

```
- 이미지 크기 9.27MB 로 현저하게 줄은 것 확인

이와 같이 멀티 스테이지 빌드는 반드시 필요한 실행 파일만 최종 이미지 결과물에 포함시킴으로써 이미지 크기를 줄일 때 유용하게 사용할 수 있다.

---
>   0, 1.. 의 순으로 구분되어 사용
```bash
FROM golang
ADD main.go /root
...

FROM golang
ADD main2.go /root
...

FROM alpine:latest
WORKDIR /root
COPY --from=0 /root/mainApp
COPY --from=1 /root/mainApp2
```

> 이름을 붙여서 사용가능
```bash
FROM golang as builder
...

...
COPY --from=builder /root/mainApp
...

```
---



### 2.4.4 기타 Dockerfile 명령어
전체 목록을 확인하고 싶다면 도커 공식 사이트의 Dockerfile 레퍼런스를 참고.  
https://docs.docker.com/reference/dockerfile/

#### 2.4.4.1 ENV, VOLUME, ARG, USER
> `ENV`: Dockerfile에서 사용될 환경변수를 지정한다.  
```bash
$ vi Dockerfile
FROM ubuntu:14.04
ENV test /home
WORKDIR $test
RUN touch $test/mytouchfile
```
- test 라는 환경변수에 /home 이라는 값을 설정
- 환경변수는 이미지에도 저장되므로 빌드된 이미지로 컨테이너를 생성하면 환경변수 사용가능  

> `VOLUME`: 빌드된 이미지로 컨테이너를 생성했을 때 호스트와 공유할 컨테이너 내부의 디렉터리를 설정함.  
```bash
$ vi Dockerfile
FROM ubuntu:14.04
RUN mkdir /home/volume
RUN echo test >> /home/volume/testfile
VOLUME /home/volume
```
- 컨테이너 내부의 /home/volume 디렉터리를 호스트와 공유하도록 설정  

> `ARG`: bulid 명령어를 실행할 때 추가로 입력을 받아 Dockerfile 내에서 사용될 변수의 값을 설정  
```bash
$ vi Dockerfile
FROM ubuntu:14.04
ARG my_arg
ARG my_arg_2=value2
RUN touch ${my_arg}/mytouch
```
- build 명령어에서 my_arg와 my_arg_2라는 이름의 변수를 추가로 입력받을 것이라고 ARG를 통해 명시  
- my_arg_2는 기본값 value2로 지정

```bash
$ docker build --build-arg my_arg=/home -t myarg:0.0 ./
```
- `--build-arg`옵션을 사용해 ARG에 값 입력 <키>=<값>
- ENV와 같이 정의 시  ENV에 의해 덮어쓰여진다.

> `USER`: USER로 컨테이너 내에서 사용될 사용자 계정의 이름이나 UID를 설정하면 그 아래의 명령어는 해당 사용자 권한으로 실행된다.  
```bash
...
RUN groupadd -r author && useradd -r -g author kimcs90610
USER kimcs90610
...
```
- 일반적으로 RUN으로 사용자의 그룹과 계정을 생성한 뒤 사용. 
- 루트 권한이 필요하지 않다면 USER를 사용하는 것을 권장.  
---
기본적으로 컨테이너 내부에서는 root 사용자를 사용하도록 설정된다.  
이는 컨테이너가 호스트의 root 권한을 가질 수 있다는 것을 의미하기 때문에 보안 측면에서 매우 바람직하지 않다.  
때문에 컨테이너 애플리케이션을 최종적으로 배포할 때는 컨테이너 내부에서 새로운 사용자를 새롭게 생성해 사용하는 것을 권장.  
docker run 명령어 자체에서도 --user 옵션을 지원하지만, 가능하다면 이미지 자체에 root가 아닌 다른 사용자를 설정해 놓는 것이 좋다.  

---


#### 2.4.4.2 Onbuild, Stopsignal, Healthcheck, Shell
> `ONBUILD`: 빌드된 이미지를 기반으로 하는 다른 이미지가 Dockerfile로 생성될 때 실행할 명령어를 추가.  
```bash
$ vi Dockerfile
FROM ubuntu:14.04
RUN echo "this is onbuild test"!
ONBUILD RUN echo "onbuild!" >> /onbuild_file
```
- 해당 도커파일로 생성된 이미지를 기반으로 Dockerfile을 만들어 build시 해당 RUN ~ 명령어가 빌드시 실행된다.

```bash
FROM maven:3-jdk-8-alpine
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
ONBUILD ADD . /usr/src/app
ONBUILD RUN mvn install
```
- 활용 예시: 메이븐의 도커파일
- 개발자는 해당 도커파일을 프로젝트 폴더에 위치시키고 해당 도커파일로부터 빌드된 이미지를 FROM 항목에 입력하여 메이븐을 쉽게 사용가능  

> `STOPSIGNAL`: 컨테이너가 정지될 때 사용될 시스템 콜의 종류를 지정.  
```bash
FROM ubuntu:14.04
STOPSIGNAL SIGKILL
```
- 아무것도 설정 안할 시 기본값은 `SIGTERM`, 예제는 `SIGKILL`로 설정
- docker run 명령어에서 --stop-signal 옵션과 같다.  
- docker stop, docker kill 에도 적용된다.

![자주쓰는시그널목록](/img/자주쓰는시그널목록.jpg)

SIGKILL, SIGSTOP을 제외한 모든 SIGNAL은 Process에 의해 intercept 될 수 있습니다.  
출처: https://kimjingo.tistory.com/73 [김징어의 Devlog:티스토리]


> `HEALTHCHECK`: 이미지로부터 생성된 컨테이너에서 동작하는 애플리케이션의 상태를 체크하도록 설정.
```bash
FROM nginx
RUN apt-get update -y && apt-get install curl -y
HEALTHCHECK --interval=1m --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1
```
- 1분마다 curl -f ~를 실행, 3초 이상 소요시 한번의 실패로 간주, 3번 이상 타임아웃이 발생하면 해당 컨테이너는 unhealthy상태가 된다.  

> `SHELL`: 사용하려는 셸을 명시한다.  
 
도커파일은 기본적으로 리눅스에서는 `/bin/sh -c`, 윈도우에서는 `cmd /S /C`를 사용하지만 사용하려는 셸을 직접 명시 할때 `SHELL` 명령어를 사용한다.
```bash
FROM node
RUN echo hello, node!
SHELL ["/usr/local/bin/node"]
RUN -v
```
- node를 기본 셸로 사용하도록 설정

#### 2.4.4.3 ADD. COPY



#### 2.4.4.4 ENTRYPOINT, CMD

### 2.4.5 Dockerfile로 빌드할 때 주의할 점