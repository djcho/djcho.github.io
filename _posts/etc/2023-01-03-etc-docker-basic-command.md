---
title : "[Docker] 기본 명령어 정리"
categories:
- Etc
tag :
- [Etc, Docker]
toc: true
toc_sticky : true
published : true
date : 2023-01-03 00:00:00
last_modified_at : 2022-01-03 00:00:00
---

## Docker 명령어 정리

### 컨테이너 명령어

#### 컨테이너 생성하기

```bash
$ docker run -d {image id|name} {CMD} {parameter}

#ex
$ docker run -d pcloudesire/tomcat:7-jre7 -p 8090:8080 /run.sh
```

| 옵션             | 설명                                                         |
| ---------------- | ------------------------------------------------------------ |
| -i --interactive | 컨테이너의 표준 입력(stdin)을 활성화. (주로 -it 함께 사용)   |
| -t --tty         | tty(가상 터미널)을 할당. 리눅스에 키보드를 통해 표준 입력(stdin)을 전달할 수 있게한다. (주로 -it 함께 사용) |
| --name           | 컨테이너 이름을 지정.                                        |
| -d --detach      | 컨테이너를 백그라운드로 실행.                                |
| --rm             | docker run 명령어가 끝나면, 컨테이너 자동 삭제.              |
| -p --publish     | 호스트와 컨테이너의 포트를 연결 (포트포워딩). -p <호스트 포트>:<컨테이너 포트> ex) -p 80:8888 - 호스트에 8888로 접속하면, 컨테이너 내부의 80포트로 자동 접속. |
| -v --volume      | 호스트와 컨테이너의 디렉토리 연결(마운트) -v <호스트 절대경로>:<컨테이너 절대경로> ex) -v /Users:/usr. - 컨테이너 /usr에 저장하는 파일은 호스트의 /Users 디렉토리에 저장. |
| --restart        | 컨테이너 종료시, 재시작 정책 설정 --restart="always" : 항상 재시작 --restart="on-failure" : 종료 스테이터스가 0이 아닐 때 재시작 * --rm 옵션과 --restart 옵션은 동시에 사용할 수 없습니다. |
| --privileged     | 컨테이너 안에서 호스트의 리눅스 커널 기능을 모두 사용        |

#### 컨테이너 실행하기

```bash
$ docker start {container id|name}
$ docker stop {container id|name}
$ docker restart {container id|name}
```

#### 모든 컨테이너 확인하기

```bash
$ docker ps -a
```

#### 컨테이너 내부 bash 쉘 진입

```bash
$ docker exec -it {container id|name} /bin/bash

#ex
$ docker exec -it web1 /bin/bash
```

#### 컨테이너 정보 확인

```bash
$ docker inspect {container id|name}

#ex
$ docker inspect web1
```

#### 컨테이너에 연결하기

```bash
$ docker attach {container id|name}

#ex
$ docker attach web1
```

#### 컨테이너 로그 출력

```bash
$ docker logs -f {container id|name}

#ex
$ docker logs -f web1
```



### 이미지 수정하기

#### 이미지 목록 확인하기

```bash
$ docker image ls
```

#### 이미지 수정하기

현재 구동중인 컨테이너 쉘에 접속하여 원하는 파일을 변경한 뒤 변경한 컨테이너를 대상으로 이미지 파일에 커밋한다.

```bash
$ docker commit --change='CMD {컨테이너 시작 시 적용할 커맨드}' {modified container id|name} {image name}

#ex
$ docker commit --change='CMD ["/run.sh"]' web1 cloudesire/tomcat-mariadb-webapp
```



### 네트워크 명령어

#### 네트워크 주소 확인하기

```bash
sudo ip addr show docker0 #도커 네트워크에 할당된 IP확인
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:e6:19:ee:27 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e6ff:fe19:ee27/64 scope link
       valid_lft forever preferred_lft forever
```

#### 네트워크 목록 확인하기

```bash
$ docker network inspect ls
```

#### 네트워크 생성하기

```bash
$ docker network create {network name}

#ex
docker network create myNetwork
```

#### 네트워크 정보 확인하기

```bash
$ docker network inspect {network name}

#ex
$ docker network inspect myNetwork
```

#### 컨테이너끼리 커스텀 네트워크로 연결

```bash
$ docker network connet {network name} {container id|name}

#ex
$ docker network connect myNetwork web1
$ docker network connect myNetwork db1
```

