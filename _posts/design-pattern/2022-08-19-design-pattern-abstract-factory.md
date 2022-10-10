---
title : "[C++][DesignPattern] 생성 패턴 - 추상 팩토리"
categories:
- GofDesignPattern
tag :
- [C++, GoF, DesignPattern, AbstractFactory]
toc: true
toc_sticky : true
published : true
date : 2022-08-19
last_modified_at : 2022-08-19
---









에릭 감마, 리처드 헬름, 랄프 존슨, 존 블라시디스 지음, [GoF의 디자인패턴::재사용성을 지닌 객체지향 소프트웨어의 핵심요소] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





## 추상 팩토리

### 의도

상세화된 서브클래스를 정의하지 않고도 서로 관련성이 있거나 독립적인 여러 객체의 군을 생성하기 위한 인터페이스를 제공한다.



### 다른 이름

키트(Kit)



### 활용성

추상 팩토리는 다음의 경우에 사용한다.

- 객체가 생성되거나 구성/표현되는 방식과 무관하게 시스템을 독립적으로 만들고자 할 때
- 여러 제품군 중 하나를 선택해서 시스템을 설정해야 하고 한번 구성한 제품을 다른 것으로 대체할 수 있을 때
- 관련된 제품 객체들이 함께 사용되도록 설계되었고, 이 부분에 대한 제약이 외부에도 지켜지도록 하고 싶을 때
- 제품에 대한 클래스 라이브러리를 제공하고, 구현이 아닌 인터페이스를 노출시키고 싶을 때



### 구조

