---
title : "[Laravel] 라라벨 개발 환경 구축 : Debugging with VSCode"
categories:
- Laravel
tag :
- [Laravel, PHP, Debugging]
toc: true
toc_sticky : true
published : true
date : 2024-02-07
last_modified_at : 2024-02-07
---



이현석님의, [이현석의 라라벨 입문] 영상 강의를 보고 정리한 필기입니다.📢
{: .notice--warning}

# 라라벨 개발 환경 구축 : Debugging with VSCode

디버깅은 소프트웨어 개발 과정에서 불가피한 부분으로, 코드의 오류를 신속하게 찾아내고 수정하는 데 큰 도움을 준다. 많은 개발자들이 사용하는 Visual Studio Code (VSCode)를 활용하여 라라벨 애플리케이션을 디버깅하는 방법 정리한다.



## Xdebug 설치 및 활성화

### Xdebug 설치 여부 확인 방법

```bash
php -v
```

```bash
PHP 8.3.1 (cli) (built: Dec 20 2023 12:44:38) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.3.1, Copyright (c) Zend Technologies
    with Xdebug v3.3.1, Copyright (c) 2002-2023, by Derick Rethans
    with Zend OPcache v8.3.1, Copyright (c), by Zend Technologies
```

위 명령어 실행 시  `with Xdebug v3.3.1, Copyright (c) 2002-2023, by Derick Rethans` 와 같이 Xdebug 관련 내용이 출력된다면 Xdebug가 설치되어 있는 것이고 그렇지 않다면 설치되어 있지 않은 것이다.

⚠️ Xdebug와 PHP의 버전 호환성은 [Xdebug 공식 페이지]에서 확인가능하다.
{: .notice--warning}

[Xdebug 공식 페이지]: https://xdebug.org/docs/compat



### Xdebug 설치

```bash
pecl install xdebug
```

### Xdebug 활성화 

Xdebug가 설치되었으면 현재 본인이 사용하고 있는 php에서 Xdebug를 활성화 해야 디버깅이 가능하다. 아래 명령을 실행하여 php.ini 파일의 위치를 확인하고 수정해야 한다.

#### php.ini 파일 위치 확인 방법

```bash
php --ini
```

```bash
Configuration File (php.ini) Path: /opt/homebrew/etc/php/8.3
Loaded Configuration File:         /opt/homebrew/etc/php/8.3/php.ini
Scan for additional .ini files in: /opt/homebrew/etc/php/8.3/conf.d
Additional .ini files parsed:      /opt/homebrew/etc/php/8.3/conf.d/ext-opcache.
```

2번째 줄 `Loaded Configuration File` 에 출력되는 php.ini 파일을 열어 아래 값을 추가한다.

```ini
zend_extension="xdebug.so"

;add xdebug configuration
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_port=9000
xdebug.client_host=127.0.0.1
xdebug.log=xdebug.log
```

- `zend_extention` :  값은 설치한 Xdebug 모듈의 경로(기본으로 xdebug.so 로 설정되어 있다. 만약, 경로를 찾지 못한다면 환경 변수를 확인한다.)  
- `xdebug.mode` : Xdebug의 모드를 설정한다. `develop`, `coverage`, `debug`, `profile`, `trace` 등 여러 모드를 지원한다.
- `xdebug.start_with_request` : 요청이 있을 때 Xdebug를 자동으로 시작할지 여부를 설정한다. 이 값이 `yes`로 되어 있어야 외부에서 API를 통해 호출될 경우 디버깅을 할 수 있게 된다.
- `xdebug.client_port` : Xdebug와 VSCode간의 디버깅 데이터를 주고 받을 때 사용되는 포트이다. 이 값은 VSCode 의 launch.json에 작성된 포트와 일치해야 한다.
- `xdebug.client_host` : Xdebug와 VSCode간의 디버깅 데이터를 주고 받을 때 사용되는 호스트 주소이다. 이 값은 VSCode 의 launch.json에 작성된 호스트 주소와 일치해야 한다. 
- `xdebug.log` : Xdebug의 로깅 파일 출력 위치를 결정 한다.



## vsCode 설정

### PHP Debug 확장 설치

VSCode의 확장 프로그램 탭에서 "PHP Debug" 확장을 찾아 설치한다.

<img width="1399" alt="image" src="https://github.com/djcho/api-laravel-basic/assets/13410737/8f9fdf4f-1f50-4260-8887-8c336ffb5994">

### 브레이크포인트 설정과 디버깅 실행

#### launch.json 작성

![launch.json](https://github.com/djcho/api-laravel-basic/assets/13410737/a5d0330d-4ec4-4176-9a16-ca158a76de2c){: .align-center width="300"}

디버깅을 시작하려면 `launch.json` 파일을 설정하여 디버깅 환경을 구성해야 한다. 이 파일은 VSCode의 디버깅 탭에서 최초로 디버깅을 실행할 경우 자동로 프로젝트의 `.vscode` 폴더 내에 생성된다. 만약 폴더가 없다면 직접 생성한 뒤 아래 내용을 추가한다.


```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for XDebug",
      "type": "php",
      "request": "launch",
      "port": 9000,
      "hostname": "localhost"
    }
  ]
}
```



딱히 많은 설정이 필요하지 않다. 중요한 부분은 `port`와 `hostname`은 위 php.ini 내 Xdebug 설정과 일치 해야 한다. 

> 만약, 라라벨 애플리케이션을 로컬에서 개발하면서 실제 서버에 업로드하여 실행하는 경우, 로컬 파일 경로와 서버 파일 경로 간의 차이가 발생하여 디버깅에 어려움이 있다면 `pathMappings` 값을 이용하면 해결가능하다고 한다.

`php artisan serve` 를 수행 후 디버깅을 원하는 코드에서 중단점을 설정한 뒤 F5를 눌러 디버깅 세션을 시작하면 된다.

![](https://github.com/djcho/api-laravel-basic/assets/13410737/ab3fe6e7-c5e3-4e53-9988-e38a9c35a639){: .align-center width="500"}

<img width="1243" alt="스크린샷 2024-02-07 17 35 07" src="https://github.com/djcho/api-laravel-basic/assets/13410737/c40e3d18-4e0c-4661-9c47-7bcaba23e178">



## Reference

[https://xdebug.org/docs/install](https://xdebug.org/docs/install){:target="_blank"}

[https://stackoverflow.com/questions/51685952/debugging-laravel-application-on-vscode](https://stackoverflow.com/questions/51685952/debugging-laravel-application-on-vscode){:target="_blank"}

[https://medium.com/@alirazalilani/debugging-php-laravel-with-visual-studio-code-37b756fb6c19](https://medium.com/@alirazalilani/debugging-php-laravel-with-visual-studio-code-37b756fb6c19){:target="_blank"}
