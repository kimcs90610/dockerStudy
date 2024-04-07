## 도커 이미지

자바의 메이븐, Node의 NPM, 그리고 github처럼  
도커는 **도커 허브** 라는 중앙 이미지 저장소가 존재한다.

- `docker create`, `docker run`, `docker pull` 의 명령어로 이미지를 내려 받을 때, 도커 허브에서 해당 이미지를 검색한 뒤 내려받는다.
- 도커 허브는 누구나 이미지를 올릴 수 있기 때문에, 공식(Official) 라벨이 없는 이미지는 주의해서 사용해야한다.
- private 저장소를 이용하려면 저장소 수에 따라 요금을 지불해야한다. (깃헙처럼)
- 직접 레파지토리를 구축할 수도 있다.
  - ex) Nexus, 비트버킷, gitlab 등등.. 처럼


> `docker search ubuntu`
```bash
$ docker search ubuntu
NAME                             DESCRIPTION                                      STARS     OFFICIAL
ubuntu                           Ubuntu is a Debian-based Linux operating sys…   16984     [OK]
neurodebian                      NeuroDebian provides neuroscience research s…   107       [OK]
ubuntu-debootstrap               DEPRECATED; use "ubuntu" instead                 52        [OK]
open-liberty                     Open Liberty multi-architecture images based…   64        [OK]
websphere-liberty                WebSphere Liberty multi-architecture images …   298       [OK]
ubuntu-upstart                   DEPRECATED, as is Upstart (find other proces…   115       [OK]
ubuntu/nginx                     Nginx, a high-performance reverse proxy & we…   112
ubuntu/squid                     Squid is a caching proxy for the Web. Long-t…   88
ubuntu/cortex                    Cortex provides storage for Prometheus. Long…   4
ubuntu/prometheus                Prometheus is a systems and service monitori…   60
ubuntu/apache2                   Apache, a secure & extensible open-source HT…   72
ubuntu/kafka                     Apache Kafka, a distributed event streaming …   45
ubuntu/bind9                     BIND 9 is a very flexible, full-featured DNS…   83
ubuntu/zookeeper                 ZooKeeper maintains configuration informatio…   13
ubuntu/mysql                     MySQL open source fast, stable, multi-thread…   61
ubuntu/jre                       Distroless Java runtime based on Ubuntu. Lon…   13
ubuntu/postgres                  PostgreSQL is an open source object-relation…   36
ubuntu/redis                     Redis, an open source key-value store. Long-…   22
ubuntu/dotnet-aspnet             Chiselled Ubuntu runtime image for ASP.NET a…   17
ubuntu/grafana                   Grafana, a feature rich metrics dashboard & …   9
ubuntu/dotnet-deps               Chiselled Ubuntu for self-contained .NET & A…   15
ubuntu/memcached                 Memcached, in-memory keyvalue store for smal…   5
ubuntu/dotnet-runtime            Chiselled Ubuntu runtime image for .NET apps…   16
ubuntu/prometheus-alertmanager   Alertmanager handles client alerts from Prom…   9
ubuntu/cassandra                 Cassandra, an open source NoSQL distributed …   2

```
도커 허브 사이트에서 검색 할수도 있지만 명령어로도 가능  

---



### 도커 이미지 생성

직접 만들어보기  

```bash
이미지로 만들 컨테이너 생성

$ docker run -it --name commit_test ubuntu:14.04
root@db8db10faa0e:/# echo test_first! >> first
```  
- first라는 이름의 파일을 생성

--- 


```bash 
$ docker commit \
> -a "chansoo" \
> -m "my first commit" \
> commit_test \
> commit_test:first
sha256:8e0638b6c46b6e610790ae0b5ff5dbb53bb46d71edc37d95e8dd4b49638f69f3
```
- `docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`
- 이미지의 태그를 입력하지 않으면 자동으로 latest로 설정된다. 
- `-a` : author를 뜻한다. (메타데이터)
- `-m` : 커밋메시지
---

