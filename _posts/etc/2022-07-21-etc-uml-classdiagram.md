---
title : "[UML]클래스 다이어그램"
categories:
- Etc
tag :
- [Etc, UML, Java, ClassDiagram]
toc: true
toc_sticky : true
published : true
date : 2022-07-21
last_modified_at : 2022-07-21
---


## 클래스 다이어그램 작성법
많은 개발자들이 오랫동안 클래스 다이어그램을 사용해 왔다. 하지만 클래스 다이어그램을 그릴 때 마다 항상 혼란스러웠던 경험은 모두 있었을 거라 생각된다. 이 포스트에서는 그런 사람들에게 도움이 되고자 간단하게 UML 과 클래스다이어그램의 소개 하고 구조 다이어그램인 클래스 다이어그램의 작성 법을 Java 샘플을 통해 알아본다. 
<br>

### UML

UML이란 Unified Modeling Language 의 약어로 1997년 OMG(Object Management Group)에서 표준으로 채택한 통합 모델링 언어이다. 즉, 모델을 만드는 표준 언어이다. UML의 큰 목적은 다음과 같다.

- 다른 사람들과의 의사소통 또는 설계 논의
- 전체 시스템의 구조 및 각 컴포넌트 간 의존성 파악
- 유지보수를 위한 설계의 back-end 문서

UML은 크게 시스템의 개념, 관계 등을 나타내고 각 요소들의 정적인 면을 보기 위한 구조 다이어그램과  각 요소들의 변화나 흐름, 주고 받는 데이터 등의 동작을 보기 위한 행위 다이어그램이 존재한다. 각 유형에는 7개 씩 총 14개의 세부 다이어그램이 존재하며 클래스 다이어그램은 구조 다이어그램에 속한다.

<br>

### 클래스 다이어그램
클래스 다이어그램은 클래스 내부의 정적인 내용이나 클래스 사이의 관계를 표현하는 다이어그램으로 시스템의 일부 또는 전체 구조를 나타낼 수 있다. 클래스 다이어그램은 의존 관계를 명확히 보여주며 순환 의존이 발생하는 지점을 찾아낼 수도 있다. 클래스 다이어그램의 모양새는 아래 그림과 같다.

