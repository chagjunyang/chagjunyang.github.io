---
layout: post
title: Swift Stuct vs Class  and closure vs block
categories: [iOS]
tags: [iOS 정보]
comments: false

---

[원문](https://www.letmecompile.com/swift-struct-vs-class-%EC%B0%A8%EC%9D%B4%EC%A0%90-%EB%B9%84%EA%B5%90-%EB%B6%84%EC%84%9D/)


# Swift

### Struct

- value타입 -> 함수 전달 시 value copy 일어남
- stack memory에 할당되어 빠름,
- 컴파일타임에 언제 메모리가 할당/해제되는지 알고있음
- CPU 캐시 히트율이 높음
- 상속 불가
- 새로운 변수로 copy가 일어나는 값타입이여서 멀티스레드환경에 이슈 확률이 적어질 수 있음

### class

- 참조 타입
- 힙 영역의 메모리 할당
- 실행시간에 alloc하여 reference counting에 의한 dealloc dl vlfdy
- 상속 가능
- deinitializer 존재

### 특수 케이스: 클로져와 struct

- 클로져에서 참조하는 strcut 는 참조타입으로 취급
- 값 타입으로 처리하려면 캡쳐드리스트에 추가

#### reference type ex)
``` swift
var anInteger = 42
 
let testClosure = {
    // anInteger는 capture되는 순간 reference copy됨
    print("Integer is: \(anInteger)")
}
 
anInteger = 84
 
testClosure()
// Prints "Integer is 84"
```

#### value type ex)

``` swift
var anInteger = 42
 
let testClosure = { [anInteger] in
    // anInteger는 capture되는 순간 value copy됨
    print("Integer is: \(anInteger)")
}
 
anInteger = 84
 
testClosure()
// Prints "Integer is 42"
```

### Objc > block

- 블럭이 선언되는 순간의 값 타입이 복사, 즉 값 타입으로 동작함
- anInterger 를 블럭에서 변형하고 싶다면, __block 사용


``` objc
int anInteger = 42;

void (^testBlock)(void) = ^{
     // anInteger는 capture되는 순간 value copy됨
    NSLog(@"Integer is: %i", anInteger);
};

anInteger = 84;

testBlock();
// Prints "Integer is: 42"
```
