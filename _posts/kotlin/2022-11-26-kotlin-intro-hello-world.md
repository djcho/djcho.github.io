---
title : "[Kotlin] 소개 - Hello world"
categories:
- Kotlin
tag :
- [Kotlin, Programming]
toc: true
toc_sticky : true
published : true
date : 2022-11-26 00:00:00
last_modified_at : 2022-11-26 00:00:00


---

## [Kotlin] 소개 - Hello world



## Hello World

```kotlin
package djcho.kotlin.study          //1  

fun main() {                        //2
    println("Hello World!")         //3
}
```

```
🖥️출력
Hello World!
```



1. 패키지
    - Kotlin 코드는 일반적으로 패키지에 정의 된다.
    - 패키지 명시는 선택 사항이며 소스 파일에 패키지를 지정하지 않으면 콘텐츠가 기본 패키지로 이동한다.
2. `main` 함수
    - Kotlin 어플리케이션의 진입점은 `main` 함수이다.
    - `main` 함수는 리턴 타입을 명시하지 않는다. 즉, 아무것도 반환하지 않는다.
    - Kotlin 1.3부터는 `main` 함수의 파라메터를 생략할 수 있다. 
3. `println` 함수
    - 표준 아웃풋(standard output)에 라인을 쓴다. 
    - 암시적으로 가져온다.
    - 세미 콜론(;) 명시는 선택 사항이다.

Kotlin 1.3 이전의 버전에서는 `main` 함수에 `Array<String>` 매개변수를 명시했어야 한다. 

```kotlin
fun main(args:Array<String>){
    println("Hello, World")
}
```

