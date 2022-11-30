---
title : "[Kotlin] 소개 - 변수(Variables)"
categories:
- Kotlin
tag :
- [Kotlin, Programming]
toc: true
toc_sticky : true
published : true
date : 2022-11-28 00:00:00
last_modified_at : 2022-11-28 00:00:00


---

## 변수(Variables)

Kotlin은 강력한 타입 추론을 가지고 있다. 변수의 타입을 명시적으로 선언할 수 있지만 일반적으로는 컴파일러가 유추하여 작업을 수행하도록 한다. Kotlin은 불변성을 강제하지는 않지만 본질적으로 `var`보다 `val`를 사용을 권장한다.

```kotlin
fun main() {
    var a: String = "initial";    //1
    println(a)
    val b: Int = 1                //2
    val c = 3;                    //3
}
```

```
🖥️출력
initial
```

1. 변경 가능한 변수를 선언하고 초기화한다.
1. 변경할 수 없는 변수를 선언하고 초기화 한다.
1. 변경할 수 없는 변수를 타입 지정 없이 선언하고 초기화한다. 컴파일러는 `Int`타입으로 유추한다.

```kotlin
fun main() {
    var e: Int  //1
    println(e)  //2
}
```

```
🖥️출력
Kotlin: Variable 'e' must be initialized
```



1. 초기화 없이 변수를 선언한다.
2. 변수를 사용하려 시도했기 때문에 컴파일러는 `Kotlin: Variable 'e' must be initialized` 에러를 출력한다.



변수 초기화를 할 때에는 자유롭게 선택할 수 있지만 처음 읽기 전에 초기화를 해야 한다.

```kotlin
fun someCondition() = true

fun main() {
    val d: Int             //1
    if(someCondition()){
        d = 1              //2
    }else{
        d = 2              //2
    }

    println(d)             //3
}
```

```
🖥️ 출력
1
```

1. 초기화하지 않고 변수를 선언한다.
1. 어떤 조건에 따라 다른 값으로 변수를 초기화한다.
1. 변수는 이미 초기화 되었기 때문에 읽을 수 있다.
