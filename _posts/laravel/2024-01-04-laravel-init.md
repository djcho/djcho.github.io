---
title : "[Laravel] 라라벨 개발 환경 구성"
categories:
- Laravel
tag :
- [Laravel, PHP, Composer, Sail]
toc: true
toc_sticky : true
published : true
date : 2024-01-04
last_modified_at : 2024-01-04
---



이현석님의, [이현석의 라라벨 입문] 영상 강의를 보고 정리한 필기입니다.📢
{: .notice--warning}

# 라라벨 개발 환경 구성

라라벨의 기본 기능을 익히고 간단한 웹어플리케이션을 구축해보기 위한 최초 개발 환경을 구축하는 방법을 정리한다.

## 필수 구성 요소

라라벨은 PHP기반 풀 스택 프레임워크이기 때문에 PHP와 NodeJs(와 NPM)가 설치되어 있어야 한다. PHP는 아래 명령어로 실행하여 설치하고 NodeJs는 [NodeJs 공식 페이지]에서 다운로드 받아 설치한다.

[NodeJs 공식 페이지]: https://nodejs.org/en

```shell
brew install php
```



### 컴포저

컴포저란 PHP에서 종속성을 관리해주는 도구로 프로젝트에서 의존하고 있는 라이브러리를 관리할 수 있게 해준다. 컴포저는 일반적으로 Yum이나 Apt와 같은 패키지 매니저가 아니다. 패키지와 라이브러리를 다루지만 각 프로젝트 내 `vendor`라는 폴더에 내에서 설치하며 관리한다. 예를 들어 어떤 프로젝트에서 여러 라이브러리에 의존하고 라이브러리들이 또 각각의 다른 라이브러리에 의존할 때 컴포저는 의존하는 라이브러리를 선언하고 필요한 버전을 찾아 프로젝트 내에 다운로드 받아 설치한다. 또한 이런 모든 종속성을 한 명령어로 업데이트도 가능하다.

컴포저 설치 방법은 [Composer 공식 페이지]를 참조해 설치하도록 한다.

[Composer 공식 페이지]: https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos





### Sail

Sail은 라라벨의 기본 도커 구성과 상호 작용할 수 있게 도와주는 경량 명령줄 인터페이스이다. Sail에 대한 모든 것은 Laravel에 포함된 docker-compose.yml 파일을 사용하여 사용자 정의할 수 있다. 기본적으로 Sail은 `docker-compose.yml` 파일이며 프로젝트의 루트에 저장된 `sail` 스크립트로 제공된다. 

#### Sail이 포함된 프로젝트 생성 방법

Mac에서 [Docker Desktop]이 이미 설치되어 있다면 아래와 같은 간단한 명령어 한줄로 새 라라벨 프로젝트를 만들 수 있다. 여기서 `example-app`은 프로젝트 이름이기 때문에 임의의 이름으로 변경 가능하다.

```sh
curl -s "https://laravel.build/example-app" | bash
```

[Docker Desktop]: https://www.docker.com/products/docker-desktop/



#### 기존 프로젝트에 Sail 추가 방법

Sail은 위에서 설치한 Composer 를 사용하여 설치한다. 기존 Laravel 프로젝트가 존재할 경우 아래와 같이 Sail을 설치할 수 있다.

```sh
composer require laravel.sail --dev
```

Sail 이 설치된 후에 `sail:install` Artisan 명령어를 실행하여 `docker-compose.yml` 파일을 프로젝트 루트에 추가한다.

```sh
php artisan sail:install
```

그후 아래 명령으로 Sail 을 시작하여 컨테이너들을 실행시킬 수 있다.

```sh
./vendor/bin/sail up
```



### IDE 설치

기본적인 라라벨 프로젝트를 생성했으니 코드를 수정할 수 있는 IDE를 설치한다. IDE은 [Visual Studio Code]를 사용하고 확장 프로그램 검색에서 라라벨 관련된 확장 플러그인을 설치한다. 

