---
title : "[Kotlin] 소개 - 클래스(Classes)"
categories:
- Kotlin
tag :
- [Kotlin, Programming]
toc: true
toc_sticky : true
published : true
date : 2022-11-30 00:00:00
last_modified_at : 2022-11-30 00:00:00


---

## 클래스(Classes)

클래스 선언은 클래스 이름, 클래스 헤더(타입, 매개변수 지정, 기본 생성자 등) 및 중괄호로 묶인 클래스 본문으로 구성된다. 헤더와 본문은 모두 생략 가능하다. 클래스에 본문이 없다면 중괄호 또한 생략할 수 있다.

```kotlin
class Customer                                            //1

class Contact(val id: Int, var email: String)             //2

fun main() {

    val customer = Customer()                             //3

    val contact = Contact(1, "eowns36@gmail.com")         //4

    println(contact.id)                                   //5
    contact.email = "djcho@gmail.com"                     //6
    println(contact.email)
}
```

```
🖥️ 출력
1
djcho@gmail.com
```



1. 속성이나 사용자 정의 생성자 없이 `Customer`라는 클래스를 선언한다. 매개변수가 존재하지 않는 기본 생성자는 Kotlin 이 자동으로 생성해 준다.
2. 변경할 수 없는 `id`와 변경할 수 있는 `email` 이라는 두 가지 속성이 존재하는 클래스와 두 개의 매개변수 `id`와 `email`이 있는 생성자를 선언한다.
3. 기본 생성자를 통해 `Customer` 클래스의 인스턴스를 생성한다. Kotlin 에는 `new`키워드는 존재하지 않는다.
4. 두 개의 인수로 되어 있는 생성자를 사용하여 `Contact` 클래스의 인스턴스를 만든다.
5. `id`속성에 접근한다.
6. `email`속성 값을 업데이트 한다.
