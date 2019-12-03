---
layout: post
title: 디자인패턴 공부
categories: [iOS]
tags: [iOS 정보]
description: 맛보기
comments: false

---

# 1.UML

## 정의

- 모델링 언어의 대표
- 표준 통합 모델링 언어

## 종류

### 구조

- 클래스 다이어그램
- 객체 다이어그램
- 복합체 구조 다이어그램
- 배치 다이어그램
- 컴포넌트 다이어그램
- 패키지 다이어그램

### 행위

- 활동 다이어그램
- 상태 머신 다이어그램
- 유즈케이스 다이어그램
- 상호작용 (interaction) 다이어그램

## 클래스 다이어그럼

### 관계

- 연관 관계 
- 의존 관계 
- 집합 관계 
- 일반화 관계 : 상속
- 실체화 관계 : 인터페이스 구현

### 연관관계, 의존관계

- 둘 다 모두 서로 상속, 집합 등으로 묶여있지 않음
- 한 클래스가 수정되면 다른클래스에 수정해야 하는 상황 발생 가능
- 연관관계 : 참조가 전역적으로 일어남. 라이프사이클이 같음
- 의존관계 : 참조가 지역적으로 일시적


# 2. 객체지향 원리

## 1. 추상화

- 사물의 공통된 특징을 추상적으로 파악해 대상을 인식하는 과정
- 수만의 특징 중 내가 관심을 갖는 것을 추사황 대상으로 삼는다.
- 예를들어 학생의 경우 부모님은 머리, 옷, 성적이 관심대상이고 교수님입장에서는 학번, 지각률 등이 관심대상일 것이다.


## 2. 캡슐화

- 요구사항 변화에 유연하려면 응집도를 높이고 결합도를 낮춘다.
- 캡슐화는 낮은 결합도를 유지하는데 도움이되는 객체지향 개념이다.
- 내부의 로직을 숨기고 바깥에 특정 인터페이스만 연결하여 그것을 사용하게한다.
- 바깥에서 변경을 최소화하여 서로의 결합도를 낮추는 것

## 3. 일반화 

- 상속이라고도 표현
- 잘못된 상속을 하면 부모클래스의 불필요한 정보까지 자식이 알게된다
- 이럴 경우 삭송이아닌 `위임`을 통해 해결한다.
- 위임이란 자신이 직접 기능을 실행하지 않고 다른 클래스의 객체가 기능을 실행하도록 위임하는 것
- 즉, 일반화 관계에서 연관관계 혹은 의존관계로 해결한다.

## 4. 다형성

- 서로다른 클래스의 객체가 같은 메시지를 받알을 때 각자의 방슥으로 동작하는 능력
- 예를들어 A클래스를 상속받아 a메소들 오버라이드하는 B,C클래스의 a메소드는 다르게 동작 할 것이다.
- 혹은 추상클래스에 정의된 a메소드를 구현하는 B,C클래스의 a메소드는 다르다.
- 다형성은 Composite패턴의 핵심이다

# 3. SOLID 원칙

## SRP (Single Responsibility Principle)

- 단일 책임 원칙
- 객체는 단 하나의 책임만 가져야 한다
- 흔히들 많은 책임을 하나의 객체가 수행하게 코딩한다. 이를 여러객체로 역할에 맞게 분산시켜야 함

## OCP (Open Closed Principle)

- 개방 폐쇠 원칙
- 기존코드를 변경하지 않으면서 기능을 추가할 수 있또록 설계가 되어야 한다.
- 이 때 좋은 방법충 하나가 캡슐화이다.
- 추상클래스에 메소드 a를 넣어두고, 요구사항이 추가되어 a메소드의 역할이 상황에따라 변경된다면 a메서드를 구현하는 개체만 하나 추가하면 된다.

## LSP (Liskov Substitution Principle)

- 리스코프 치환 원칙
- 일반화 관계에서 자식이 부모의 역할을 수행할 수 있어야 한다는 것
- 예를들어보자

