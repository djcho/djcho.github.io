---
title : "[Kotlin] 소개 - Null 안전(Null Safety)"
categories:
- Kotlin
tag :
- [Kotlin, Programming]
toc: true
toc_sticky : true
published : true
date : 2022-11-28 01:00:00
last_modified_at : 2022-11-28 01:00:00


---

## [Kotlin] 소개 - Null 안전(Null Safety)



세상에서 `NullPointerException`을 제거하려는 노력의 일환으로 Kotlin의 변수 유형은 `null` 할당을 허용하지 않는다. `null`이 될 수 있는 변수가 필요한 경우 변수 타입 끝에 `?`를 추가하여 nullable 로 선언이 가능하다. 

```kotlin
fun main() {
    var neverNull: String = "This can't be null"            //1

    neverNull = null                                        //2

    var nullable: String? = "You can keep a null here"      //3

    nullable = null                                         //4

    var inferredNonNull = "The compiler assumes non-null"   //5

    inferredNonNull = null                                  //6

    fun strLength(notNull: String): Int {                   //7
        return notNull.length
    }

    strLength(neverNull)                                    //8
    strLength(nullable)                                     //9
}
```

```
🖥️출력
Kotlin: Null can not be a value of a non-null type String
Kotlin: Null can not be a value of a non-null type String
Kotlin: Type mismatch: inferred type is Nothing? but String was expected
```

1. `null`이 아닌 문자열 변수를 선언한다.
2. `null`을 허용하지 않는 변수에 `null`을 할당하면 컴파일 오류가 발생한다.
3. `null`이 될 수 있는 문자열 변수를 선언한다.
4. `null`이 될 수 있는 변수에 `null`을 할당할 수 있다.
5. 컴파일러는 타입을 유추할 때 값으로 초기화된 변수에 대해 `null`이 아닌 타입으로 가정한다.
6. 유추된 타입 변수에 `null`을 할당하려고 하면 컴파일 오류가 발생한다.
7. `null`이 아닌 문자열 매개변수로 함수를 선언한다.
8. `null`이 아닌 문자열을 인자로 함수 호출이 가능하다.
9. `String?`으로 함수를 호출할 때 nullable 인수를 사용하면 컴파일 오류가 발생한다.
