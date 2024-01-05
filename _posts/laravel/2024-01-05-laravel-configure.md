---
title : "[Laravel] 프로젝트 설정"
categories:
- Laravel
tag :
- [Laravel, PHP]
toc: true
toc_sticky : true
published : true
date : 2024-01-05
last_modified_at : 2024-01-05
---



이현석님의, [이현석의 라라벨 입문] 영상 강의를 보고 정리한 필기입니다.📢
{: .notice--warning}

# 프로젝트 설정

라라벨의 설정을 배워본다. 라라벨의 모든 설정은 기본적으로 `config` 디렉토리에서 관리된다. `config` 폴더에 저장되어 있는 설정 값들은 `config` 라는 메서드로 불러올 수 있다.![image](https://github.com/djcho/ot-laravel-basic/assets/13410737/71f2618b-746a-465c-8b81-7b2fd30f5f80)

또한 `env()`는 `.env`파일에 정의되어 있는 설정을 불러올 수 있는 메서드이다. 

- `/config` : 애플리케이션 동작에 관련된 설정을 관리하는 곳
- `.env` : 애플리케이션이 구동됨에 있어 필요한 환경을 설정하는 파일



⚠️ `env()` 메서드는 `config/`폴더 내의 코드에서만 사용해야 한다. 라라벨은 성능을 높히기 위해 설정 값들은 캐싱하곤 하는데 캐시된 결과가 `config/`폴더에만 반영되기 때문에 다른 곳의 소스에서 사용하게 null이 반환될 수 있어서 개발환경에선 동작하나 운영 환경에서는 오류가 발생할 수 있으므로 주의해야 한다.
{: .notice--warning}



