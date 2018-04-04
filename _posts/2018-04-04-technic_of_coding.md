---
layout: post
title: "C++ 코딩의 기술"
tags: [design patten, OOP]
---

<!-- TOC -->

- [코딩의 기술](#코딩의-기술)
  - [객체지향 설계의 기본](#객체지향-설계의-기본)
    - [캡슐화](#캡슐화)
    - [응집도](#응집도)
    - [결합도](#결합도)
    - [상속과 이양(위임)](#상속과-이양위임)
  - [객체지향 설계의 원칙](#객체지향-설계의-원칙)
    - [단일 책임의 원칙 (SRP, Single Responsibility Principle)](#단일-책임의-원칙-srp-single-responsibility-principle)
    - [개방 폐쇄 원칙 (OCP, Open-Closed Principle)](#개방-폐쇄-원칙-ocp-open-closed-principle)
    - [리스코프 치환 원칙 (LSB, Liskov Substitution Principle)](#리스코프-치환-원칙-lsb-liskov-substitution-principle)
    - [인터페이스 분리 원칙 (ISP, Interfcae Segregation Principle)](#인터페이스-분리-원칙-isp-interfcae-segregation-principle)
    - [의존 관계 역전 원칙 (DIP, Dependency Inversion Principle)](#의존-관계-역전-원칙-dip-dependency-inversion-principle)
    - [데메테르 법칙](#데메테르-법칙)
  - [디자인 패턴](#디자인-패턴)
    - [Template Method 패턴](#template-method-패턴)
    - [Strategy 패턴](#strategy-패턴)
    - [State 패턴](#state-패턴)
    - [Composite 패턴](#composite-패턴)
    - [Iterator 패턴](#iterator-패턴)
    - [Observer 패턴](#observer-패턴)
    - [Singleton 패턴](#singleton-패턴)
  - [클래스 설계 요령](#클래스-설계-요령)
    - [클래스 설계의 기본](#클래스-설계의-기본)
    - [클래스의 역할](#클래스의-역할)
    - [클래스의 책임](#클래스의-책임)
    - [클래스의 추상도](#클래스의-추상도)
    - [클래스 결합](#클래스-결합)
    - [추상 인터페이스 사용 방법](#추상-인터페이스-사용-방법)
    - [그밖에](#그밖에)
      - [생성자로 완전한 상태 만들기](#생성자로-완전한-상태-만들기)
      - [멤버 함수 호출 순서와 관련한 대처 방법](#멤버-함수-호출-순서와-관련한-대처-방법)
      - [초기화는 생성자, 뒤처리는 소멸자로 시행](#초기화는-생성자-뒤처리는-소멸자로-시행)
      - [구현 상세를 은폐하는 이름 붙이기](#구현-상세를-은폐하는-이름-붙이기)

<!-- /TOC -->

# 코딩의 기술

## 객체지향 설계의 기본

### 캡슐화

* getter 와 setter 를 되도록 사용하지 말것.
* 부모 자식 관계라도 private 을 사용하자.

```java
// bad
Vector2 postion = player.getPosition();
position += player.getVelocity();
player.setPosition(position);

// good
// '직접하지 말고 명령하라'
player.move();
```

### 응집도

* 클래스의 멤버 변수는 최소한으로 만들자.

```java
// bad
class Game {
  private:
    int   score;
    float timer;
    list<Actor*> actors;
};

// good
// 클래스 멤버 변수가 1개라면 하나의 역할에 집중할 수 있다.
class Score {
  private:
    int score;
}

class Timer {
  private:
    float time;
}

class ActorManager {
  private:
    list<Actor*> actors;
}

class Game {
  private:
    Score score;
    Timer timer;
    ActorManager actors;
}
```

### 결합도

* 결합도를 낮게 설계하자.
* 단위 테스트가 어렵다면 결합도가 높은 것.

### 상속과 이양(위임)

* 상속보다는 이양을 우선해서 설계하자.

```java
// bad
class Robot {
  public:
    void update() {
      move();
    }

  private:
    virtual void move() = 0;
}

class CleanerRobot : public Robot {
  private:
    virtual void move() ovveride {}
}

class CombatRobot : public Robot {
  private:
    virtual void move() override {}
}

// good
class Robot {
  public:
    Robot(RobotBehavior* behavior) : _behavior(behavior) {}
    ~Robot() {
      delete _behavior;
    }
    void update() {
      _behavior->move();
    }
  private:
    RobotBehavior* _behavior;
  }

class CleanerBehavior : public RobotBehavior {
  public:
    virtual void move() override {}
}

class CombatBehavior: public RobotBehavior {
  public:
    virtual void move() override {}
}

// usage
Robot cleanerRobot(new CleanerBehavior());
Robot combatRobot(new CombatBehavior());
```

* 상속 관계에선 실행중 변경이 어렵고, 이양을 사용하면 가능하다.

```java
cleaner.Robot.changeBehavior(new CombatBehavior());
```

* 다른 유형의 클래스를 만들때도 재사용할 수 있다.

```java
SuperRobot superRobot(new CombatBehavior());
```

## 객체지향 설계의 원칙

### 단일 책임의 원칙 (SRP, Single Responsibility Principle)

* 하나의 클래스는 하나의 책임만 가져야 한다.

### 개방 폐쇄 원칙 (OCP, Open-Closed Principle)

* 소프트웨어 구성 요소는 확장에 관해서는 열려있어야 하고, 변경에 대해서는 닫혀있어야 한다.
* 변화하지 않는 부분과 열린 부분을 분리하자.
* 추상 클래스 (닫힌 부분), 구현 클래스 (열린 부분)

### 리스코프 치환 원칙 (LSB, Liskov Substitution Principle)

* 파생 자료형은 기본 자료형과 치환할 수 있어야 한다.
* 부모 클래스로 치환한 상태에서도 정상 작동해야 한다.
* 깊은 상속관계를 만들지 말고, 위임을 우선하자.

### 인터페이스 분리 원칙 (ISP, Interfcae Segregation Principle)

* 클라이언트가 사용하지 않는 멤버 함수의 의존을 클라이언트에 강요하면 안된다.
* 클래스 사용자에게 불필요한 인터페이스는 공개하지 말라.
* 다수의 멤버 함수를 가진 클래스를 사용할 때 필요한 멤버 함수만으로 한정한 전용 클래스를 따로 만들어간접적으로 사용하는 편이 좋다.

### 의존 관계 역전 원칙 (DIP, Dependency Inversion Principle)

* 상위 모듈은 하위 모듈에 의존하지 않는다, 두 모듈 모두 별도의 추상화 된것에 의존한다.
* 구체적인것이 아닌 추상적인것에 의존하자.

### 데메테르 법칙

* 최소 지식의 원칙, 직접적인 친구와만 관련한다.

```java
// bad
Player* player = world_.getManager().find("player");

// good
Player* palyer = world.find("Player");
```

## 디자인 패턴

### Template Method 패턴

* 부모 클래스에 정형화한 처리 과정을 정의하고, 처리가 다른 부분은 자식 클래스로 구현하는 패턴
* 클래스 라이브러리의 애플리케이션 프레임워크 등에 자주 사용됨
* 상속을 재사용한 고전적인 패턴.

### Strategy 패턴

* 알고리즘이 변화(전략)하는 부분을 클래스화해서 교환할 수 있게 하는 패턴
* switch 조건문이 나오면 Strategy 패턴으로 해결할 수 없는지 살펴볼 것
* Strategy 패턴은 추상 인터페이스를 사용해서 구현 부분을 교환하는 고전적인 패턴

### State 패턴

* 객체가 가지는 여러가지 상태를 클래스화하는 패턴
* Strategy 패턴과 구현방법이 거의 비슷하지만, 사용하는 상황이 다르다.

### Composite 패턴

* 트리 구조로 재귀적인 데이터를 만들어 객체를 관리하는 패턴
* 게임 엔진의 게임 객체 관리 등에 자주 사용됨.

### Iterator 패턴

* 컨테이너 클래스 내부 요소에 순서대로 접근하는 방법을 제공한다.
* 자료구조의 구현 상세를 캡슐화

### Observer 패턴

* 객체의 상태 변화를 다른 객체에 통지해주는 패턴

### Singleton 패턴

* 객체 인스턴스화를 1 개로 제한하고, 전역에서 접근할 수 있게 하는 패턴

## 클래스 설계 요령

### 클래스 설계의 기본

* 클래스 설계는 현실 세계의 업무 분담과 비슷
* 각 클래스의 역할을 정하고, 업무 책임을 분담하는 작업
* 클래스 사용 관점에서는 자료구조와 알고리즘 구현보다는 어떤 역할과 책임이 있는지가 중요함.

### 클래스의 역할

* 작업과 관리, 중재, 창구, 생성, 전용과 범용

- 작업과 역할 클래스와 관리 클래스
  * 작업 역할 클래스는 구체적인 구현을, 관리 역할 클래스는 작업 역할 클래스를 제어
  * 개자이너 -> 관리자 - 설계자/프로그래머/디자이너
  * 1 개의 클래스에 너무 많은 일을 주지 말것.

- 중재 역할 클래스
  * 객체들의 조정을 담당
  * Mediator 패턴에 해당함
  * 중재자를 사용해 의존 관계를 단순화 하자.

- 창구 역할 클래스
  * 작업을 의뢰받고 실질적인 작업을 다른 클래스에 전달하는 역할
  * Facade 패턴에 해당
  * 외부와의 인터페이스 부분을 클래스화 하자.

- 생성 역할 클래스
  * 객체 생성을 수행하는 공장과 같은 클래스
  * 객체의 변경 또는 수정이 간단해지고 연관을 줄일 수 있다.
  * Abstract Factory 패턴

- 전용클래스와 범용 클래스
  * 특정 애플리케이션에서만 사용하는 전용 클래스와 재사용을 목적으로 하는 범용 클래스를 구별해서 설계하자.

### 클래스의 책임

* 명확한 역할과 책임을 갖게 하는게 중요하다.
* 사용자 측의 코드가 간단해질 수 있도록 담당 클래스에 제대로 이양하자.

### 클래스의 추상도

* 추상도가 높으면 코드가 단순해져서 수정 또는 변경에 유연하게 대응할 수 있다.
* '문제의 본질'을 파악해서 추상도를 높이자.
* 클래스 사용자측이 시점에서부터 탑다운(top-down) 해서 생각하면 추상도 높은 클래스를 설계할 수 있다.
* 추상도 높은 인터페이스를 어떻게 생성할지 생각해보자.

### 클래스 결합

* 추상 인터페이스를 활용해 소결합으로 유도하자.

### 추상 인터페이스 사용 방법

* 클래스 공통화: 부모 클래스 또는 추상 클래스를 사용해 클래스를 관리하는 것과 같은 방법
* 구현 수단 교환: Strategy 패턴
* 변화하기 쉬운 부분 분리: 개방/폐쇄 원칙
* 부적절한 결합 관계 분리: 중재 역할 또는 창구 역할 담당, 직접 결합을 분리해서 소결합으로 만듦
* 패키기 경계 분리: 패키지를 독립시키고 외부 변경의 영향을 피할 때, 의존관계 역전의 법칙

### 그밖에

#### 생성자로 완전한 상태 만들기

* 객체는 생성자가 호출된 시점에 완전한 상태를 생성해야 한다.
* 호출 순서에 제한이 있는 클래스는 사용자에게 부담되고, 버그의 원인이 된다

#### 멤버 함수 호출 순서와 관련한 대처 방법

* 호출 순서에 제약이 있는 클래스는, 제어 역할을 하는 클래스와 함께 조합해서 사용

#### 초기화는 생성자, 뒤처리는 소멸자로 시행

#### 구현 상세를 은폐하는 이름 붙이기

```java
// bad!
// 사용자 입장에서는 flag 수정하는것을 알필요가 없다.
// 멤버함수 이름이 바뀌면 사용자측 코드도 변경되어야 한다.
bool getDeadFlag() {
  return isDead;  
}

// good!
bool isDead() {
  return isDead;
}
```
