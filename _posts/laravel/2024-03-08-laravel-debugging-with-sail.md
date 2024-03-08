---
title : "[Laravel] Sail을 활용한 라라벨 개발 환경 구축 : Debugging with VSCode"
categories:
- Laravel
tag :
- [Laravel, PHP, Debugging]
toc: true
toc_sticky : true
published : true
date : 2024-03-08
last_modified_at : 2024-02-07
---




# Sail을 활용한 라라벨 개발 환경 구축 : Debugging with VSCode

## 개요

이 문서는 로컬 도커 컨테이너로 호스팅 되고 있는 Laravel 및 Lumen 앱을 로컬 IDE에서 디버깅할 수 있도록 환경 구성하는 방법에 대해 설명한다. sail 을 통해 도커 개발 환경을 구축하는 방법에 대해서는 이 문서에서 설명하지 않으며, 도커 환경이 아닌 일반적인 라라벨 프로젝트의 디버깅 환경 구성의 경우 [이 문서]를 참고하길 바란다.



## 디버깅 환경 구성도

개략적인 디버깅 환경의 구성도는 아래와 같다. 브라우저 및 httpClient(postman 등)에서 80번 포트로 리스닝하고 있는 nginx 웹서버에 접속하면, php-fpm 은 Xdebug 라는 PHP디버깅 툴의 설정을 로드하고 이 디버깅 툴은 IDE와 디버그 데이터를 송수신 한다. 여기서 주고 받는 디버그 데이터는 현재 사용자에 의해 IDE에 설정된 중단점 정보 및 사용자의 디버깅 행위(스텝오버, 스텝인 등)와 이에 해당하는 변수나 함수 콜 정보이며 IDE에서는 이러한 데이터들을 바탕으로 사용자에게 디버깅 정보를 뷰로 제공한다.

