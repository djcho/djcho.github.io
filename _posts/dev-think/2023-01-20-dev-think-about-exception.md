---
title : "[Exception] 예외 처리에 대한 고찰"
categories:
- DevThink
tag :
- [Common, Exception, Java]
toc: true
toc_sticky : true
published : true
date : 2023-01-20
last_modified_at : 2023-01-20
---



# 예외 처리에 대한 고찰

팀원 중 한 분께서 예외처리에 대한 고민을 늘어 놓으셨다. 주된 고민의 내용은 전반적인 예외 처리 방식에 대한 장단점과 예외 전환 시점에 대한 의문이었다. 이번 기회에 스스로 정리할겸 예외 처리에 대한 나름대로의 생각을 정리해 본다.



## 언제 예외를 던져야 하는가

언제 예외를 던져야 하는지 생각해 보려면 우선 예외가 무엇인지를 생각해 볼 필요가 있다. 예외란 무엇일까, 예외는 에러와 다르다. 예측이 불가하고 프로그램의 동작을 보장할 수 없게 되는 에러와는 다르게 예외는 개발자가 어느 정도 예측이 가능하며 적절한 처리로 프로그램의 동작을 보장시킬 수 있다. 예외를 한 마디로 정의한다면 아마도 "예측 가능한 수준의 의도치 않은 동작" 정도로 정의할 수 있을 것이다. 따라서 예외를 던져야 할 때는 "의도하지 않은 동작이 예측될 때"이다.



## `Exception`을 사용해야 하는 이유

`Exception`을 사용하면 사용하지 않았을 때보다 취할 수 있는 이점이 꽤 많다. 어떤 이점이 있는지 아래 예제 코드를 통해 알아본다.



### C 스타일의 예외처리

```java
int modifyCar(Car car){
    if(car == null){
        return -1;
    }
    car.setName = "myCar";
    ...
}

void BusinessLogic(){
    ...
    int result = modifyCar(car1);
    if(result != 0){
        logger.debug("modifyCar failed, result : ", result);
        return Error.MODIFY_CAR_FAILED;
    }
    result = modifyCar(car2);
    if(result != true){
        logger.debug("modifyCar failed, result : ", result);
        return Error.MODIFY_CAR_FAILED;
    }
    ...
}
```

위 코드는 `Exception` 이 존재하지 않는 C언어 처럼 예외의 발생 유무를 함수의 리턴 값을 사용하도록 작성된 코드이다. 매 함수를 호출하면서 반복적인 코드로 예외검사를 수행할 수밖에 없기 때문에 비지니스 코드가 비효율적으로 늘어나게 가독성에도 좋지 않다. 또한, 예외 검사 시점에 어떤 이유로 예외가 발생했는지 확인하려면 내부 코드를 직접 보고 어떤 예외 코드가 발생했는지 대조하여 확인해야 하기 때문에 개발 생산성에도 악영향을 끼치게 된다.



### `Exception` 의 장점

```java
void modifyCar(Car car){
    if(car == null){
        throw new IllegalArgumentException("input car is null");
    }
    car.setName = "myCar";
    ...
}

void BusinessLogic(){
    try{
        modifyCar(car1);
        modifyCar(car2);
    }catch(Exception ex){
        //일괄 처리
        logger.debug("modifyCar failed, Exception message : " + ex.getMessage());
        throw new CarException("Modify car failed", ex)
    }  
}
```

위 코드는 `Exception`을 사용했을 때의 코드이다. 불필요하게 반복되던 리턴 값 체크 로직이 없어져 한결 가독성이 좋아졌다. try-catch 문으로 발생한 예외를 일괄로 처리할 수 있게 되었으며 `Exception` 객체로 예외 검사 시점에 어떤 이유에서 예외가 발생했는지 문자열로 즉시 확인이 가능해졌다. 



## 예외를 던지는 방법

### 커스텀 예외의 필요성

비즈니스 로직 같은 경우에는 자바에서 제공하는 표준 예외만을 사용하여 충분한 정보의 예외를 전달할 수 없는 경우가 존재한다. 예를 들어 얼굴 캡처 라이브러리를 구현하는 도중 카메라가 오픈되지 않은 상태에서 캡처가 시도될 경우를 예측하여 예외를 던져야할 때 자바에서 지원하는 일반 표준 Exception 만으로는 예외를 던진다고 하면 아래와 같은 코드일 것이다.

```java
if(!camera.isOpened()){
    String message = camera.toString() + "etcInfo1 : " + etcInfo1 + "etcInfo2" + etcInfo2;
    throw new Exception("camera is not opened " + message);
}
```

내부 카메라 객체의 값을 `toString()`메서드를 사용해 제공할 수 있겠지만 매번 예외처리가 필요한 부분마다 가독성을 해치는 코드가 될 가능성이 크고 외부 값과 같이 제공하고 싶다면 기본 예외 클래스로는 이를 해결할 방법이 없다. 실무에서 예외의 정보는 얼마만큼 빠르게 이슈를 해결하느냐와도 직결되기 때문에 이럴 경우 커스텀 예외 클래스를 정의하는 것이 좋을 수 있다.

