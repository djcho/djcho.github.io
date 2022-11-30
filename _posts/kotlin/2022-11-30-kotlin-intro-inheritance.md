---
title : "[Kotlin] 소개 - 제네릭(Generics)"
categories:
- Kotlin
tag :
- [Kotlin, Programming]
toc: true
toc_sticky : true
published : true
date : 2022-11-30 02:00:00
last_modified_at : 2022-11-30 02:00:00


---

## 상속(Inheritance)

Kotlin은 전통적인 객체 지향 상속 매커니즘을 완벽하게 지원한다.

```kotlin
open class Dog {                              //1
    open fun sayHello(){                      //2
        println("wow wow!")
    }
}

class Yorkshire: Dog() {                      //3
    override fun sayHello() {                 //4
        println("wif wif!")
    }
}

fun main() {
    val dog: Dog = Yorkshire()
    dog.sayHello()
}
```

```
🖥️ 출력
wif wif!
```



1. Kotlin 클래스는 기본적으로 final 클래스이다. 클래스 상속을 허용하려면 `open`키워드를 사용하면 된다.
2. Kotlin 메서드도 기본적으로 final 메서드이다. 클래스와 마찬가지로 `open` 키워드는 메서드를 재정의할 수 있다.
3. 이름 뒤에 {: 부모클래스이름()} 을 지정하면 클래스가 부모 클래스를 상속 받는다. 빈 괄호는 부모 클래스의 기본 생성자 호출을 의미한다.
4. 메서드 또는 특성을 재정의하려면 `override` 키워드가 필요하다.



### 매개변수가 존재하는 생성자 상속(Inheritance with Parameterized Constructor)

```kotlin
open class Tiger(val origin: String){
    fun sayHello(){
        println("A tiger from $origin says: grrhhh!")
    }
}

class KoreaTiger: Tiger("Korea")               //1

fun main() {
    val dog: Dog = Yorkshire()
    dog.sayHello()

    val tiger: Tiger = KoreaTiger()
    tiger.sayHello()
}
```

```
🖥️ 출력
A tiger from Korea says: grrhhh!
```

1. 자식 클래스를 생성할 때 부모 클래스의 매개변수가 존재하는 생성자를 사용하려면 자식 클래스 선언에 생성자 인수를 제공해야한다.



### 부모 클래스에 생성자 인수전달(Passing Constructor Arguments to Superclass)

```kotlin
open class Lion(val name: String, val origin: String) {
    fun sayHello(){
        println("$name, the lion from $origin says: graoh!")
    }
}

class Asiatic(name: String): Lion(name = name, origin = "India")   //1

fun main() {
    val lion: Lion = Asiatic("Rufo")                               //2
    lion.sayHello()
}
```

```kotlin
🖥️ 출력
Rufo, the lion from India says: graoh!
```

1. `Asiatic` 선언의 `name`은 `var`도 `val`도 아니다. 생성자 인수이며 값은 부모 클래스인 `Lion`의 `name` 속성에 전달된다.
2. 이름이 `Rufo`인 `Asiatic` 인스턴스를 생성한다. 호출은 `Rufo` 및 `India`인수를 사용하여 `Lion` 생성자를 호출한다.