![image](https://user-images.githubusercontent.com/13410737/185437997-75c1f8e4-c87a-4a19-b918-513aebbb1a15.png){: .align-center}



- **AbstractFactory** : 개념적 제품에 대한 객체를 생성하는 연상으로 인터페이스를 정의한다.
- **ConcreteFactory** : 구체적인 제품에 대한 객체를 생성하는 연산을 구현한다.
- **AbstractProduct** : 개념적 제품 객체에 대한 인터페이스를 정의한다.
- **ConcreteProduct** : 구체적으로 팩토리가 생성할 객체를 정의하고, **AbstractProduct**가 정의하는 인터페이스를 구현한다.
- **Client** : AbstractFactory 와 AbstractProduct 클래스에 선언된 인터페이스를 사용한다.



### 관계

- 일반적으로 ConcreteFactory 클래스의 인스턴스 한 개가 런터임에 만들어지고 어떤 특정 구현을 갖는 제품 객체를 생성한다. 
  - 서로 다른 제품 객체를 생성하려면 사용자는 서로 다른 팩토리를 사용해야 한다.
- AbstractFactory 는 필요한 제품 객체를 생성하는 책임을 ConcreteFactory 서브 클래스에 위임한다.



### 추상 팩토리의 장단점

#### 장점

1. 구체적인 클래스를 분리한다.
   - 응용 프로그램이 생성할 객체의 클래스를 제어할 수 있다. 팩토리는 제품 객체를 생성하는 과정과 책임을 캡슐화했기 때문에, <u>구체적인 구현 클래스가 사용자에게서 분리된다.</u>
   - 제품 클래스 이름이 구체 팩토리(ConcreteFactory) 의 구현에서 분리되므로, 사용자 코드에는 나타나지 않는다.
2. 제품군을 쉽게 대체할 수 있도록 한다.
   - 구체 팩토리를 변경하기 쉽다. 또한, 구체 팩토리를 변경함으로써 응용프로그램은 서로 다른 제품을 사용할 수 있게 변경된다.
3. 제품 사이의 일관성을 보장 시킨다.



#### 단점

1. 새로운 종류의 제품을 제공하기 어렵다.
   - 생성되는 제품은 추상 팩토리가 생성할 수 있는 제품 집합에만 고정되어 있기 때문에 새로운 종류의 제품을 만들기 휘새 기존 추상 팩토리를 확장하기가 쉽지 않다.  
   - 새로운 종류의 제품이 등장하면 팩토리의 구현을 변경해야 한다. 이는 추상 팩토리와 모든 서브 클래스의 변경을 가져오게 된다.



### 예제 코드

미로를 생성하는 프로그램을 작성한다고 가정한다. `MazeFactory` 클래스는 미로의 구성 요소들, 방, 벽과 방 사이의 문을 만들어 낸다.

```c++
class MazeFactory {
  public:
  MazeFactory();
  
  virtual Maze* MakeMaze() const{
    return new Maze;
  }
  virtual Wall* MakeWall() const{
    return new Wall;
  }
  virtual Room* MakeRoom(int n) const{
    return new Room(n);
  }
  virtual Door* MakeDoor(Room* r1, Room* r2) const{
    return new Door(r1, r2);
  } 
}
```

추상 팩토리를 적용하지 않은 앞의 예제에서의 `CreateMaze()` 메서드는 다른 메서드를 이용해서 방들 사이에 문이 있는 방 두 개 짜리 미로를 만들어 내는 역할을 한다. `CreateMaze`는 클래스 이름이 하드코딩되어 있기 때문에 서로 다른 구성요소를 가지고 미로를 만들어 내기가 힘들다. 아래 코드는 `MazeFactory`를 매개변수로 받도록 하여 그 문제를 해결한 `CreateMaze` 메서드이다.

```c++
Maze* MazeGame::CreateMaze(MazeFactory& factory) {
  Maze* aMaze = factory.MakeMaze();
  Room* r1 = factory.MakeRoom(1);
  Room* r2 = factory.MakeRoom(2);
  Door* aDoor = factory.MakeDoor(r1, r2);
  
  aMaze->AddRoom(r1);
  aMaze->AddRoom(r2);
  
  r1->SetSide(North, factory.MakeWall());
  r1->SetSide(East, aDoor);
  r1->SetSide(South, factory.MakeWall());
  r1->SetSide(West, factory.MakeWall());
  
  r2->SetSide(North, factory.MakeWall());
  r2->SetSide(East, factory.MakeWall());
  r2->SetSide(South, factory.MakeWall());
  r2->SetSide(West, aDoor);
  
  return aMaze; 
}
```

`EnchantedMazeFactory` 를 `MazeFactory`에서 서브클래싱한 후, 메서드를 재정의하여 Room, Wall을 상속하는 다른 서브클래스의 인스턴스를 반환하게 만든다.

```c++
// 마법이 걸린 미로 팩토리
class EnchantedMazeFactory : public MazeFactory{
public:
  EnchantedMazeFactory();
  
  virtual Room* MakeRoom(int n) const{
    return new EnchantedRoom(n, CastSpell());
  }
  virtual Door* MakeDoor(Room* r1, Room* r2) const{
    return new DoorNeedingSpell(r1, r2);
  }
  
protected:
  Spell* CastSpell() const;
}
```

만약 폭탄이 장착된 방을 만들고 싶다거나 폭탄이 터진 후 손상된 벽의 모습을 구현하려면 아래와 같이 만들면 된다.

```c++
// 폭탄이 장착된 미로 팩토리
class BombedMazeFactory : public MazeFactory{
  Wall* BombedMazeFactory::MakeWall() const{
    return new BombedWall;
  }
  Room* BombedMazeFactory::MakeRoom(int n) const{
    return new RoomWithABomb(n);
  }
}
```

폭탄이 들어 있을 수 있는 간단한 미로를 만드려며, 그냥 `BombedMazeFactory`를 `CreateMaze`에 넘겨서 호출하면 된다.

```c++
MazeGame game;

// 생성하고자 하는 요소를 생성하는 MazeFactory의 서브클래스인 BombedMazeFactory의 인스턴스 정의
BombedMazeFactory factory;

// CreateMaze 메서드 호출 시 생성의 책임을 지닐 BombedMazeFactory 인스턴스를 매개변수로 전달
game.CreateMaze(factory);
```

마법이 걸린 미로를 만드려면, 마찬가지로 `EnchantedMazeFactory`의 인스턴스를 `CreateMaze` 메서드에 넘기면 된다. 

위와 같은 모습이 추상 팩토리 패턴을 구현하는 가장 일반적인 방법이다. `MazeFactory` 는 팩토리 메서드를 모두 정의하는 구체 클래스이므로 변경할 메서드를 재정의하여 새로운 `MazeFactory`를 쉽게 만들 수 있다.



### 잘 알려진 사용처

- InterView 에서는 "Kit" 라는 접미어를 이요해서 추상 팩토리 클래스임을 표시하였다. Ex) WidgetKit, DialogKit 등..
- ET++(WGM88)은 서로 다른 윈도우 시스템에 걸친 이식성을 제공하기 위해 추상 팩토리 패턴을 사용한다.



### 관련 패턴

- AbstractFactory 클래스는 팩토리 메서드 패턴을 이용해서 구현되는데, 원형 패턴을 이용할 때도 있다. 

- 구체 팩토리는 단일체 패턴을 이용해 구현하는 경우가 많다.
