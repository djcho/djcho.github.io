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

팀원 중 한 분께서 예외처리에 대한 고민을 늘어 놓으셨다. 주된 고민의 내용은 언제 기능의 흐름을 끊어 내면서까지 throw를 던져야 하는지와 어떨 때 발생된 예외를 catch 하여 하는지 판단하기 기준을 잘 모르겠다는것이었다. 이번 기회에 스스로 정리할겸 예외처리에 대한 나름대로의 생각을 정리해 본다.



## 언제 예외를 throw 해야 하는가

언제 예외를 던져야 하는지 생각해 보려면 우선 예외가 무엇인지를 생각해 볼 필요가 있다. 예외란 무엇일까, 예외는 에러와 다르다. 예측이 불가하고 프로그램의 동작을 보장할 수 없게 되는 에러와는 다르게 예외는 개발자가 어느 정도 예측이 가능하며 적절한 처리로 프로그램의 동작을 보장시킬 수 있다. 예외를 한 마디로 정의한다면 아마도 "예측 가능한 수준의 의도치 않은 동작" 정도로 정의할 수 있을 것이다. 따라서 예외를 던져야 할 때는 "의도하지 않은 동작이 예측될 때"이다.



## Exception을 사용해야 하는 이유

예외를 사용하면 사용하지 않았을 때보다 취할 수 있는 이점이 꽤 많다. 어떤 이점이 있는지 아래 예제 코드를 통해 알아보자.



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

위 코드는 Exception 이 존재하지 않는 C언어 처럼 예외의 발생 유무를 함수의 리턴 값을 사용하도록 작성된 코드이다. 매 함수를 호출하면서 반복적인 코드로 예외검사를 수행할 수밖에 없기 때문에 비지니스 코드가 비효율적으로 늘어나게 되고 매번 리턴 값을 확인하는 로직 때문에 가독성에도 좋지 않다. 또한, 예외 검사 시점에 어떤 이유로 예외가 발생했는지 확인하려면 내부 코드를 직접 보고 어떤 예외 코드가 발생했는지 직접 눈으로 보고 대조하며 확인해야 하기 때문에 개발 생산성에도 악영향을 끼치게 된다.



### Exception 의 장점

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

위 코드는 Exception을 사용했을 때의 코드이다. 불필요하게 반복되던 리턴 값 체크 로직이 없어졌으며 한결 가독성이 좋아졌다. try-catch 문으로 발생한 예외를 일괄로 처리할 수 있게 되었으며 예외 객체를 사용하여 예외 검사 시점에 어떤 이유에서 예외가 발생했는지 문자열로 즉시 확인이 가능해졌다. 



## 예외를 throw하는 방법

### 커스텀 Exception 의 필요성

비즈니스 로직 같은 경우에는 자바에서 제공하는 일반 표준 예외만을 사용하여 충분한 정보의 예외를 전달할 수 없는 경우가 존재한다. 예를 들어 얼굴 캡처 라이브러리를 구현하는 도중 카메라가 오픈되지 않은 상태에서 캡처가 시도될 경우를 예측하여 예외를 던져야할 때 자바에서 지원하는 일반 표준 Exception 만으로는 예외를 던진다고 하면 아래와 같이 던질 수 있을 것이다.

```java
if(!camera.isOpened()){
    String message = camera.toString() + "etcInfo : " + etcInfo1 + "etcInfo2" + etcInfo2;
    throw new Exception("camera is not opened " + message);
}
```

예외의 정보라고는 카메라가 열리지 않았다는 메시지의 내용만 있을 뿐이다. 내부 카메라 객체의 값을 `toString()`메서드를 사용해 제공할 수 있겠지만 매번 예외처리가 필요한 부분마다 가독성을 해치는 코드가 될 가능성이 크고 외부 값과 같이 제공하고 싶다면 기본 예외 클래스로는 이를 해결할 방법이 없다. 실무에서 예외의 정보는 얼마만큼 빠르게 이슈를 해결하느냐와도 직결되는 정보로 최대한 많은 정보를 제공하는 편이 좋기 때문에 이럴 때 커스텀 예외 클래스를 정의하여 사용한다.

```java
if(!camera.isOpened()){
    throw new CameraException("camera is not opened", camera, etcInfo1, etcInfo2);
}
```

커스텀 Exception의 가장 큰 장점은 클래스의 이름을 보고도 어느정도 개발자가 어떤 예외가 발생했는지를 예상할 수 있는 점과 호출하는 쪽에서 커스텀 예외만을 위한 특별한 처리를 가능하게 해준다는 점이다.

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



### 예외를 잡고 던지는 기준

#### 예외 전환이 필요할 때

예외 전환이 필요할 때

예외 전환은 레이어의 외부로 노출 되어 있는 메서드들에서 한다.

예외 전환 시에는 해당 객체나 기능의 컨텍스트에 맞게 재해석하여 던진다.



## 정리