```java
if(!camera.isOpened()){
    throw new CameraException("camera is not opened", camera, etcInfo1, etcInfo2);
}
```

커스텀 예외의 가장 큰 장점은 클래스의 이름을 보고도 개발자가 어느정도 어떤 예외가 발생했는지를 예상할 수 있다는 점과 호출하는 쪽에서 커스텀 예외만을 위한 특별한 처리가 가능하다는 점이다.

```java
try{
    //예외 발생
}catch(CameraException ex){
    System.out.println("카메라 장치 관련 예외가 발생하였습니다.");
}
catch(CaptureException ex){
    System.out.println("캡처 관련 예외가 발생하였습니다.");
}
...
catch(Exception ex){
    System.out.println("예상하지 못한 예외가 발생하였습니다.");
}
```



### 예외 전환

예외 전환은 받은 예외를 잡아 새로운 예외로 다시 던지는 것을 말하지만 중요한 요점은 **자신만의 문맥으로 재해석한 새로운 예외를 던져**야한다는 것이다. 예외 전환을 적절히 사용한다면 위 코드처럼 비지니스 로직에서 예외를 일괄적으로 처리하기 용이해지고 계층적인 예외 구조로 예외 발생 시점에 어떤 예외가 어떤 이유에서 발생했는지 쉽게 확인이 가능해진다. 

```java
try{
    ...
}catch(Exception ex){
    throw new CarException("Modify car failed", ex); //예외 전환
}  
```



#### 예외 전환이 필요할 때

일반적인 애플리케이션의 구조를 개략적으로 그린다면 아래와 같을 것이다. 일반적인 기능을 제공하여 여러 위치에서 사용할 수 있는 제네럴 기능들과, 이 제네럴 기능들을 사용하는 비지니스 기능들이 모여 도메인을 구성하고, 하나의 제품 혹은 서비스가 완성된다.

<p align="center">
  <img alt="img-name" src="https://user-images.githubusercontent.com/13410737/214315434-4037ea5c-e7e4-4039-b09f-6d7ad9c725f4.png">
  <br>
    <em>일반적인 애플리케이션 구조</em>
</p>



애플리케이션들의 기능들은 아래와 같이 논리적인 기능 레이어로 구분할 수 있고 각 기능들은 각자 자신들만의 독자적인 문맥으로 기능들을 정의하고 있을 것이다. 예외 전환은 앞서 언급한것 처럼 단순히 예외를 잡아 새로운 예외로 다시 던지는 것을 의미하는 것이 아닌 각 기능들의 문맥으로 재해석하여 예외를 던지는 걸 말한다. 따라서 예외 전환이 필요한 시점은 바로 서로 다른 기능들 간의, 혹은 레이어간의 경계를 통과해야 할 때 수행되어야 한다.

<p align="center">
  <img alt="img-name" src="https://user-images.githubusercontent.com/13410737/214321681-91068778-2da2-4809-be54-5f1224e055ba.png">
  <br>
    <em>애플리케이션 기능 레이어</em>
</p>



위 내용의 예시를 들자면 아래 그림과 같다. 인증 서비스를 제공하는 ATH라는 서비스가 존재한다고 가정했을 때, 사용자가 OTP 인증을 수행 중에 숫자가 아닌 값을 입력했을 경우  문자열이 숫자임을 판단해주는 제네럴 코드에서는 입력된 문자열이 숫자가 아니라는 `IllegalArgumentException` 예외를 던지고, OTP 기능을 수행하는 비지니스 코드에서는 자신의 기능 문맥에 맞게 `OtpAuthException`이라는 커스텀 예외를 던진다. 그럼 인증을 통합하여 담당하는 레이어에서는 `AuthenticationException` 으로 인증에 실패했다는 메시지를 출력한다. 이럴 경우 인증 레이어에서는 각 인증 기능별로 예외를 일괄적으로 처리할 수 있게 되고 각 기능별로 예외를 계층적으로 관리할 수 있다.

<p align="center">
  <img alt="img-name" src="https://user-images.githubusercontent.com/13410737/214331016-fdf4df12-39ad-4fa0-b2f0-19f0c29388ea.png">
  <br>
    <em>예외 전환의 예</em>
</p>



## 정리

- C스타일의 예외 처리 방식은 코드 가독성을 떨어트리므로 try-catch를 사용하자.
- 예외를 사용하면 각 예외별 일괄 처리가 가능하여 반복적인 코드의 양을 줄일 수 있다.
- 비지니스 로직에서 효율적으로 예외처리를 하기 위해 커스텀 예외를 사용하자.
- 예외 전환의 시점은 기능 레이어을 통과할 때이며 새로운 예외를 던질 때 자신만의 문맥으로 재해석한 예외로 던지자.





