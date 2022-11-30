---
title : "[Kotlin] 소개 - 제네릭(Generics)"
categories:
- Kotlin
tag :
- [Kotlin, Programming]
toc: true
toc_sticky : true
published : true
date : 2022-11-30 01:00:00
last_modified_at : 2022-11-30 01:00:00


---

## 제네릭(Generics)

제네릭은 현대 언어들에서 표준이 된 일반성 메커니즘이다. 제네릭 클래스와 함수는 `T` 와 무관한 `List<T>` 내의 로직과 같이 독립적인 공통 논리를 캡슐화하여 코드 재사용성을 높인다.

## 제네릭 클래스(Generic Classes)

Kotlin에서 제네릭을 사용하는 첫 번째 방법은 제네릭 클래스를 만드는 것이다.

```kotlin
class MutableStack<E>(vararg items: E){                      //1
    private val elements = items.toMutableList()

    fun push(element: E) = elements.add(element)             //2

    fun peek(): E = elements.last()                          //3

    fun pop(): E = elements.removeAt(elements.size - 1)

    fun isEmpty() = elements.isEmpty()

    fun size() = elements.size

    override fun toString() = "MutableStack(${elements.joinToString()})"
}

fun main(args: Array<String>) {
    val stack = MutableStack(0.62, 3.14, 2.7)
    stack.push(9.87)
    println(stack)

    println("peek(): ${stack.peek()}")
    println(stack)

    for(i in 1..stack.size()){
        println("pop(): ${stack.pop()}")
        println(stack)
    }
}
```

```
🖥️ 출력
MutableStack(0.62, 3.14, 2.7, 9.87)
peek(): 9.87
MutableStack(0.62, 3.14, 2.7, 9.87)
pop(): 9.87
MutableStack(0.62, 3.14, 2.7)
pop(): 2.7
MutableStack(0.62, 3.14)
pop(): 3.14
MutableStack(0.62)
pop(): 0.62
MutableStack()
```

1. 제네 클래스 `MutableStack<E>`를 정의한다. 여기서 `E`는 제네릭 타입 매개변수라 한다. 호출 부분에서 `MutableStack<Int>`로 선언하여 `Int`와 같은 특정 유형에 할당한다.
2. 제네릭 클래스 내에서 `E`는 다른 타입과 마찬가지로 매개변수로 사용할 수 있다.
3. `E`를 반환 타입으로도 사용할 수 있다.



## 제네릭 함수(Generic Funtions)

로직이 특정 타입과 독립적인 경우 제네릭 함수를 생성할 수도 있다. 예를 들어 가변 스택을 생성하는 유틸 함수를 작성할 수 있다.

```kotlin
fun<E> mutableStackOf(vararg elements: E) = MutableStack(*elements)

fun main(args: Array<String>) {
    val stack = mutableStackOf(0.62, 3.14, 2.7)
}
```

```
🖥️ 출력
MutableStack(0.62, 3.14, 2.7)
```

컴파일러는 `muatableStackOf`의 매개변수에서 제네릭 유형을 유추할 수 있으므로 `mutableStackOf<Double>(...)`를 작성할 필요가 없다.
