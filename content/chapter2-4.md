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
- 


```bash
$ docker build . -t go_helloworld:multi-stage

---

$ docker images
REPOSITORY                TAG           IMAGE ID       CREATED          SIZE
go_helloworld             multi-stage   1f64bbc89371   47 minutes ago   9.27MB

```


### 2.4.4 기타 Dockerfile 명령어

#### 2.4.4.1 ENV, VOLUME, ARG, USER

#### 2.4.4.2 Onbuild, Stopsignal, Healthcheck, Shell

#### 2.4.4.3 ADD. COPY

#### 2.4.4.4 ENTRYPOINT, CMD

### 2.4.5 Dockerfile로 빌드할 때 주의할 점