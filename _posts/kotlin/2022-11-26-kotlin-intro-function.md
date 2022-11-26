---
title : "[Kotlin] 소개 - Functions"
categories:
- Kotlin
tag :
- [Kotlin, Programming]
toc: true
toc_sticky : true
published : true
date : 2022-11-26 01:00:00
last_modified_at : 2022-11-26 01:00:00


---

## [Kotlin] 소개 - 함수(Functions)



## 함수(Functions)

### 디폴트 파라메터와 네임드 아규먼트

```kotlin
fun printMessage(message: String): Unit {                               //1
    println(message);
}

fun printMessageWithPrefix(message: String, prefix: String = "Info"){   //2
    println("[$prefix] $message");
}

fun sum(x: Int, y: Int): Int {                                          //3
    return x + y;
}

fun multiply(x: Int, y: Int) = x * y                                    //4

fun main() {
    printMessage("Hello~")                                              //5
    printMessageWithPrefix("Hello", "Log")                              //6
    printMessageWithPrefix("Hello")                                     //7
    printMessageWithPrefix(prefix = "Log", message = "Hello")           //8
    println(sum(1, 2))                                                  //9
    println(multiply(2, 4));                                            //10
}
```

```
🖥️출력
Hello~
[Log] Hello
[Info] Hello
[Log] Hello
3
8
```

1. `String`을 매개변수로 받고 `Uint`를 반환하는 간단한 함수이다.
2. 기본값으로 `Info`로 하는 디폴트 매개변수를 사용하는 함수이다. 반환 타입은 생략되어 실제로는 `Uint`를 의미한다.
3. 정수를 반환하는 함수이다.
4. 정수를 반환하는 단일 표현식 함수이다.
5. 인자로 `Hello` 를 넘겨 첫 번째 함수를 호출한다.
6. 두 개의 매개변수를 사용하여 함수를 호출하고 두 매개변수 모두에 값을 전달한다.
7. 두번 째 매개변수를 생략하여 동일한 함수를 호출한다. 기본 값은 `Info`가 사용된다.
8. 이름있는(명명된) 인자(Named Argument)를 사용하여 동일한 함수를 호출하고 인자 순서를 변경한다.
9. `sum` 함수를 호출한 결과를 출력한다.
10. `multiply` 함수를 호출한 결과를 출력한다.



### 중위 함수(Infix Functions)

단일 매개변수를 가지는 멤버 함수와 확장은 중간 삽입 함수(중위 함수)로 전환될 수 있다.

```kotlin
fun main() {
    infix fun Int.times(str: String) = str.repeat(this)        //1
    println(2 times "Bye ")                                    //2
    
    val pair = "Ferrari" to "Katrina"                          //3
    println(pair)
    
    infix fun String.onto(other: String) = Pair(this, other)   //4
    val myPair = "McLaren" onto "Lucas"
    println(myPair)
    
    val sophia = Person("Sophia")
    val claudia = Person("Claudia")
    sophia likes claudia                                       //5
}

class Person(val name: String){
    val likedPeople = mutableListOf<Person>()
    infix fun likes(other: Person) { likedPeople.add(other) }  //6
}
```

```
🖥️ 출력
Bye Bye 
(Ferrari, Katrina)
(McLaren, Lucas)
```

1. `Int` 에 중위 확장 함수을 정의한다.
2. 중위 함수를 호출한다.
3. 표준 라이브러리의 `to` 중위 함수를 호출하여 `Pair` 를 생성한다.
4. 독창적으로 `to` 를 `onto` 로 구현한다.
5. 중위 표현은 멤버 함수(메서드)에서도 동작한다.
6. 클래스가 첫 번째 매개변수가 됩니다.



### 연산자 함수(Operator Functions)

함수는 연산자로 "업그레이드"되어 해당 연산자 기호로 호출할 수 있다.

```kotlin
fun main() {
    operator fun Int.times(str: String) = str.repeat(this)                 //1
    println(2 * "Bye")                                                     //2

    operator fun String.get(range: IntRange) = substring(range)            //3
    val str = "Always forgive your enemies; nothing annoys them so much"
    println(str[0..14])                                                    //4
}
```

```
🖥️ 출력
Bye Bye
Always forgive 
```

1. `operator` 키워드를 사용하여 위의 중위 함수를 한 단계 더 수행합니다.
2. `times()`의 연산자 기호는 `*`이므로 `2 * "Bye"`를 사용하여 함수를 호출할 수 있다.
3. 연산자 함수를 사용하면 문자열에 대한 쉬운 범위 접근이 가능하다.
4. `get()` 연산자는 인덱스 접근 구문을 활성화한다.



### 가변 인자(Variable number of arguments) 매개변수

`Varargs` 를 사용하면 쉼표로 구분하여 여러개의 인자를 전달할 수 있다.

```kotlin
fun main() {    
    fun printAll(vararg messages: String){
        for (m in messages) println(m)
    }
    printAll("Hello", "Hallo", "Salut", "Hola", "안녕")

    fun printAllWithPrefix(vararg messages: String, prefix: String){
        for (m in messages) println(prefix + m)
    }
    printAllWithPrefix(
        "Hello", "Hallo", "Salut", "Hola", "안녕",
        prefix = "Greeting: "
    )

    fun log(vararg entries: String){
        printAll(*entries)
    }
}
```

```
🖥️ 출력
Hello
Hallo
Salut
Hola
안녕
Greeting: Hello
Greeting: Hallo
Greeting: Salut
Greeting: Hola
Greeting: 안녕
```

1. `vararg` 키워드는 매개변수를 가변 인자로 바꾼다.
2. 임의 개수의 문자열 인수로 printAll을 호출할 수 있다.
3. 명명된 매개변수 덕분에 가변 인자 뒤에 동일한 유형의 다른 매개변수를 추가할 수도 있다. 값을 전달할 방법이 없기 때문에 Java에서는 허용되지 않는다.
4. 명명된 매개 변수를 사용하여 가변 인자와 별도로 `prefix`에 값을 설정할 수 있다.
5. 런타임 시 가변인자는 배열일 뿐이다. 가변 인자들을 매개변수로 전달하려면 (`Array<String>`) 대신 `*entries`(`String`의 vararg)을 전달할 수 있는 특수 연산자 `*`를 사용한다.