![image](https://user-images.githubusercontent.com/13410737/180127877-7e6ed339-f30f-4ac2-848d-f09feab418cc.png){: .align-center}

<br>

### 클래스 표현
클래스의 세부사항들을 상세하게 적는 것이 유용할 때도 있지만 UML 다이어그램은 필드나 메서드를 모두 선언하는 곳이 아니기 때문에 다이어 그램을 그리는 목적에 필요한 것만 사용하는 것이 좋다. 
<br>

```java
public class Person {
    private String name;
    private int age;
    private String email;

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

<br>
클래스는 보통 3개의 구역으로 나누어 클래스의 이름, 속성 기능을 표기한다. 속성과 기능은 옵션으로 생략이 가능하지만 이름은 필수로 명시해야 한다. 클래스의 세부 사항은 필드와 메서드의 가시성, 필드명, 데이터타입, 매개변수, 리턴 타입 등을 나타낼 수 있다. 위 클래스에 대한 클래스 다이어그램은 아래와 같다. 
<br>
![image](https://user-images.githubusercontent.com/13410737/180129913-cbbda2bb-14c0-4416-a0a2-a809b8d30a08.png){: .align-center}
<br>
속성과 가시성은 아래와 같이 표시한다.
<br>
| 키워드 | 표시       | 설명                                             |
| ------ | ---------- | ------------------------------------------------ |
| static | _          | 필드나 메서드에 언더바를 적용한다.               |
| final  | {readOnly} | 필드나 메서드 뒤에 {readOnly} 키워드를 추가한다. |
<br>
| 접근 제어자 | 표시 | 설명                                                         |
| ----------- | ---- | ------------------------------------------------------------ |
| public      | +    | 어떤 클래스의 객체에서든 접근 가능                           |
| privet      | -    | 이 클래스에서 생성된 객체들만 접근 가능                      |
| protected   | #    | 이 클래스와 동일 패키지에 있거나 상속 관계에 있는 하위 클래스의 객체들만 접근 가능 |
| package     | ~    | 동일 패키지에 있는 클래스의 객체들만 접근 가능               |
<br>
또한 매개변수의 방향을 명시할 필요가 있을 경우에는 아래와 같시 표시할 수 있다.
<br>

| 표시  | 설명                                      |
| ----- | ----------------------------------------- |
| in    | 입력 매개 변수 일 경우                    |
| inout | 입력 및 출력 모두 가능한 매개 변수일 경우 |
| out   | 출력 매개 변수일 경우                     |

<br>

### 스테레오 타입
스테레오 타입이란 UML에서 제공하는 기본 요소 외에 추가적인 확장요소를 나타내는 것으로 길러멧(guillemt, ≪≫)사이에 적는다.  이 길러멧이란 기호는 쌍 꺾쇠와는 다른 것으로 일반 업무에서 사용할 때는 쌍꺾쇠를 사용해도 좋지만 공식적인 문서라면 이 기호를 사용하는 것이 좋다.

```java
public interface Developer {
    public void writeCode();
}
```

![image](https://user-images.githubusercontent.com/13410737/180132144-21951a04-c564-4848-a185-fcbbc4550095.png){: .align-center}

<br>

```java
public class Math {
    public static final double PI = 3.14159;
    
    public static double sin(double theta){
        return 0;
    }
    public static double cos(double theta){
        return 0;
    }
}
```

![image](https://user-images.githubusercontent.com/13410737/180132991-98cef830-3ca7-4113-92bd-72dc90305dce.png){: .align-center}

<br>

스테레오 타입으로 많이 사용되는 것은 **`« interface »`**, **`« utility »`**, **`« abstract »`**, **`« enumeration »`** 등이 있다.

<br>

### 추상 클래스에서의 표현

추상 클래스에서의 표현은 이름과 메서드는 italic 체나, 필드 및 메서드 뒤에 `{abstract}` 프로퍼티를 사용하여 표기한다. 

```java
public abstract class Person {
    public abstract  void walk();
}
```

![image-20220721140634989](C:\Users\djcho\AppData\Roaming\Typora\typora-user-images\image-20220721140634989.png){: .align-center}

<br>

### 클래스 관계
클래스 다이어그램의 주 목적은 클래스간의 관계를 한눈에 쉽게 보고 의존 관계를 파악하는 것에 있다. 표현할 수 있는 관계는 아래와 같다. 
<br>

| 관계                           | 표기                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| 일반화(Generalization)         | ![image](https://user-images.githubusercontent.com/13410737/180134480-0f6ecf38-7395-40ce-acd6-15a74b0ad9c8.png){: .align-center} |
| 실체화(Realization)            | ![image](https://user-images.githubusercontent.com/13410737/180135031-b93c62ef-7f51-44fb-b827-a69d6864d3d2.png){: .align-center} |
| 의존(Dependency)               | ![image](https://user-images.githubusercontent.com/13410737/180135533-1a5e5546-2255-423f-be84-741796e47c61.png){: .align-center} |
| 연관(Association)              | ![image](https://user-images.githubusercontent.com/13410737/180135645-a8b20d3e-3fe9-4b38-b152-f0afe43a8cb7.png){: .align-center} |
| 직접연관(Directed Association) | ![image](https://user-images.githubusercontent.com/13410737/180135712-bcf52cf3-693b-4f22-a767-409c259d33b3.png){: .align-center} |
| 집합, 집합연관(Aggregation)    | ![image](https://user-images.githubusercontent.com/13410737/180135776-e5b7a260-af33-47ce-b0a0-ed2d2e8a0078.png){: .align-center} |
| 합성, 복합연관(Composition)    | ![image](https://user-images.githubusercontent.com/13410737/180135832-a33e5d2d-d33d-4c43-b065-2a940d8c7b8a.png){: .align-center} |

#### 일반화(Generalization)
일반화 관계는 슈퍼 클래스와 서브 클래스간에 상속 관계를 나타낸다. 여기서 일반화란 서브 클래스가 주체가 되어 서브 클래스를 슈퍼 클래스로 일반화 하는 것을 말하고 반대의 개념은 슈퍼 클래스를 서브 클래스로 Specialize(구체화)하는 것이다.
<br>

```java
public class User {
    private String id;
    private String password;
    
    public void login(){
    }
}
```

```java
public class Customer extends User {
    @Override
    public void login(){
    }
}
```

```java
public class Admin extends User {
    @Override
    public void login(){
    }
}
```

![image](https://user-images.githubusercontent.com/13410737/180137232-f8ed4067-b737-4f14-8b38-fc8960530708.png){: .align-center}
<br>

#### **실체화(Realization)**
실체화 관계는 interface의 스펙만 있는 메서드를 오버라이딩 하여 실제 기능을 구현하는 것을 말한다.

<br>
```java
public interface Service{
    public Object getObject();
}
```
```java
public class ServiceImpl implements Service{
    @Override
    public Object getObject(){
        return new Object();
    } 
}
```
![image](https://user-images.githubusercontent.com/13410737/180138647-2547c9cf-c9a8-414f-9d44-af2ae4a27b2e.png){: .align-center}
<br>
Realization을 나타내는 표기법은 2가지가 있다. 첫 번째는 인터페이스를 클래스처럼 표기하고 스테레오 타입 « interface » 를 추가한다. 그리고 인터페이스와 클래스 사이의 Realize 관계는 점선과 인터페이스 쪽의 비어있는 삼각형으로 연결한다. 두 번째는 인터페이스를 원으로 표기하고 인터페이스의 이름을 명시한다. 그리고 인터페이스와 클래스 사이의 관계는 실선으로 연결한다.

#### 의존(Dependency)

의존 관계는 클래스 다이어그램에서 일반적으로 가장 많이 사용되는 관계로 어떤 클래스가 다른 클래스를 참조하는 것을 말한다.

<br>

```java
public class X{
    public void function1(Y y){
        y.foo();
    }
    
    public void function2(){
        Y y = new Y();
        y.foo()
    }
}
```
![image](https://user-images.githubusercontent.com/13410737/180139841-4fef2a49-9f70-424f-a472-f21d4632f751.png){: .align-center}
<br>
클래스의 메서드에 매개변수로 들어오거나 **로컬 변수**로 특정 클래스를 참조할 경우 의존 관계라 한다. 추가로 스테레오 타입으로 점선 화살표 위에 어떤 목적의 의존인지 의미를 명확히 표시할 수도 있다.
<br>

#### 연관, 방향성 있는 연관(Association, Directed Association)
연관 관계는 어떤 클래스가 다른 클래스를 필드로  가진다는 의미이다. (C++에서는 포인터 및 참조를 필드로 가질 경우 연관 관계라 한다.) 
```java
public User{
    private List<Address> addresses;
}
```
![image](https://user-images.githubusercontent.com/13410737/180143601-7d4b6e5d-9ceb-4bb5-bdf7-de789fcee8a2.png){: .align-center}
<br>
첫 번째 다이어그래은 일반적인 연관 관계로 단지 실선 하나로  클래스를 연결하여 표기하고 두 번째 다이어그램은 방향성 있는 연관 관계로 실선 끝에 화살표를 추가했다. 연관 관계와 방향성 있는 연관 관계의 차이는 표살표가 의미 하는 방향성(navigability)인데 이것에 따라 참조하는 쪽과 당하는 쪽을 구분한다.
화살표 옆에 있는 `-addresses` 는 역할명을 나타내는 것인데 `Address`가 `User`클래스에서 참조 될때 어떤 역할을 가지고 있는지를 의미한다.
`*`은 개수(Muliplicity)을 나타내는데 대상 클래스의 가질 수 있는 인스턴스 개수 범위를 의미한다. `0...1`과 같이 점으로 구분하여 앞에 값는 최소 값, 뒤에 값은 최대 값을 의미하게 표기할 수도 있다. `*`은 `0...*`과 같은 의미로 객체가 없을 수도 있고 또는 수가 정해지지 않은 여러 개일 수도 있다는 것을 의미한다.
세 번째 다이어그램은 두 번째의 다이어그램과 비슷한 의미를 가지고 있지만 다른 형태인 속성 표기법으로 나타낸 것이다. 속성 표기법에서는 여러 개의 객체데 대한 컨테이너(Container)가 List라는 것 까지 알려 줄 수 있다.
속성 표기법으로 표현하지 않고 컨테이너 정보까지 표기하려면 아래와 같이 그릴 수 있다.
<br>
![image](https://user-images.githubusercontent.com/13410737/180144528-74cd5ae6-0e72-45e4-85eb-0bacdc65b068.png){: .align-center}
<br>

#### 집합(Aggregation)
집합 관계는 합성(Composition) 관계와 함께 연관(Association) 관계를 조금 더 특수하게 나타낸 것으로 전체(whole)와 부분(part)의 관계를 나타낸다. 연관 관계는 집합이라는 의미를 내포하고 있지 않지만 집합 관계는 집합이라는 의미를 가지고 있다.
<br>
```java
public User{
    private List<Address> addresses;
}
```
![image](https://user-images.githubusercontent.com/13410737/180146002-10c10aca-fa28-4419-8a99-b6d2b1026911.png){: .align-center}
<br>
표기법은 위와 같이 whole과 part를 실선으로 연결 후 whole 쪽에 비어있는 다이아몬드를 표기한다. 다이아몬드가 이미 방향을 표현하고 있기 때문에 화살표는 명시해도 되고 명시하지 않아도 된다. 
<br>
> 코드를 보면 연관 관계와 똑같은걸 알 수 있는데 집합 관계와 연관 관계는 집합이라는 개념적인 차이는 있지만 코드에서 이 차이를 구분하기는 힘들다. UML은 집합이라는 개념 외에 명확한 연관 관계의 정의를 제공하지 않았다 그래서 여러 프로그래머나 분석가, 설계사가 연관 관계에 대해 자기 나름대로 정의를 내리는 혼란이 생겨 집합 관계는 되도록이면 사용하지 않는 것이 좋다고 한다.
<br>

#### 합성(Composition)
합성 관계도 집합 관계와 비슷하게 전체( whole)와 부분(part)의 집합 관계를 나타내지만 개념적으로 집합 관계보다 더 강한 집합을 의미한다.

![image](https://user-images.githubusercontent.com/13410737/180147663-91093301-234d-48be-986c-de67b230ba2f.png){: .align-center}

표기법은 집합 관계에서 다이아몬드 내부가 채워져 있다는 점만 다르다. 합성 관계는 집합 관계보다 강한 집합이라고 했는데, 여기서 강한 집합이란 부분 객체가 전체 객체에 종속적이어서 부분 객체가 전체 객체의 소유 됨을 의미한다. 집합과의 다른 점은 아래와 같다.

- 부분 객체를 가지는 전체 객체가 부분 객체의 인스턴스의 전체 수명을 책임진다.
  - 전체 객체의 인스턴스가 부분 객체의 인스턴스를 생성
  - 전체 객체의 인스턴스가 소멸되면 부분 객체의 인스턴스로 함께 소멸
  - 전체 객체의 인스턴스가 복사되면 부분 객체의 인스턴스로 함께 복사
- 부분 객체에 해당하는 인스턴스는 공유 될 수 없다.
  - 전체 객체의 인스턴스가 복사 되면서 포함하고 있던 부분 객체의 인스턴스가 얕은 복사가 될 경우