포유류 <- 원숭이

포유류는 알을 낳지 않고 새끼를 낳아 번식한다.
포유류는 젖을 먹여서 새끼를 키우고 폐를 통해 호흡한다.

원숭이는 알을 낳지 않고 새끼를 낳아 번식한다.
원숭이는 젖을 먹여서 새끼를 키우고 폐를 통해 호흡한다.

## DIP (Denpendency Inversion Principle)

- 의존 역전 원칙
- 의존 관계를 맺을 떄의 가이드라인
- 자주 변화하는 것보다는 변화하기 어려운것에 의존하라는 원칙
- DIP 는 유연한 프로그램에 필수이다.
- DI (Denpendency injection) 기술로 변화헤 쉽게 수용가능한 코딩이 가능하다

## ISP (Interface Segregation Principle)

- 인터페이스 분리 원칙
- 인터페이스를 클라이언트에 특화되도록 분리시키는 설계 원칙
- 클라이언트 단위로 혹은 기능 단위로 인터페이스를 묶어 코딩
- 이 역시 DIP와 마찬가지로 의존성 주입을 통해 유연한 코딩이 가능해짐


# 4. 다지안 패턴

## 디자인 패턴의 이해

- 문제 해결을 위해 동일한 유형이나 양식을 기술하는 것
- 한 번 하면 두번다시 하지않고 해결할 수 있는 템플릿 같은 것

## Gof 디자인 패턴


- 생성 패턴 : 객체 생성에 관련된 패턴으로, 객체 생성과 조합을 캡슐화해 특정 객체가 생성되거나 변경되어도 프로그램에 구조에 영향을 주지 않도록 유연성 제공

- 구조 패턴 : 클래스나 객체를 조합해 더 큰 구조를 만드는 패턴
- 행위 패턴 : 개개체나 클래스 사이의 알고리즘이나 책임 분배에 관련된 패턴


| 생성 패턴 | 구조 패턴 | 행위 패턴 |
| --- | --- | --- |
| 추상 팩토리 | 어댑터 | 책임 연쇄 |
| 빌더 | 브릿지 | 커맨드 |
| 팩토리 메서드 | 컴퍼지트 | 인터프리터 |
| 프로토타입 | 데커레이터 | 미디에이터 |
| 싱글턴 | 퍼사드 | 메멘토 |
|  | 플라이웨이트 | 옵저버 |
|  | 프록시 | 스테이트 |
| | |스트래티지|
| | |템플릿 메서드|
| | |비지터|

---


# 1. 객체 생성

1. 팩토리메서드
2. 추상 팩토리
3. 빌더

## 팩토리 메서드  

-  정의 : 객체를 생성하는 인터페이스를 정의하되, 실제 객체 생성은 해당 객체가 속하는 서브 클래스에서 하도록 한다. 

### 사용하는 시점

- 정확히 어떤 클래스의 객체가 생성될 지 예상할 수 없을때
- 생성되는 객체의 클래스들을 특정 클래스의 서브 클래스들로 두고자 할 때

### 장점

- 클라이언트가 내부를 알 필요없이 인터페이스만 알면됨
- 객체가 추가되거나 수정되어도 클라이언트에 영향이 없거나 적음

### 단점

- 팩토리 메서드패턴은 상속을 기반으로하기 때문에 뎁스가 깊어지거나 잘못 사용되면 복잡해질 수 있다.
- 부모 클래스를 전혀 확장하지 않고 정해진 인터페이스만 사용함 -> 만약 어떤 하위클래스에서 특정 메소드가 필요하면 부모에 선언하고 다른 객체에 불필요한 메소드가 추가되어야함

### ex

``` objc

@interface Pizza : NSObject

- (NSNumber *)price;
+ (Pizza *)pizzaWithName:(NSString *)aName;

@end

@interface SeafoodPizza : Pizza
@end

@interface CheesePizza : Pizza
@end

```

