---
layout: post
title: iOS App Life Cycle
categories: [iOS]
tags: [iOS 정보]
comments: false

---

[참고](https://velog.io/@cskim/iOS-App-%EC%83%9D%EB%AA%85%EC%A3%BC%EA%B8%B0Life-Cycle)

### Life Cycle

1. App touch
2. main() 호출 : 스위프트의 경우 @UIApplicationMain어노테이션
3. main함수 안에서 UIApplicationMain()호출 -> UIApplication 객체 생성
4. UIApplication 객체는 info.plist 파일을 바탕으로 앱에 필요한 데이터와 객체 로드
5. AppDelegate 객체 생성 후 UIApplication 객체와 연결
6. application(_:willFinishLaunchingWithOptions:)호출
7. application(_:didfinishLaunchingWithOptions:) 호출 

### 앱 상태변화

<img src="/assets/media/iOS/Background1.png">

#### Not Running

- 앱이 실행되지 않았거나 완전히 종료된 상태

#### Foreground > Inactive

- 앱이 전면에 나타나있으나 이벤트를 받지 않음, 
- Active로 넘어가는 잠시의 순간이거나 알림센터를내렸을경우

#### Foreground > Active

- 이벤트를 받을 수 있음

#### Background

- 앱이 메모리에 올라와있고, 동작을 수행하고있는 상태
- Suspended가 되기 전에 잠시 머무르는 상태이기도함.
- 명시적 background코드가 없어도 Suspended가 되기전에 추가작업을 위한 시간을 주기도함

#### Suspend

- 앱이 메모리에만 올라가있고 실행하지 되지않음
- OS에서는 메모리부족 시 suspended상태의 app드을 별로의 알림없이 종료시켜 메모리 확보할 수 있음