```bash
$ docker images
REPOSITORY                 TAG       IMAGE ID       CREATED         SIZE
commit_test                first     8e0638b6c46b   4 minutes ago   197MB
wordpress                  latest    d8bcd4d2a02b   3 weeks ago     737MB
ubuntu                     latest    ca2b0f26964c   5 weeks ago     77.9MB
mysql                      5.7       5107333e08a8   3 months ago    501MB
docker/welcome-to-docker   latest    c1f619b6477e   5 months ago    18.6MB
centos                     7         eeb6ee3f44bd   2 years ago     204MB
ubuntu                     14.04     13b66b487594   3 years ago     197MB
alicek106/volume_test      latest    342a67bc9486   7 years ago     197MB
```
- commit_test라는 이름에 first라는 태그로 이미지 생성 확인

### 이미지 구조 이해
> 이미지레이어의 구조

`docker inspect`  
![우분투](/img/ubuntu.png)  
`ubuntu:14.04`  
![first](/img/commit-first.png)  
`commit_test:first`  
![second](/img/commit-second.png)  
`commit_test:second`

- `docker images`에서 3개의 이미지가 188MB라고 출력되더라도 실제 변경된 사항만 새로운 레이어에 추가되어 저장된다.  
(188MB + first 파일의크기 + second 파일의 크기)

---

```bash
이미지의 구조는 history 명령을 통해 좀 더 쉽게 확인할 수 있다.

$ docker history commit_test:second
IMAGE          CREATED             CREATED BY                                       SIZE      COMMENT
01535899b7fa   About an hour ago   /bin/bash                                        13B       my second commit
8e0638b6c46b   2 hours ago         /bin/bash                                        12B       my first commit
```

---

> 이미지 삭제

```bash
$ docker rmi commit_test:first
Error response from daemon: conflict: unable to remove repository reference "commit_test:first" (must force) - container a793851e69e1 is using its referenced image 8e0638b6c46b

이미지를 사용중인 컨테이너가 존재하므로 해당 이미지를 삭제할 수 없다는 내용.
```
- `docker rmi -f [컨테이너이름]` 으로 강제 삭제할 수 있지만 이미지 레이어파일을 삭제하지 않고 이미지 이름만 삭제하기때문에 의미가 없다.

- 따라서 컨테이너를 삭제한 뒤 이미지를 삭제해야 한다.  

		$ docker stop commit_test2 && docker rm commit_test2  
		$ docker rmi commit_test:first`
		Untagged: commit_test:first

- `Untagged: ...` 는 이미지에 부여된 이름만 삭제한다는 뜻이다. 고로 아직 commit_test:first 이미지는 삭제되지 않은 상태이다.  
( commit_test:first 를 기반으로 하는 second가 존재하기 떄문 )

```bash
$ docker rmi commit_test:second
Untagged: commit_test:second
Deleted: sha256:01535899b7fa7c89efa18038dc06f38f350e2714a2c21dc0e63c19a05b5a7587
Deleted: sha256:d6c2ed883b0e9cc702c1966ed6382a334cf7a0c327af00a8d0a28a02eb70a2d8
Deleted: sha256:8e0638b6c46b6e610790ae0b5ff5dbb53bb46d71edc37d95e8dd4b49638f69f3
Deleted: sha256:ea0645d05ff2a336d1b9ef4c78d201a0d78f42552f0cb28cc60b90d6695d38de
```
- `Deleted:...` 출력결과는 이미지 레이어가 실제로 삭제됐음을 뜻함.
- 삭제되는 이미지의 부모이미지(first)가 존재하지 않아야만 해당 이미지의 파일이 실제로 삭제된다.  
(ubuntu이미지는 아직 존재하기때문에 삭제되지 않는다.)

---

```plaintext
docker rmi -f 로 강제 삭제하게되면 이미지의 이름이 <none>으로 변경되며 이러한 이미지를 댕글링(dangling) 이미지라고 한다.  
댕글링 이미지는 docker images -f dangling=true 명령어로 확인가능하며, docker image prune 으로 한꺼번에 삭제할 수 있다.
```
---



### 이미지 추출

### 이미지 배포

#### 도커 허브 저장소

#### 도커 사설 레지스트리