### 컨테이너간 통신

#### bridge 이용

bridge 모드로 컨테이너에 부여된 IP를 확인하고, 같은 네트워크안에 묶인 IP와 통신 가능

기본적으로 docker 는 디폴트 네트워크에 모든 컨테이너를 연결 함

##### 같은 네트워크에 연결 되었는지 확인

```bash
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
8f51e48c7983        bridge              bridge              local
21cf1de9fbbd        docker_default      bridge              local
1c613b5b4259        host                host                local

$ docker network inspect docker_default
[
    {
        "Name": "docker_default",
        "Id": "21cf1de9fbbd3f9dee2b432803985b5280df95a04cb49019fc071a517c785086",
        "Created": "2020-10-21T05:08:01.128636231Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0362edb6b43156a61190049caf6a8dac01b71a8721774966a563d5f758e2566e": {
                "Name": "webapp1",
                "EndpointID": "d5e460ee03bcf3d9be15d5a5db7976334aabccbf5baa2b7877b63781755d91c7",
                "MacAddress": "02:42:ac:12:00:08",
                "IPv4Address": "172.18.0.8/16",
                "IPv6Address": ""
            },
            "9b55100f4d3c01c1e0d5d68bacb3f9679bf1718ee54a90dbae078b78e71739ba": {
                "Name": "db1",
                "EndpointID": "6c129dc79e2131b01e7c43e5f08c4e34d08b048a58fc98c2776de33ff4a2c9de",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "default",
            "com.docker.compose.project": "docker"
        }
    }
]
```



#### 네트워크 및 IP지정하여 컨테이너 생성

```bash
$ docker run -it --rm --network docker_default --ip 172.18.0.9 -p 8281:8080 -d cloudesire/tomcat:latest /run.sh
```



#### host 이용

network-mode 를 host 로 지정할 경우 호스트의 설정들을 모두 따르므로 port forwarding 은 동작하지 않음

```bash
$ docker run --net=host {image id|name} {container id|name}
```

### docker-compose 구성

```yml
version : '3'
services :
        {컨테이너 이름} :
#ex     test_maria_db :
                image : {이미지 이름}
#ex             image : mariadb:latest
                network-mode : host
                environment :
                        MYSQL_ROOT_PASSWORD : {ROOT 비밀번호}
                        MYSQL_DATABASE : {DB 이름}
                        MYSQL_USER : {DB 접속 유저 이름}
                        MYSQL_PASSWORD : {DB 접속 유저 비밀번호}
                volumes :
                      - {src path}:{dest path}                
#ex                   - /home/djcho/docker/shared_volumes/mariadb_10_0/dbdata:/var/lib/mysql
                networks:
                        default: #network 지정
                                ipv4_address: 172.18.0.5 #ip지정
        test_server_webapp :
                image : cloudesire/tomcat:latest
                ports :
                      - "{src port}:{dest port}"                
#ex                   - "8280:8080"
#ex                   - "8643:8443"
                volumes :
                      - /home/djcho/docker/shared_volumes/tomcat/webapps:/tomcat/webapps
                      - /home/djcho/docker/shared_volumes/tomcat/my_files:/tomcat/my_files
                environment:
                      JAVA_OPTS : "-Dspring.profiles.active=dev-jndi  -Doracle.jdbc.timezoneAsRegion=false"
                networks:
                        default:
                                ipv4_address: 172.18.0.5
```



#### 컨테이너간 통신 여부 확인하기

```bash
$ docker-compose exec {service1 name} ping {service1 name}

#ex
$ docker-compose exec test_server_webapp ping test_maria_db
PING test_server_webapp (172.18.0.8) 56(84) bytes of data.
64 bytes from docker_test_server_webapp_1.docker_default (172.18.0.8): icmp_seq=1 ttl=64 time=0.092 ms
64 bytes from docker_test_server_webapp_1.docker_default (172.18.0.8): icmp_seq=2 ttl=64 time=0.077 ms
64 bytes from docker_test_server_webapp_1.docker_default (172.18.0.8): icmp_seq=3 ttl=64 time=0.075 ms
64 bytes from docker_test_server_webapp_1.docker_default (172.18.0.8): icmp_seq=4 ttl=64 time=0.073 ms
64 bytes from docker_test_server_webapp_1.docker_default (172.18.0.8): icmp_seq=5 ttl=64 time=0.079 m
```

