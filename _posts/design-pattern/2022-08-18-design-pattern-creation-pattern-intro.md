---
title : "[C++][DesignPattern] 생성 패턴 - 생성 패턴이란?"
categories:
- GofDesignPattern
tag :
- [C++, GoF, DesignPattern, CreationalPattern]
toc: true
toc_sticky : true
published : true
date : 2022-08-18
last_modified_at : 2022-08-18
---









에릭 감마, 리처드 헬름, 랄프 존슨, 존 블라시디스 지음, [GoF의 디자인패턴::재사용성을 지닌 객체지향 소프트웨어의 핵심요소] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





# 생성패턴이란?

생성 패턴(creation pattern)은 인스턴스를 만드는 절차를 추상화하는 패턴이다. 이 범주에 해당하는 패턴은 객체를 생성, 합성하는 방법이나 객체의 표현 방법과 시스템을 분리해 준다. 

<br>

**생성 패턴의 핵심**

- 생성 패턴을 이용하면 **<u>무엇</u>**이 생성되고, **<u>누가</u>** 이것을 생성하며, 이것이 **<u>어떻게</u>** 생성되는지, **<u>언제</u>** 생성할 것인지 결정하는 데 유연성을 확보할 수 있게 된다.

<br>

생성 패턴으로 분류되는 패턴은 여러개인데, 이런 여러 생성 패턴들은 서로 보완적일 수도 있고, 선택되기 위해 서로 경쟁적일 수도 있다. 즉, <u>동일한 문제 해결을 위해서 어떤 패턴을 사용해야 할지 결정을 내리기 어렵다.</u>

<br>

그럼 게임에 들어갈 미로를 개발한다고 가정을하고 일반적으로 설계할 수 있는 방법을 통해 어떤 문제가 있고 왜 이 패턴을 써야하는지 알아보자.

미로를 개발하기 위해서는  `Room`, `Door`, `Wall` 클래스가 필요로할 것이며 아래와 같은 클래스 구조를 가질것이다.

<br>