``` objc
@implementation Pizza

+ (Pizza *)pizzaWithName:(NSString *)aName
{
    Pizza *sResult = nil;
    
    if([aName isEqualToString:@"SeafoodPizza"])
    {
        sResult = [SeafoodPizza new];
    }
    else if([aName isEqualToString:@"CheesePizza"])
    {
        sResult = [CheesePizza new];
    }
    
    return sResult;
}

- (NSNumber *)price { return nil;}

@end

@implementation SeafoodPizza : Pizza
- (NSNumber *)price
{
    return @(2000);
}
@end

@implementation CheesePizza : Pizza
- (NSNumber *)price
{
    return @(1000);
}
@end
```

## 추상 팩토리

-  다양한 구성 요소 별로 '객체의 집합'을 생성해야 할 때 유용하다. 이 패턴을 사용하여 상황에 알맞은 객체를 생성할 수 있다
-  단순 객체를 생성해서 반환하는게아닌 연관된 객체간의 의존성 죽입 혹은 직접관여를 하기도함

http://besma.tistory.com/entry/팩토리-메소드-패턴

### 추상 팩토리 vs 팩토리 메소드

- 팩토리 메소드는 하나의 목적을 달성하는 객체를 생성 (상속, 추상화도 가능)
- 추상 팩토리는 팩토리 메소드를 사용하던가, 여러 구성요소의 객체의 집합을 생성하여 반환하므로 추상팩토리 안에서 여러조합의 조합식이 만들어질 수 있음

## 빌더

- https://johngrib.github.io/wiki/builder-pattern/

- 복잡한 객체를 만들 때 사용
- 객체를 생성하는 방법과 표현하는 방법을 분리하는게 핵심

- Builder : 빌더 인터페이스.
- ConcreteBuilder : 빌더 인터페이스 구현체. 부품을 합성하는 방식에 따라 여러 구현체를 만든다.
- Director : Builder를 사용해 객체를 생성한다.
- Product : Director가 Builder로 만들어낸 결과물.

### ex

#### Builder

``` objc
@interface ProductBuilder : NSObject <Builder>

@property (nonatomic) NSString *name;
@property (nonatomic) NSNumber *price;

- (Product *)buildProduct;

@end
```

#### Director
``` objc
@interface ProductDirector : NSObject

- (Product *)makeMonitorWithBuilder:(id<Builder>)builder;
- (Product *)makeKeyboardWithBuilder:(id<Builder>)builder;

@end


@implementation ProductDirector

- (Product *)makeMonitorWithBuilder:(id<Builder>)builder
{
    builder.name = @"alphascan";
    builder.price = @(100000);
    
    return [builder buildProduct];
}


- (Product *)makeKeyboardWithBuilder:(id<Builder>)builder;
{
    builder.name = @"anne pro2";
    builder.price = @(120000);
    
    return [builder buildProduct];
}

@end
```

### Client

``` objc
@interface Client : NSObject
@end

@implentation Client

- (Product *)sellMonitor
{
    ProductDirector *director = [ProductDirector new];
    ProductBuilder *builder = [ProductBuilder new];
    Product *monitor = [director makeMonitorWithBuilder:builder];
    
    return monitor;
}

- (Product *)sellKeyboard
{
    ProductDirector *director = [ProductDirector new];
    ProductBuilder *builder = [ProductBuilder new];
    Product *monitor = [director makeKeyboardWithBuilder:builder];
    
    return monitor;
}

@end
```

# 2. 인터페이스 맞추기

## 어댑터

- 어떤 클래스의 인터페이스를 클라이언트가 원하듣 다른 인터페이스로 변환
- 원본 클래스의 수정없이 인터페이스만 내가 원하는 요구사항에 맞춰 사용하고싶을 때 사용

### 클래스 어댑터

- 올드 클래스를 상속받아 구현

### 객체 어댑터

- 올드 클래스를 멤버변수로 두고 사용해서 구현

## 브리지

