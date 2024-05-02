# 도커 컴포즈
## 4.1 도커 컴포즈를 사용하는 이유
여러개의 컨테이너가 하나의 애플리케이션으로 동작할 때 이틀 테스트하려면 각 컨테이너를 하나씩 생성해야한다.  
이때, 매번 run 명령어에 옵션을 설정해 CLI로 컨테이너를 생성하기보다는 여러 개의 컨테이너를 하나의 서비스로 정의해 컨테이너 묶음으로 관리할 수 있다면 좀 더 편리할 것.  
이를 위해 도커 컴포즈는 컨테이너를 이용한 서비스의 개발과 CI를 위해 여러 개의 컨테이너를 하나의 프로젝트로서 다룰 수 있는 작업 환경을 제공한다.  
도커 컴포즈는 여러 개의 컨테이너의 옵션과 환경을 정의한 파일을 읽어 컨테이너를 순차적으로 생성하는 방식을 동작한다.  


## 4.2 도커 컴포즈 설치
-- 생략 --  

데스크탑 버전 도커 설치시 자동으로 설치됨  
```bash
$ docker -v
Docker version 24.0.7, build afdd53b

$ docker-compose -v
Docker Compose version v2.23.3-desktop.2
```

## 4.3 도커 컴포즈 사용

### 4.3.1 도커 컴포즈 기본 사용법

도커 컴포즈는 컨테이너의 설정이 정의된 YAML 파일을 읽어 도커 엔진을 통해 컨테이너를 생성하므로 도커 컴포즈를 사용하려면 YAML파일을 작성해야한다.

#### 4.3.1.1 docker-compose.yml 작성과 활용

> run 명령어를 docker-compose.yml 파일로 변환하기