![image](https://user-images.githubusercontent.com/13410737/185420714-28d7ca08-7608-4294-9c87-430f172d1c02.png){: .align-center}

<br>

`SiteMap` 클래스는 미로의 구성 요소들에 필요한 모든 연산을 정의한 공통 추상 클래스이다. 

```c++
class MapSite {
public:
    virtual void Enter() = 0; // 구현부를 갖지 않는 순수 가상 메서드
};
```

<br>

`MapSite`에 정의된 `Enter()` 는 좀더 섬세한 게임 동작을 만드는데 쓸 수 있는 기본 연산이다. 서브클래스가 어떤 것이냐에 따라 `Enter()` 는 위치를 변경하도록 수현할 수도 있고, 이동을 못하고 상처를 입도록 구현할 수도 있다.

`Room` 클래스는 `MapSite`를 상속받은 구체적인 클래스로 미로에 있는 다른 요소와 관련성이 갖도록 정의한다. `Room` 클래스는 방 번호를 저장하는데, 이 번호로 미로에 있는 방을 식별할 수 있다.

<br>

```c++
class Room : public MapSite {
public:
    Room(int RoomNo);
    MapSite* GetSide(Direction) const;
    void SetSide(Direction, MapSite*);
    
    virtual void Enter();
    
private:
    MapSite* _side[4]; // 방은 네개의 방향을 갖고 있고 각 방향에는 MapSite 의 서브클래스 인스턴스가 올 수 있다.
    int _roomNumber;
};
```

<br>

다음 클래스는 방의 각 측면(side)에 있을 수 있는 문과 벽을 보여준다.

<br>

```c++
class Wall : public MapSite {
public:
    Wall();
    
    virtual void Enter();
};
```

```c++
class Door : public MapSite {
public:
    Door(Room* room = 0, Room* = 0);
    // 문을 초기화하기 위해서는 문이 어드 방 사이에 있는지 알아야 한다.
    
    virtual void Enter();
    Room* OtherSideFrom(Room*);
    
private:
    Room* _room1;
    Room* _room2;
    bool _isOpen;
};
```

<br>

또한 방들의 집합을 표현하기 위해 클래스 `Maze`를 정의하겠다. `Maze`는 `RoomNo()` 를 호출하여 방 번호로 특정 방을 찾을 수도 있다.

<br>

```c++
class Maze {
public:
    Maze();
    
    void Addroom(Room*);
    Room* RoomNo(int) const;
private:
    // ...
};
```

<br>

다른 필요한 클래스로는 `MazeGame` 이 있다. 실제로 미로를 생성하는 클래스이다. 미로를 생성하는 가장 간단한 일반적인 방법은 빈 미로에 미로의 구성요소(벽, 방, 문 등)들을 추가하고 각 요소끼리 연결하는 것이다. 예를 들면 아래와 같다.
<br>

```c++
Maze* MazeGame::CreateMaze() {
    Maze* aMaze = new Maze;
    Room* r1 = new Room(1);
    Room* r2 = new Room(2);
    Door* theDoor = new Door(r1, r2);
    
    aMaze->AddRoom(r1);
    aMaze->AddRoom(r2);
    
    r1->SetSide(North, new Wall);
    r1->SetSide(East, theDoor);
    r1->SetSide(South, new Wall);
    r1->SetSide(West, new Wall);
        
    r2->SetSide(North, new Wall);
    r2->SetSide(East, new Wall);
    r2->SetSide(South, new Wall);
    r2->SetSide(West, theDoor);
    
    return aMaze;
}
```

<br>

허나, 위 함수는 겨우 방 두 개를 만든 것뿐인데 매우 비효율적으로 보인다. 좀 더 효율적이게 만드려면 `Room`클래스의 생성자에서 모든 방향의 면(side)들을 벽(wall)로 초기화해 두었다면 방의 모든 면을 세팅하는 코드는 없앨 수 있다.

<br>

```c++
    r1->SetSide(North, new Wall);
    r1->SetSide(East, theDoor);
    r1->SetSide(South, new Wall);
    r1->SetSide(West, new Wall);
        
    r2->SetSide(North, new Wall);
    r2->SetSide(East, new Wall);
    r2->SetSide(South, new Wall);
    r2->SetSide(West, theDoor);
```

<br>

즉, 위와 같은 코드는 없앨 수 있다. 다시 말해, `Room` 클래스의 생성자에서 이런 일은 처리하고 단지 다음과 같이 변경되는 부분만 코드화하면 된다.

<br>

```c++
    r1->SetSide(East, theDoor);
    r2->SetSide(West, theDoor);
```

<br>

그러나 앞의 `CreateMaze()` 코드의 진짜 문제는 방의 레이아웃을 하드코딩하고 있기 때문에 코드의 **<u>유연성이 떨어진다</u>**는 것이다.  즉, 레이아웃을 바꾸고 싶으면 멤버 함수를 바꾸는 수밖에 없다. 이 함수를 전체를 재구현하여 오버라이드하던지 함수의 일부를 바꾸던지의 방법 밖에 없다.

생성 패터는 이런 상황에서 어떻게 **<u>유연한</u>** 설계를 할 수 있는지에 대한 해법을 제공한다. 특히 미로에 구성요소를 정의하는 클래스를 쉽게 변경할 수 있도록 방법을 제공한다.

예를 들어 기존 미로가 갖고 있는 레이아웃을 재사용하면서 새로운 타입의 미로를 만들고 싶다고 가정하면, 단어를 맞춰야 문이 열리는 `DoorNeedingSpell` 이라던지..?

어떻게 하면  `CreateMaze()` 함수를 쉽게 바꿔 이런 것들이 달린 미로를 만들 수 있을까? 생성 패턴은 이런 어려움을 해소할 수 있도록 여러 가지 방법을 제공한다.

- `CreateMaze`가 방, 벽, 문을 생성하기 위해서 생성자를 이용하지 않고 가상 함수를 호출하도록 구현되어 있다면, 이 가상 함수의 실제 구현을 다양한 방법으로 변경할 수 있다. (팩토리 메서드 패턴)
- `CreateMaze`가 방,벽, 문을 생성하기 위해 생성 방법을 알고 있는 객체를 매개변수로 넘겨받을 수 있다면, 생성 방법이 바뀔 때마다 새로운 매개변수를 넘겨받음으로써 생성할 객체의 유형을 달리할 수 있다. (추상 팩토리 패턴)
- `CreateMaze`가 생성하고자 하는 미로에 방, 문, 벽을 추가하는 연산을 사용해서 새로운 미로를 만들 수 있는 객체를 넘겨받는다면 미로를 만드는 방법이나 변경을 이 객체의 상속을 통해 해결할 수 있다. (빌더 패턴)
- `CreateMaze`를 이미 만든 다양한 방, 문, 벽 객체로 매개변수화 하는 방법도 가능한데, 이미 만든 객체를 복사해서 미로에 추가하면, 이 인스턴스를 교체하여 미로의 모습을 다양하게 변경할 수 있다. (원형 패턴)

<br>

위 다섯개 생성 패턴 중에서 쓰지 않은 단일체 패턴은 한 게임에 오로지 하나의 미로 객체만 존재할 수 있고 그 게임의 모든 게임 객체들이 이 미로에 접근이 가능하도록 보장한다. 전역 변수나 전역 함수에 의존할 필요도 없으며 기존 코드를 건드리지 않고도 미로를 쉽게 대체하거나 확장할 수 있도록 해준다.



