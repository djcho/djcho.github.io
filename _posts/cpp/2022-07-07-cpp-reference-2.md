---
title : "[C++] 참조(Reference) 변수 #2"
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

# [C++] 참조(Reference) 변수 #2

## 함수 매개변수로서의 참조

참조는 주로 함수의 매개변수로 사용된다. 그것은 어떤 함수에서 사용하는 변수의 이름을 그 함수를 호출한 프로그램(호출 함수)에 있는 어떤 변수의 대용 이름으로 만든다. 이러한 방식으로 매개변수를 전달하는 것을 참조로 전달이라 한다.

![image](https://user-images.githubusercontent.com/13410737/177801552-4ff5950e-c7af-41a0-bede-c31a8cb6d947.png)

![image](https://user-images.githubusercontent.com/13410737/177801646-329372de-e05e-4ae2-b61a-e8723bd90a67.png)

이 그림을 보며 예제 코드를 분석해보자.



## 예제

```c++
#include <iostream>
 
using namespace std;
 
void swapr(int & a, int & b);
void swapp(int *p, int *q);
void swapv(int a, int b);
 
void main()
{
    int wallet1 = 3000;
    int wallet2 = 3500;
 
    cout << "지갑 1 = " << wallet1 << "원";
    cout << ", 지갑2 = " << wallet2 << "원\n";
    
    cout << "참조를 이용하여 내용들을 교환: \n";
    swapr(wallet1, wallet2); //변수를 전달
    cout << "지갑 1 = " << wallet1 << "원";
    cout << ", 지갑 2 = " << wallet2 << "원\n";
    
    cout << "주소를 이용하여 내용들을 다시 교환:\n";
    swapp(&wallet1, &wallet2); //변수의 주소를 전달
    cout << "지갑 1 = " << wallet1 << "원";
    cout << ", 지갑 2 = " << wallet2 << "원\n";
    
    cout << "값으로 전달하여 내용 교환 시도:\n";
    swapv(wallet1, wallet2);
    cout << "지갑 1 = " << wallet1 << "원";
    cout << ", 지갑 2 = " << wallet2 << "원\n";
}
 
void swapr(int & a, int & b) //참조를 이용하여 교환
{
    int temp = 0;
 
    temp = a; //변수의 값으로 a, b를 사용
    a = b;
    b = temp;
}
 
void swapp(int *p, int *q) //포인터를 이용하여 교환
{
    int temp = 0;
    
    temp = *p; //변수의 값으로 *p, *q를 사용
    *p = *q;
    *q = temp;
}
 
void swapv(int a, int b) //값으로 전달하여 교환 시도
{
    int temp = 0;
    
    temp = a; //변수의 값으로 a, b를 사용
    a = b;
    b = temp;
}
```

```
💻출력💻
지갑 1 = 3000원, 지갑2 = 3500원
참조를 이용하여 내용들을 교환:
지갑 1 = 3500원, 지갑2 = 3000원
주소를 이용하여 내용들을 다시 교환:
지갑 1 = 3000원, 지갑2 = 3500원
값으로 전달하여 내용 교환 시도:
지갑 1 = 3000원, 지갑2 = 3500원
계속하려면 아무 키나 누으십시오...
```

swapr()에 있는 변수 a, b는 각각 wallet1, wallet2의 대용 이름으로 작동하기 때문에, a와 b를 교환하면 wallet1, wallet2가 교환된다는 것이다. 그러나 swapv()에 있는 변수 a, b는 단순히 wallet1, wallet2를 각각 복사한 값이므로, a와 b를 교환해도 원본인 wallet1과 wallet2는 영향을 받지 않는다.