[Visual Studio Code]: https://code.visualstudio.com/download



## 프로젝트 생성

VSCode를 열고 아래와 같이 터미널에서 새로운 라라벨 프로젝트를 생성한다. 프로젝트 이름은 `ot-laravel-basic` 으로 지정한다.

```sh
curl -s "https://laravel.build/ot-laravel-basic" | bash
```

프로젝트가 생성이 되면 아래 명령어로 Sail을 실행한다.

```sh
cd ot-laravel-basic
./vendor/bin/sail up
```

![image](https://github.com/djcho/ot-laravel-basic/assets/13410737/6051e078-2ec8-4913-98ce-8076dc3edafc)

⚠️ Sail 실행 시 포트 충돌로 인해 에러가 발생할 경우 루트 폴더의 `docker-compose.yml` 에서 충돌이 나는 포트를 찾아 수정한 뒤 다시 시도한다.
{: .notice--warning}

이렇게 프로젝트를 만들면 개발 환경이 로컬에 생성된것이 아니라 도커 컨테이너로 실행된것이다. `docker ps` 로 컨테이너를 확인하면 아래와 같이 컨테이너로 라라벨 웹 어플리케이션이 구동 중인것을 확인할 수 있다.![image](https://github.com/djcho/ot-laravel-basic/assets/13410737/974931ef-d17a-4238-85b6-1f93223537a1)

`docker exec -it 875ce786d95a /bin/bash` 명령어로 도커 인스턴스 내부로 들어가서 특정 명령어를 수행하는것이 일반적이지만, Sail 명령어를 앞에 붙히면 굳이 내부로 들어가지 않도록 외부에서 바로바로 명령어 실행이 가능하다. 

![image](https://github.com/djcho/ot-laravel-basic/assets/13410737/fcc3825d-8abf-4b09-ad78-8a27db6b65ff)

![image](https://github.com/djcho/ot-laravel-basic/assets/13410737/5f6c294c-c864-4781-927b-8938d867e29f)

이렇듯 Sail 명령어는 도커 내부의 실행 결과를 응답으로 받아 사용자에게 편의성을 제공해준다.

### 프로젝트 구조

![image](https://github.com/djcho/ot-laravel-basic/assets/13410737/4b17dd5b-c102-424f-b88d-7b4fe1e7bfaa)

프로젝트 생성 시 구조는 위 그림과 같고 각 폴더마다 설명을 아래와 같다.

- app : 애플리케이션의 핵심 코드가 들어있다. 애플리케이션의 거의 모든 클래스가 이곳에 존재한다.
- bootstrip : 애플리케이션의 부트 환경설정을 담당하는 app.php가 존재하는 폴더
- config : 애플리케이션의 설정 파일을 포함한다.
- database : 데이터베이스 마이그레이션 파일, 모델 팩토리, 시딩 파일들을 포함한다. 이 폴더를 SQLite 가 저장되는 곳으로 사용할 수도 있다.
- public : 애플리케이션에 진입하는 모든 request요청들에 대한 진입점 역할과 오토로딩을 설정하는 `index.php`파일을 가지고 있다. 또한 이미지나 자바스크립트, CSS와 같은 asset파일들도 포함되어 있다.
- resources : 뷰 파일과 CSS, 자바스크립트와 같이 컴파일 되기 전의 asset파일들을 가지고 있고 다국어 파일도 있다.
- routes : 애플리케이션에서 정의된 모든 라우트들이 존재한다.
- storage : 애플리케이션의 로그, 컴파일된 블레이트 템플릿, 파일 기반의 세션, 파일 캐시 그리고 기타 프레임워크에서 생성된 파일들을 가지고 있다.
- tests : 자동화된 테스트가 포함되어 있다. `PHPUnit`단위 테스트와 기능 테스트 예제가 제공된다.
- vendor : `Composer` 의 의존송 폴더이다.