![image](https://github.com/djcho/draw-io-repo/assets/13410737/2bc42960-5fdc-4c47-9286-fae8205f0fbf)

## 구성 방법

구성 방법은 크게 도커 관련 설정, 컨테이너 내부 php 설정 확인, IDE 설정 순으로 이루어 진다. 

### .env 파일 수정

```ini
APP_NAME=s1esp
APP_ENV=testing
APP_DEBUG=true              <---추가
APP_TIMEZONE=Asia/Seoul
APP_URL=http://localhost
APP_PORT=80
```



### 로컬 도커 설정

#### 1. xdebug.ini파일 추가

로컬 프로젝트 디렉토리에 `docker/8.0/xdebug.ini` 파일을 생성한 뒤 아래 내용을 추가한다.

```ini
[XDebug]
zend_extension = xdebug.so
xdebug.mode = develop,debug
xdebug.start_with_request = yes
xdebug.discover_client_host = true
xdebug.idekey = VSCODE
xdebug.client_host = host.docker.internal
xdebug.client_port = 9003
xdebug.log=/var/www/html/xdebug.log
```

- zend_extension : xdebug 모듈의 경로를 지정한다.
- xdebug.mode : xdebug 의 동작 방식을 지정한다. 
- xdeubg.start_with_request : 클라이언트 요청이 있을 때 자동으로 디버깅 세션이 시작되도록 설정한다. 편의상 api 등을 테스트하기 위해서 반드시 yes로 설정한다.
- xdebug.discover_client_host : 클라이언트 호스트를 자동으로 감지하도록 설정한다.
- xdebug.idekey : 약속된 디버깅을 세션을 시작하기 위해 필요한 키 값
- xdebug.client_host : xdebug가 클라이언트 IDE를 찾기위한 호스트 주소, IDE가 도커 외부에 있을 경우엔 host.docker.internal로 지정한다.
- xdebug.client_port : xdebug와 클라이언트 IDE가 디버깅 정보를 주고 받을 포트 xdebug 3.0 이 후 기본값 9003을 사용한다. fpm을 사용하는 환경에선 9000포트는 피해야 한다.
- xdebug.log : xdebug 의 로그 파일이 생성될 경로를 지정한다.

#### 2. Dockerfile 수정

컨테이너가 다시 생성될 때마다 위에서 생성한 xdebug.ini가 복사될 수 있도록 dockerfile에 아래 내용을 추가한다.

```dockerfile
COPY xdebug.ini /etc/php/8.0/fpm/conf.d/20-xdebug.ini
```

만약, dockerfile이 존재하지 않다면, `sail artisan sail:publish` 명령어를 실행하여 sail 내부 dockerfile를 추출한 뒤 원하는 원하는 버전의 dockerfile을 골라 수정하면 된다. 물론 해당 버전은 docker-compose.yml 과 맞춰야 한다.

> **💡Tips.**
>
> 프로젝트 베이스가 Lumen 일경우 sail:publish 명령이 없을 수 있는데, 이 때는 임의의 Laravel 프로젝트를 생성 하여 위 명령어로 생성된 dockerfile을 복사해 가져오면 된다.



#### 3. 이미지 빌드

위에서 만든 dockerfile을 기반으로 도커 이미지를 빌드 한 후 컨테이너들을 생성한다.

```sh
docker build --no-cache docker up -d
```



#### 4. docker-compose.yml 수정

```yml
version: '3'
services:
  laravel.test:
    build:
        context: ./docker/8.0
        dockerfile: Dockerfile
        args:
            WWWGROUP: '${WWWGROUP}'
            XDEBUG: '${APP_DEBUG:-false}'   <---추가
    image: sail-8.0/app
```



### 컨테이너 설정

#### 1. 컨테이너 내부 xdebug 설정 확인

컨테이너가 생성되고 실행 되었으면, 컨테이너 내부로 진입에서 아래 명령어로 PHP의 xdebug 설정이 잘 적용 되었는지 확인한다.

```sh
php -v
```



#### 2. xdebug 모듈 로딩 확인

아래 명령어를 실행하여 Xdebug 와 Zend OPcache의 로딩 순서를 확인한다. 아래 사진처럼 Xdebug가 Zend OPcache보다 나중에 로딩되어야 한다.

![](https://github.com/djcho/draw-io-repo/assets/13410737/ca32d359-b176-403a-845a-187bb6006175){: .align-center width="500"}

그렇지 않을 경우 순서가 바뀌어 Xdebug가 먼저 출력되며 로그 파일에 두 모듈이 충돌했다는 경고와 함께 Xdebug 의 설정들이 정상적으로 로딩되지 않는다. 이럴 경우에는 PHP 설정 파일을 확인하여 로딩 순서를 조정해 줘야 하는데, 아래 명령어를 실행해 로드된 PHP 설정 파일들을 확인하고 zend_extension=opcache.so 구문이 우리가 설정한 zend_extension=xdebug보다 먼저 설정되고 있는지 확인한다. (* 파일 명 앞에 숫자가 높을 수록 나중에 로딩된다.)

```
php --ini
```

![](https://github.com/djcho/draw-io-repo/assets/13410737/673306f7-9a10-4836-8365-8a508ccb9e60){: .align-center width="500"}

#### 3.xdebug 설정 확인

아래 명령어를 실행하여 위에서 만든 xdebug.ini의 설정들이 잘 반영 되었는지 확인한다.

```sh
php -i | grep xdebug
```

![](https://github.com/djcho/draw-io-repo/assets/13410737/1fdd9c76-3bb5-457c-92b5-df5649157e89){: .align-center width="500"}



#### 4.php-fpm 사용시) fpm 에 xdebug 관련 설정 확인

Laravel 내장 서버(php artisan serve)를 사용하는 경우엔 php-fpm 를 사용하지 않아 대부분 윗 단계에서 디버깅 설정이 끝나지만, nginx + phpfpm 를 사용하는 경우엔 fpm에서 xdebug 설정을 제대로 로드하고 있는지 확인해야 한다. php-fpm 설정 파일 위치로 이동하여 우리가 설정한 xdebug.ini 가 존재하는지 확인한다. 위 dockerfile에서 php 설정 파일 경로의 20-xdebug.ini에 복사를 했기 때문에, php-fpm이 로드되면서 아래 이미지와 같이 20-xdebug.ini 파일이 심볼릭 링크로 연결되어 있어야 하며 내용 또한 위에서 설정한 내용이 잘 반영되어 있어야 한다.

```sh
cd /etc/php/{fpm-version}/fpm/conf.d
ls -al
```

![](https://github.com/djcho/draw-io-repo/assets/13410737/0187afaa-2d44-43b6-8914-0d3cf5b3a77b){: .align-center width="600"}



 ⚠️ 만약, xdebug.ini 의 설정과 php -v 로 출력되는 설정 모두 문제가 없음에도 원치 않는 동작을 일으킨다면 환경변수를 확인하라, 환경 변수에 XDEBUG_XXX 와 같은 변수가 존재한다면 php -v 에 출력된 설정보다 먼저 적용되는 경우도 있다.
{: .notice--warning}



### 로컬 IDE 설정

#### 1. VSCode xdebug 확장 플러그인 설치

이제 로컬 호스트 환경으로 돌아와 VSCode 에서 Xdebug를 사용하기 위해 "PHP Debug" 플러그인을 설치한다.

![](https://github.com/djcho/draw-io-repo/assets/13410737/fd72133a-9ab0-4fcb-9a32-7e731fe0b2ec){: .align-center width="800"}



#### 3. 디버깅 구성 추가

디버깅을 시작하려면 launch.json 파일을 설정하여 디버깅 환경을 구성해야 한다. 이 파일은 VSCode의 디버깅 탭에서 최초로 디버깅을 실행할 경우 자동로 프로젝트의 .vscode 폴더 내에 생성되며, 만약 폴더가 없다면 직접 생성한 뒤 아래 내용을 추가한다. 여기서 포트는 반드시 컨테이너 내부 Xdebug 설정에서 포트로 지정되어야 하며, 디버깅할 로컬 호스트의 소스코드와 도커 내부에서 웹서버로 실행되는 PHP 파일들의 위치 정보를 pathMappings 로 매핑시켜 줘야한다.

```json
{
  "version": "0.2.0",
  "configurations": [
    
      {
          "name": "Listen for Xdebug",
          "type": "php",
          "request": "launch",
          "port": 9003,
          "hostname": "localhost",
          "log": true,
          "pathMappings": {
            "/var/www/html/": "${workspaceFolder}"
        }
      }
  ]
}
```



## 테스트

### 1. IDE 중단점 추가 → 디버깅 서버 실행(F5)

![](https://github.com/djcho/draw-io-repo/assets/13410737/764d7f7f-c0b6-4dbd-96d1-f2bc3a4f7c26){: .align-center width="500"}



### 2. postman 또는 browser 에서 api 호출

![](https://github.com/djcho/draw-io-repo/assets/13410737/ec156cf8-7c6a-44be-ad5b-375afce52431){: .align-center width="800"}



### 3. 중단점 적중 여부 확인

![](https://github.com/djcho/draw-io-repo/assets/13410737/4f6c28e7-a258-446f-b505-843443fade07){: .align-center width="600"}
