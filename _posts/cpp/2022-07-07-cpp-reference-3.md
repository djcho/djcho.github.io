---
title : "[C++] 참조(Reference) 변수 #3"
categories:
- Cpp
tag :
- [Cpp, Programming]
toc: true
toc_sticky : true
published : true
date : 2022-07-07
last_modified_at : 2022-07-07


---

## [C++] 참조(Reference) 변수 #3

### 구조체에 대한 참조

구조체에 대한 참조를 사용하는 방법은, 기본 데이터형의 변수에 대한 참조를 선언할 때와 마찬가지로, 구조체 매개변수를 선언할 때 참조 연산자 &를 앞에 붙이면 된다.

```cpp
#include <iostream>
 
using namespace std;
 
struct sysop
{
    char name[26];
    char quote[64];
    int used;
};
 
const sysop & use(sysop & sysopref); //참조를 리턴하는 함수
 
void main()
{
    sysop looper = {"Rick \"Fortran\" Looper", "I'm a goto kind of guy.", 0};
    use(looper); //looper는 sysop형이다.
    cout<< "Looper: " << looper.used << "번 사용" << endl;
    sysop copycat;
    copycat = use(looper);
    cout << "Looper: " << looper.used << "번 사용" << endl;
    cout << "Copycat: " << copycat.used << "번 사용" <<  endl;
    cout << "use(looper): " << use(looper).used << "번 사용" << endl;
}
 
//use()는 전달받은 참조를 다시 리턴한다.
const sysop & use(sysop & sysopref)
{
    cout << sysopref.name << " says:" << endl;
    cout << sysopref.quote << endl;
    sysopref.used++;
    return sysopref;
}
```

```
💻출력💻
Rick "Fortran" Looper says:
I'm a goto kind of guy.
Looper: 1번 사용
Rick "Fortran" Looper says:
I'm a goto kind of guy.
Looper: 2번 사용
Copycat: 2번 사용
Rick "Fortran" Looper says:
I'm a goto kind of guy.
use(looper): 3번 사용
계속하려면 아무 키나 누으십시오...
```

use(looper); 이 함수 호출 명령문은 looper 구조체를 use()함수에 참조로 전달한다. 이 함수는 sysopref를 looper의 대용 이름으로 사용한다. 그래서 use() 함수가 sysopref의 name 멤버와 quote 멤버를 출력하면 실제로는 looper의 name 멤버와 quote멤버가 출력된다. 또한 use() 함수가 sysopref.used르 1만큼 증가 시키면 looper.used가 1만큼 증가한다.

 또한, 일반적으로 함수의 리턴 값은 임시 저장 영역에 복사되고, 호출 프로그램이 그것을 사용한다. 그러나 참조를 리턴하면 호출 프로그램은 복사본이 아니라 리턴값을 직접 사용한다. 리턴되는 참조는 처음에 함수에 전달했던 바로 그 참조를 말하므로, 호출 프로그램은 실제로는 자기 자신의 변수를 직접 사용하게 된다. 예를 들면, sysopref는 looper에 대한 참조이므로, 리턴값은 main()에 있는 looper변수 자체가 된다. 다음과 같은 코드를 고려해 보자

- `copycat = use(looper);`

use()함수가 단순히 구조체를 리턴한다면 , sysopref의 내용이 임시 리턴 위치에 복사될 것이다. 그러고 나서 그 임시 리턴 위치의 내용이 copycat으로 복사될 것이다. use()가 looper에 대한 참조를 리턴하기 때문에, 이 경우에는 looper 의 내용이 copycat으로 직접 복사된다. 이것이, <u>구조체를 리턴하는 대신에 구조체에 대한 참조를 리턴함으로써 얻을 수 있는 주요 이점 중의 하나인 효율성을 설명한다.</u>

**<u>참조를 리턴하는 함수는 실제로는 참조된 변수의 대용 이름이다</u>**