`run 명렁어`
```bash
$ docker run -d --name mysql \
alicek106/composetest:mysql \
mysqld

$ docker run -d -p 80:80 \
--link mysql:db --name web \
alicek106/composetest:web \
apachectl -DFOREGROUND
```
`docker-compose.yml`
```yml
version: '3.0'
services:
  web:
    image: alicek106/composetest:web
    ports:
      - "80:80"
    links:
      - mysql:db
    command: apachectl -DFOREGROUND
  mysql:
    image: alicek106/composetest:mysql
    command: mysqld
```
- `version`: YAML 파일 포맷의 버전을 나타냄. 버전은 1, 2, 2.1, 3.0 등이 있지만, 도커 컴포즈 버전은 도커 엔진 버전에 의존성이 있으므로 가능하다면 최신 버전을 사용하는 것이 좋다.  
[호환성목록](https://github.com/docker/compose/releases)

- `services`: 생성될 컨테이너들을 묶어놓은 단위. 서비스 항목 아래에는 각 컨테이너에 적용될 생성 옵션을 지정한다.
- `web, mysql`: 생성될 서비스의 이름.  
  이 항목 아래에 컨테이너가 생성될 때 필요한 옵션을 지정할 수 있다.  


> 실행결과  
> 
`docker-compose.yml`  
- 어떠한 설정도 하지 않으면 현재 디렉토리의 docker-compose.yml 파일을 읽는다.
```bash
$ docker-compose up -d
[+] Running 10/10
 ✔ mysql 5 layers [⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                           34.6s
   ✔ d89e1bee20d9 Pull complete                                                                              22.6s
   ✔ 9e0bc8a71bde Pull complete                                                                              0.8s
   ✔ 27aa681c95e5 Pull complete                                                                              1.1s
   ✔ a3ed95caeb02 Pull complete                                                                              1.8s
   ✔ 7ab04d11bb96 Pull complete                                                                              23.8s
 ✔ web 3 layers [⣿⣿⣿]      0B/0B      Pulled                                                                34.3s
   ✔ 15bc302aa28e Pull complete                                                                             19.5s
   ✔ 7233974738a3 Pull complete                                                                             21.4s
   ✔ 732ac06e8a0b Pull complete                                                                             22.7s
[+] Running 3/3
 ✔ Network dockercompose_default    Created                                                                  0.1s
 ✔ Container dockercompose-mysql-1  Started                                                                  1.3s
 ✔ Container dockercompose-web-1    Started                                                                  0.1s
```

![컴포즈실행결과](/img/도커컴포즈테스트.png)  

![확인1](/img/컴포즈컨테이너확인1.png)  
![확인2](/img/컴포즈컨테이너확인2.png)
- 프로젝트의 이름을 명시하지 않아 디렉토리의 이름으로 시작하는 이름을 갖게 되었다.  
  (디렉토리: C:/dockerCompose)

#### 4.3.1.2 도커 컴포즈의 프로젝트, 서비스, 컨테이너
도커 컴포즈는 컨테이너를 프로젝트 및 서비스 단위로 구분하므로 컨테이너의 이름은 일반적으로 다음과 같은 형식으로 정해진다.

	[프로젝트 이름]_[서비스 이름]_[서비스 내에서 컨테이너의 번호]  

위 예제에서 생성한 프로젝트의 이름은 `dockercompose` 이고, 각 서비스의  이름은 `mysql`, `web` 이다.  

> `docker-compose scale` 명령어로 컨테이너 2, 3 ... 을 생성할 수 있다.  
```bash
$ docker-compose scale mysql=2
[+] Running 2/2
 ✔ Container dockercompose-mysql-1  Running                                                                         0.0s
 ✔ Container dockercompose-mysql-2  Started                                                                         0.1s
```

> `docker-compose down` 명령어로 프로젝트를 삭제할 수 있다.
```bash
$ docker-compose down
[+] Running 4/4
 ✔ Container dockercompose-web-1    Removed                                                                         10.5s
 ✔ Container dockercompose-mysql-2  Removed                                                                         2.2s
 ✔ Container dockercompose-mysql-1  Removed                                                                         1.6s
 ✔ Network dockercompose_default    Removed                                                                         0.2s
```

도커 컴포즈는 기본적으로 현재 디렉토리의 이름으로 된 프로젝트를 제어한다.  
하지만 `-p` 옵션으로 프로젝트의 이름을 직접 명시하여 제어할 수 있다.

> `-p`옵션 사용
```bash
$ docker-compose -p myproject up -d
[+] Running 3/3
 ✔ Network myproject_default    Created                                                                             0.1s
 ✔ Container myproject-mysql-1  Started                                                                             0.1s
 ✔ Container myproject-web-1    Started                                                                             0.1s
```
![p옵션](/img/컴포즈p옵션사용.png)
```bash
$ docker-compose -p myproject down
[+] Running 3/3
 ✔ Container myproject-web-1    Removed                                                                        10.3s
 ✔ Container myproject-mysql-1  Removed                                                                        2.1s
 ✔ Network myproject_default    Removed                                                                        0.2s
```

---
> `docker-compose up` 명령어의 끝에 서비스의 이름을 입력해  yml파일에 명시된 특정 서비스의 컨테이너만 생성할 수 있다.  

	$ docker-compose up -d mysql

> `docker-compose run` 명령어로 컨테이너를 생성할 수도 있다. 이때는 interactive 셸을 사용할 수 있다.  

	$ docker-compose run web /bin/bash

---

### 4.3.2 도커 컴포즈 활용
#### 4.3.2.1 YAML 파일 작성
도커 컴포즈를 사용하려면 컨테이너 설정을 저장해 놓은 YAML 파일이 필요. 그러므로 기존에 사용하던 run 명령어를 YAML 파일로 변환하는 것이 도커 컴포즈 사용범의 대부분이다.  
YAML 파일은 크게 `버전`, `서비스`, `볼륨`, `네트워크` 정의로 구성된다.  
각 항목의 하위 항목을 정의하려면 `2개의 공백(스페이스)`로 들여쓰기해야 한다.  

---
도커 컴포즈는 기본적으로 현재 디렉토리 또는 상위 디렉토리에서 docker-compose.yml 파일을 찾아서 컨테이너를 생성한다.  그러나 docker-compose 명령어의 `-f` 옵션을 사용하면 yml파일의 위치와 이름을 지정할 수 있다.

```bash
$ docker-compose \
-f c:/dockerCompose/my_compose_file.yml \
up -d
```
---

(1) 버전정의  
YAML 파일 포맷에는 버전 1, 2, 2.1, 3 이 있지만 가능하면 최신버전을 사용하는게 좋다.  
(버전 3부터 도커 스웜모드와 호환된다.)  

버전 항목은 일반적으로  YAML파일의 맨 윗부분에 명시한다.

	version: '3.0'

(2) 서비스 정의  
서비스는 도커 컴포즈로 생성할 컨테이너 옵션을 정의한다.  
각 서비스는 컨테이너로 구현되며, 하나의 프로젝트로서 도커 컴포즈에 의해 관리된다.  
서비스의 이름은 `services`의 하위 항목으로 정의하고, 컨테이너의 옵션은 서비스 이름의 하위 항목에 정의한다.  
```yaml
services:
  my_container_1:
    image: ...
  my_container_2:
    image: ...
```
[공식문서: https://docs.docker.com/compose/compose-file](https://docs.docker.com/compose/compose-file)  

> 서비스항목이 가질 수 있는 주요 컨테이너 옵션  

- `image`: 서비스의 컨테이너를 생성할 때 쓰일 이미지의 이름
```yaml
services:
  my_container_1:
    image: ubuntu:14.04
```
- `link`: docker run의 -link와 같다. 다른 서비스에 서비스 명만으로 접근하도록 설정한다. 별칭을 줄 수도 있다. (SERVICE:ALIAS)
```yaml
services:
  web:
    links:
      - db
			- db:database
			- redis
```
- `environment`: docker run의 --env, -e 옵션과 동일. 컨테이너 내부에서 사용할 환경변수 설정 (딕셔너리나 배열 형태 가능)
```yaml
services:
  web:
    environment:
      - MYSQL_ROOT_PASSWORD=mypassword
      - MYSQL_DATABASE_NAME=mydb  

#  = 를 : 로 가능
```

- `command`: docker run의 마지막에 붙는 커맨드와 같다. 컨테이너가 실행될 때 수행할 명령어를 지정한다.  
```yaml
services:
  web:
    image: alicek106/composetest:web
    command: apachectl -DFOREGROUND # [apachectl, -DFOREGROUND]
```

- `depends_on`: 특정 컨테이너에 대한 읜존 관계를 나타내며, 이 항목에 명시된 컨테이너가 먼저 생성되고 실행됩니다.  
```yaml
services:
  web:
    image: alicek106/composetest:web
    depends_on:
      - mysql
  mysql:
    image: alicek106/composetest:mysql
```
특정 서비스의 컨테이너만 생성하되 의존성이 없는 컨테이너를 생성하려면 `--no-deps` 옵션을 사용한다.  

	$ docker-compose up --nodeps web
---
links, depends_on 모두 실행 순서만 설정할 뿐 컨테이너 내부의 애플리케이션이 준비된 상태인지에 대해서는 확인하지 않는다.  
이를 해결하는 방법으로 컨테이너에 셸 스크립트를 entrypoint로 지정하는 방법이 있다.
```yaml
services:
  web:
    # ...
    entrypoint: ./sync_script.sh mysql:3306
```

```sh
# sync_script.sh

until (상태를 확인할 수 있는 명령어); do # 상태를 확인할수있는 명령어 예) curl mysql:3306
  echo "depend container is not available yet"
  sleep 1
done
echo "depends_on container is ready"
```

---

- `ports`: docker run 명령어의 -p와 같으며 서비스의 컨테이너를 개방할 포트를 설정한다. 그러나 단일 호스트 환경에서 80:80과 같이 호스트의 특정 포트를 서비스의 컨테이너에 연결하면 docker-compose scale 명령어로 서비스의 컨테이너 수를 늘릴 수 없다.  

```yaml
services:
  web:
    image: alicek106/composetest:web
      ports:
        - "8080"
        - "8081-8085"
        - "80:80"
```

- `build`: build 항목에 정의된 Dockerfile에서 이미지를 빌드해 서비스의 컨테이너를 생성하도록 설정한다.  
```yaml
services:
  web:
    build: ./composetest
    context: ./composetest
    dockerfile: myDockerfile
    args:
      HOST_NAME: web
      HOST_CONFIG: self_config

# image 항목을 설정하지 않으면 이미지의 이름은 [프로젝트 이름]:[서비스 이름]
```
---
build 항목을 YAML 파일에 정의해 프로젝트를 생성하고 난 뒤 Dockerfile을 변경하고 다시 프로젝트를 생성해도 이미지를 새로 빌드하지 않는다.  
docker-compose up -d에 --build 옵션을 추하하거나 docker-compose build 명령어를 사용해 Dockerfile이 변경돼도 컨테이너를 생성할 때마다 빌드하도록 설정할 수 있다.
```
$ docker-compose up -d --build
$ docker-compose build [yml 파일에서 빌드할 서비스 이름]
```

---

- `extends`: 다른 YAML 파일이나 현재 YAML 파일에서 서비스 속성을 상속받게 설정한다.  

파일에서 상속 받는 경우
```yaml
# 상속받을 docker-compose.yml 파일
version: '3.0'
  services:
    extends:
      file: extends_compose.yml
      service: extend_web

# 상속할 파일명 extend_compose.yml
version: '3.0'
  services:
    extend_web:
    image: ubuntu:14.04
    ports:
      - "80:80"
```
현재 파일의 서비스 속성을 상속받는 경우
```yaml
version: '3.0'
  services:
    web:
      extends:
        service: extend-web
    extend_web:
      image: ubuntu:14.04
      ports:
        - "80:80"
```

`depencds_on`, `links`, `volumes_from` 항목은 각 컨테이너 사이의 의존성을 내포하고 있으므로 extends로 상속받을 수 없다.


(3) 네트워크 정의  

- `driver`: 도커 컴포즈는 생성된 컨테이너를 위해 기본적으로 브리지 타입의 네트워크를 생성한다. 그러나 YAML 파일에서 `driver` 항목을 정의해 다른 네트워크를 사용하도록 설정할 수 있다.  
드라이버에 필요한 옵션은 `driver_opts`로 전달할 수 있다.

```yaml
version: '3.0'
services:
  myservice:
    image: nginx
    networks:
      - mynetwork     # 선언한 네트워크 사용
networks:
  mynetwork:          # 네트워크 선언
    driver: overlay   # 스웜 모드나 주키퍼(분산처리 시스템)를 사용하는 환경에서만 생성가능
    driver_opts:
      subnet: "255.255.255.0"
      IPAdress: "10.0.0.2"
```

- `ipam`: IPAM(IP Address Manager)를 위해 사용할 수 있는 옵션으로서 subnet, ip 범위 등을 설정할 수 있다. driver 항목에는 IPAM을 지원하는 드라이버의 이름을 입력한다.  
```yaml
services:
...

networks:
  ipam:
    driver: mydriver
    config:
      subnet: 172.20.0.0/16
      ip_range: 172.20.5.0/24
      gateway: 172.20.5.1
```

- `external`: YAML 파일을 통해 프로젝트를 생서할 때마다 네트워크를 생성하는 것이 아닌, 기존의 네트워크를 사용하도록 설정합니다. 이를 설정하려면 사용하려는 외부 네트워크의 이름을 하위 항목으로 입력한 뒤 external의 값을 true로 설정한다.  
external 옵션은 준비된 네트워크를 사용하므로 driver, driver_opts, ipam을 사용할 수 없다.

```yaml
services:
  web:
    image: alicek106/composetest:web
    networks:
      - alicek106_network
networks:
  alicek106_network:  # 기존 네트워크
    external: true
```
(4) 볼륨 정의  

- `driver`: 볼륨을 생성할 때 사용될 드라이버를 설정한다. 어떠한 설정도 하지 않으면 local로 설정된다. 추가 옵션은 `driver_opts`를 통해 인자로 설정 가능
```yaml
version: '3.0'
...

volumes:
  driver: flocker
    driver_opts:
      opt: "1"
      opt2 : 2
```

- `external`: 도커 컴포즈는 YAML파일에서 volume, volumes-from 옵션 등을 사용하면 프로젝트마다 볼륨을 생성한다. 이때 external 옵션을 설정하면 볼륨을 프로젝트를 생성할 때마다 매번 생성하지 않고 기존 볼륨을 사용하도록 설정한다.

```yaml
services:
  web:
    image: alicek106/composetest:web
    volumes:
      - myvolume:/var/www/html   # myvolume 이란 외부 볼륨을 web서비스의 컨테이너에 마운트
volumes:
  myvolume:
    external: true
```
(5) YAML 파일 검증하기  
YAML 파일을 작성할 때 오타 검사나 파일 포맷이 적절한지 등을 검사하려면 `docker-compose config` 명령어를 사용한다. 기본적으로 현재 디렉토리의 docker-compose.yml 파일을 검사하지만 `-f`옵션으로 검사할 파일의 경로를 설정할 수 있다.

```bash
# 정상적일때
$ docker-compose config
name: dockercompose
services:
  mysql:
    command:
      - mysqld
    image: alicek106/composetest:mysql
    networks:
      default: null
  web:
    command:
      - apachectl
      - -DFOREGROUND
    depends_on:
      mysql:
        condition: service_started
        restart: true
        required: true
    image: alicek106/composetest:web
    links:
      - mysql:db
    networks:
      default: null
    ports:
      - mode: ingress
        target: 80
        published: "80"
        protocol: tcp
networks:
  default:
    name: dockercompose_default

# 오타냈을때  networks -> netwodrks
$ docker-compose config
validating C:\dockerCompose\docker-compose.yml: (root) Additional property netwodrks is not allowed

```


#### 4.3.2.2 도커 컴포즈 네트워크
YAML 파일에 네트워크 항목을 정의하지 않으면 도커 컴포즈는 프로젝트별로 브리지 타입의 네트워크를 생성한다.  생성된 네트워크의 이름은 `{프로젝트 이름}_default`로 설정되며, docker-compose up 명령어로 생성되고 docker-compose down 명령어로 삭제된다.  
```bash
# Network dockercompose_default Created
$ docker-compose up -d
[+] Running 2/3
 - Network dockercompose_default    Created                                                      1.1s
 ✔ Container dockercompose-mysql-1  Started                                                     0.6s
 ✔ Container dockercompose-web-1    Started                                                     1.0s

# Network dockercompose_default Removed
$ docker-compose down
[+] Running 3/3
 ✔ Container dockercompose-web-1    Removed                                                      10.4s
 ✔ Container dockercompose-mysql-1  Removed                                                      2.2s
 ✔ Network dockercompose_default    Removed                                                      0.3s

```

`docker-compose scale` 명령어로 생성되는 컨테이너 전부가 이 브리지 타입의 네트워크를 사용한다.  
서비스 내의 컨테이너는 --net-alias가 서비스의 이름을 갖도록 자동으로 설정되므로 이 네트워크에 속한 컨테이너는 서비스의 이름으로 서비스 내의 컨테이너에 접근할 수 있다.  

예) scale로 컨테이너 수를 늘려도 (mysql-1, mysql-2 ... )  web서비스의 컨테이너가 mysql이라는 호스트 이름으로 접근하면 mysql 서비스의 컨테이너 중 하나의 IP로 변환(resolve)되며, 컨테이너가 여러 개 존재할 경우 라운드 로빈으로 연결을 분산한다.


---
`run` 명령어의 `--net-alias`와 브리지 타입의 네트워크가 작동하는 방식에 대한 자세한 설명은 2.2.7.2절 "도커 네트워크 기능"의 --net-alias 사용법 참고  

---


#### 4.3.2.3 도커 스웜 모드와 함께 사용하기
스웜모드와 함께 사용되는 개념인 스택(stack)  
스택은 YAML 파일에서 생성된 컨테이너의 묶음  
YAML파일로 스택을 생성하면 YAML 파일에 정의된 서비스가 스웜 모드의 클러스터에서 일괄적으로 생성된다.  

`$ docker stack deploy -c docker-compose.yml mystack`
- mystack이라는 이름의 스택을 생성

## 4.4 도커 학습을 마치며: 도커와 컨테이너 생태계
컨테이너 생태계에는 runC와 containerd 외에도 다양한 컨테이너들이 존재.  
나중에 필요한 순간이 왔을때 `runC`, `containerd` 등의 키워드를 생각해내고 스스로 익힐 수 있으면 그걸로 충분