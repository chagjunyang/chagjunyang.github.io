---
layout: post
title: Deferred Deeplink 조사
categories: [iOS]
tags: [iOS > 정보]
description: 개념과 앱에 적용하려면 어떻게 해야하는지 정리
comments: false
---

# 정의

- [참조](http://blogs.innovationm.com/deferred-deep-linking-in-ios-with-universal-link/)


### DeepLink

- 앱이 단순이 열리는게 아닌 앱이열린 후에 동작을 하는 것을 뜻함
- iOS 에서 제공하는 기능
-  universalLink
-  urlScheme


### Deferred DeepLink

<img src="/assets/media/iOS/deferredDeepLink1.png">

- 앱이 설치되지않았을 경우 앱스토어로 이동
- 앱스토에서 앱 설치 후 앱실행 > 원라하고자했던 DeepLink 동작 수행
- iOS 에서 제공하지 않음
- AOS에서는 플레이스토어에 한해서 제공

### 부가 정보 > 스마트 배너

- universalLink용 브릿지페이지에서 사용
- 제대로 열리지않을 경우 저걸 통해 앱을 열기 위함 
- urlScheme 방식으로 동작함
- 애플에서 제공하는 표준
- 아래와 같이 메타태그 정의하면 사파리에서 랜더링

``` html
<meta name="apple-itunes-app" content="app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL">
```
<img src="/assets/media/iOS/deferredDeepLink2.png">


# Deferred Deeplink  적용 방법


## 방법1. 호스팅 서비스 이용

- 호스팅 서비스는 branch가 유명한 것 같고, 사용방법 정리가 필요하다면 따로 정리해야함

### 공급사

- Branch, firebase등 있음
- Deferred DeepLink 뿐만아니라 데이터 수집도 추가   (설치후 유입경로 추적 가능)


### 호스팅 서비스 사용 이유

- 애플과 구글은 이미 딥링크 제공
- 애플 딥링크의 예상치 못한 코너사례들이 많음
- 애플, 구글이 제공하는 기능외에 추가 기능 제공
- deferred deeplink
- 데이터수집 등...


## 방법 2. 직접 구현 


### Branch > 매칭 원리

https://docs.branch.io/resources/matching/
https://blog.branch.io/the-importance-of-matching-accuracy-in-deep-linking/

### 사파리에서 접근했을 경우

- 쿠키 활용
- 유니버셜 링크 브릿지페이지에서 쿠키에 페이코식별가능 키값으로 스킴 저장
- 앱에서는 Safafi View Contorller를 활용해 사파리의 쿠키정보 가져올 수 있음
- 앱구동시 쿠키값 비교하여 실행 할 스킴이 있으면 실행

### 다른 앱에서 접근했을 경우

- facebook API를 활용하여 facebook에서 접근했는지 확인


#### 다른앱이 제휴사여서 우리 브릿지페이지에 IDFA값을 전달해줄 수 있는경우

- 브릿지페이지에서 페이코앱 설치로 보내기전에 IDFA - 동작하려했던 스킴 서버에 저장
- 페이코앱설치 후 IDFA값 가져와 서버에 저장된 IDFA값 있는

#### IDFA 없는경우

- IP Address (including v6), OS, OS version, device model 을 사용자식별키로 사용



## 직접구현 > 웹에서 할일 (브릿지페이지)

<img src="/assets/media/iOS/deferredDeepLink3.png">

- userAgnet를 통해 사파리인지 웹뷰인지 판단
- 모든 저장된 정보는 특정시간이후 삭제하는게 맞는거같음...

### 사파리

- 페이코식별 키 + 스킴정보 쿠키에 저장

### 웹뷰

- IDFA있으면  IDFA + 스킴 서버에 저장
- 없으면 IP Address (including v6), OS, OS version, device model  서버에 저장


## 집저구현 > 앱이 할 일


1. 설치 후 최초구동인지 아닌지 판단
- 이 조건 빼면 유니버셜링크 동작 안했을 때 앱그냥 열면 실행되게할수있을듯
2. facebook API 호출해야 facebook에서 호출된게 있는지 확인
3. SafariViewController를 활용하여 쿠키가져와서 스킴 있는지 확인
4. 서버 API 호출해서 scheme 정보 받아오기  
- 파라미터 : IDFA, IP, OS, OS Version, Device Model
- 결과로 받아온 스킴이 있으면 실행

### IP, OS, OS Version, Device Model...식별자의 유효성

- [브랜치 개발자?? 답변](https://stackoverflow.com/questions/28686508/how-does-branch-io-handle-situations-where-multiple-devices-may-have-the-same-fi)
- 결론적으로 매우 드문확률이고, 등록된 딥링키는 한번동작하면 제거되므로 확률이 더 낮아짐
- 논리적으로 발생할 수 있으나 실질적으로 같은 IP에, OS, 기종에서 제한된 쿠키시간내에 실행할 확률이 낮다고 함
- 이를 위해 옵션 설정할 수 있게 제공 ( 동일한 식별키를 가진 2개의 기기의 딥링크가 등록되어있을 때 동작안할지, 둘다할지 등의 옵션 )
