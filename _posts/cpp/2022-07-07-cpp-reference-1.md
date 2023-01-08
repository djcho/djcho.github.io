---
title : "[C++] 참조(Reference) 변수 #1"
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



# [C++] 참조(Refernce) 변수 #1

참조(reference)는 미리 정의된 어떤 변수의 실제 이름 대신 쓸 수 있는 대용 이름이다. 참조의 주된 용도는 함수의 형식 매개변수에 사용하는 것이다. 참조를 전달인자로 사용하면, 그 함수는 복사본 대신 원본 데이터를 가지고 작업한다. C와 C++는 변수의 주소를 나타내기 위해 &기호를 사용한다. C++는 & 기호에 또 하나의 의미를 추가하여 참조 선언을 나타내게 하였다. 예제를 통해 알아 보자.

## 예제 #1

```cpp
#include <iostream>

using namespace std;
void main()
{
    int rats = 101;
    int & rodents = rats; // rodents는 참조변수이다.
    
    cout << "rats = " << rats;
    cout << ", rodents = " << rodents << endl;
    rodents++;
    cout <<"rats = "<< rats;
    cout <<", rodents="<< rodents<<endl;
    
    cout <<"rats의 주소 = " << &rats;
    cout <<", rodents의 주소"<< &rodents << endl;
}
```

```
💻출력💻
rats = 101, rodents = 101
rats = 102, rodents = 102
rats의 주소 = 0031FC38, rodents의 주소0031FC38
```

다음 명령문에 사용된 & 연산자는

`int & rodents = rats;`

주소 연산자가 아니라, rodents 변수를 int &형, 즉 int 형 변수에 대한 참조로 선언한다. 그러나 다음 명령문에 사용된 & 연산자는 

`cout <<", rodents의 주소"<< &rodents << endl;`

주소 연산자이며, &rodents는 rodents가 참조하는 변수 의주소를 나타낸다.

 실행 결과를 보면, rats와 rodents는 같은 값과 같은 주소를 가지고 있다. 그리고 rodents를 1만큼 증가시키는 일이 두 변수 모두에 영향을 주었다. 더 정확하게 말하자면, rodents++는 이름이 두 개인 변수의 값을 증가시켰다.

```cpp
int rat;
int & rodent;
rodent = rat; //이렇게 할 수는 없다.
// 또한 참조를 선언할 때 참조 변수를 함께 초기화 해야 한다.
```

## 예제 #2

```cpp
#include <iostream>
 
using namespace std;
 
void main()
{
    int rats = 101;
    int & rodents = rats;    //rodents는 참조이다
    
    cout<< "rats = " << rats;
    cout << ", rodents = " << rodents << endl;
    
    cout << "rats의 주소 = " << &rats;
    cout << ", rodents의 주소 = " <<&rodents << endl;
    
    int bunnies = 50;
    rodents = bunnies;                     //참조를 바꿀수 있을까?
    cout << "bunnies = " << bunnies;
    cout << ", rats = " << rats;
    cout << ", rodents = " << rodents << endl;
    
    cout << "bunnies의 주소 = " << &bunnies;
    cout << ", rodents의 주소 = " << &rodents << endl;
}
```

```
💻출력💻
rats = 101, rodents = 101
rats의 주소 = 0031FC38, rodents의 주소0031FC38
bunnies = 50, rats = 50, rodents = 50
bunnies의 주소 = 0029FD20, rodents의 주소 = 0029FD38
```

언뜻 보기에 rodents의 값이 101에서 50으로 변했기 때문에 rodents로 하여금 bunnies를 참조하게 하려는 시도는 성공한 것 처럼 보인다. 그러나 자세히 보면 rats와 rodents가 여전히 같은 주소를 공유하고 있다. 이 주소는 bunnies의 주소와는 다르다. rodents가 rats 의 대용 이름이기 때문에, rodents = bunnies; 이 대입 명령문은 실제로는 다음 명령과 동일한 것이다.

- `rats = bunnies;`

즉, "rats 변수에 bunnies 변수의 값을 대입하라" 는 의미이다. 쉽게 말해서, 참조는 대입문이 아니라 초기화 선언에 의해서만 설정할 수 있다.