- 추상체를 그것의 구현체와 분리시킴으로써 두가지를 독립적으로 분리
- POP에서 상요되는 Protocol과 구현체가 완전히 분리된상태를 뜻함


## 퍼사드

- 외부에서 사용하기쉽게 여러 인터페이스를 하나로 모아서 동작하게하는 것
- 바깥에서 원하는 동작을 하기위해 A, B, C 3개의 메소드를 호출해야했다면  퍼사드클래스를통해 메소드1개만 호출하는 식
- 고수준 인터페이스 제공 

``` java
class CPU {
    public void freeze() { ... }
    public void jump(long position) { ... }
    public void execute() { ... }
}
class Memory {
    public void load(long position, byte[] data) {
        ...
    }
}
class HardDrive {
    public byte[] read(long lba, int size) {
        ...
    }
}
/* Façade */
class Computer {
    public void startComputer() {
        CPU cpu = new CPU();
        Memory memory = new Memory();
        HardDrive hardDrive = new HardDrive();
        cpu.freeze();
        memory.load(BOOT_ADDRESS, hardDrive.read(BOOT_SECTOR, SECTOR_SIZE));
        cpu.jump(BOOT_ADDRESS);
        cpu.execute();
    }
}

/* Client */
class You {
    public static void main(String[] args) throws ParseException {
        Computer facade = /* grab a facade instance */;
        facade.startComputer();
    }
}
```

# 3. 객체 분리

## 미디에이터 
## 옵저버


# 4. 추상 컬렉션

## 컴포지트 패턴

```java
/** "Component" */
interface Graphic {
    public void print();
}

/** "Composite" */
class CompositeGraphic implements Graphic {
    private List<Graphic> mChildGraphics = new ArrayList<Graphic>();
    
    public void print() {
        for (Graphic graphic : mChildGraphics) {
            graphic.print();
        }
    }
    public void add(Graphic graphic) {
        mChildGraphics.add(graphic);
        
        public void remove(Graphic graphic) {
            mChildGraphics.remove(graphic);
        }
    }
    
    class Ellipse implements Graphic {
        public void print() {
            System.out.println("Ellipse");
        }
    }
    
    public class Program {
        public static void main(String[] args) {
            Ellipse ellipse1 = new Ellipse();
            Ellipse ellipse2 = new Ellipse();
            Ellipse ellipse3 = new Ellipse();
            Ellipse ellipse4 = new Ellipse();
            
            CompositeGraphic graphic = new CompositeGraphic();
            CompositeGraphic graphic1 = new CompositeGraphic();
            CompositeGraphic graphic2 = new CompositeGraphic();
            
            graphic1.add(ellipse1);
            graphic1.add(ellipse2);
            graphic1.add(ellipse3);
            
            graphic2.add(ellipse4);
            
            graphic.add(graphic1);
            graphic.add(graphic2);
            
            graphic.print();
        }
    }
}
```


## 이터레이터 패턴

- 컬렉션 구현 방법을 노출시키지 않으면서도 그 집합체 안에 들어있는 모든 항목에 접근할 수 있는 방법을 제공한다.

``` C#
var primes = new List<int>{ 2, 3, 5, 7, 11, 13, 17, 19};
long m = 1;
foreach (var p in primes)
m *= p;
```

# 5. 객체의 행동 확장

## 스트레티지 패턴

- 전략을 쉽게 변경하게 해주는 패턴
- 프로그램에서 특정 알고리즘 혹은 로직등을 쉽게 변경하거나 동적으로 변경될 수 있는데 이 때 적용하기 좋음
- 기존 로직에 새로운 케이스가 생길경우 적용

### 상황

- 로봇이 움직이는데 나는로봇, 걸어가는 로봇, 달려가는로봇 3가지가 있다고 가정
- 로봇의 move라는 추상 메소드를 구현한 flyingStrategy, WalkingStrategy, runingStrategy 3가지 스트래티지(클래스)를 만든다
- 나는 로봇을 만든다면 self.movingStrategy = new flyingStrategy() 이런식으로 DI
