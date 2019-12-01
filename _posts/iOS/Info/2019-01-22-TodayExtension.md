---
layout: post
title: Today Extension  정리
categories: [iOS]
tags: [iOS 정보]
description: 프로젝트에 Today Extension  적용하기 가이드
comments: false

---

# app extension
<img src="/assets/media/iOS/TodayExtension1.png" width='350'/>

## 용어 정리
* app extension : 앱의 기능을 확장해서 사용할 수 있기 위한 용도.(앱이 아님)
* containing app : app extension을 포함 한 앱
* host app : app extension을 호출한 앱


<img src="/assets/media/iOS/TodayExtension2.png" width='350'/>

## 라이프 사이클
<img src="/assets/media/iOS/TodayExtension3.png">

1. 호스트 앱에서 데이터 공유하기 위해 적절한 app extension 선택
2. app extension 인스턴스화한 후 호스트 앱과 app extension 의 통신 채널 생성. 호스트 앱 위에 app extension ui 노출한 후 데이타 전달 받음.(공유하고 싶은 텍스트나 이미지 등...)
3. 작업 수행 후, app extension 화면 내린 후에 호스트 앱으로 이동.
4. app extension 종료

## 통신
<img src="/assets/media/iOS/TodayExtension4.png">

* containing app과 app extension은 직접적인 통신 하지 않음.
* containing app과 host app은 통신하지 않음.
* containing app과 app extension 은 별도로 실행 됨.(app extension이 실행된다고 해서 containing app이 실행되야 하는건 아님)
* today widget 경우에는 app extension에서 containg app을 호출하기 때문에 점선으로 연결 됨.

<img src="/assets/media/iOS/TodayExtension5.png">

## containing app과의 데이타 공유

<img src="/assets/media/iOS/TodayExtension6.png">

* containing app bundle에 app extension bundle 포함되어 있음.
* 각자의 컨터이너에 접근 할 수 없기 때문에 공유 컨테이너 통해서 읽기 및 쓰기 실행.

# 공유 컨테이너

* Capabilities 메뉴에서 App Goups 활성화

<img src="/assets/media/iOS/TodayExtension7.png">

## payco 앱에서 공유 컨테이너

### 1.entitlements 파일이란?

* ios 앱의 특정 기능 또는 보안 권한을 부여.
* ios에서는 모든 앱이 자동으로 샌드 박스 됨 -> 샌드 박스 권한 받을 필요 없음.


1. iCloud 데이터 공유 권한
2. 푸시 노티피케이션
3. apple pay

### 2.payco app

<img src="/assets/media/iOS/TodayExtension8.png">

### 3.app extension (service short cut)

<img src="/assets/media/iOS/TodayExtension10.png">

# today extension

* 위젯에서 스크롤은 userInteraction이 겹치기 때문에 권하지 않음.


## max height
* display height (캡쳐화면 회색 영역.)
* device 별로 max height 달라짐.
* iOS 10 이상부터 위젯 maxHeight 메소드 제공

<img src="/assets/media/iOS/TodayExtension11.png" width='320'/>

``` objc
//1.
- (void)widgetActiveDisplayModeDidChange:(NCWidgetDisplayMode)activeDisplayMode withMaximumSize:(CGSize)maxSize
{
    /*
     iOS 10이상부터 펼치기/접기 기능을 위해 제공.
     ㄴNCWidgetDisplayModeCompact : 접힌 상태 (디폴트 사이즈)
     ㄴNCWidgetDisplayMode : 펼쳐진 상태
     */
    
    //preferredContentSize 값이 달라져야지 펼치기/접기 애니메이션 실행
    self.preferredContentSize = maxSize;
}

//2.

/*
 displayMode에 따른 maxSize retur
 */
[self.extensionContext widgetMaximumSizeForDisplayMode:NCWidgetDisplayModeCompact];
```

## iOS 9
<img src="/assets/media/iOS/TodayExtension12.png" width='320'/>

* 특별한 기능 없음.
* 지정한 위젯 사이즈에서 기능 넣으면 됨.
* 높이 지정 가능.

## iOS 10
<img src="/assets/media/iOS/TodayExtension13.png" width='320'/>

* 위젯 컨테이너 유아이가 달라짐.
* iOS10 부터 펼치기 접기 기능 제공 (설정에 따라 enable 기능 enable 가능)
* 높이 : 시뮬레이터에서는 110pt로 고정되어 있지만 `실제 기기에서는 os따라 높이가 달라짐.`


## iOS 11
<img src="/assets/media/iOS/TodayExtension15.png" width='320'/>
* iOS 10과 스펙 동일해 보임.

## iOS 12
<img src="/assets/media/iOS/TodayExtension16.png" width='320'/>
* iOS 10과 스펙 동일해 보임.

## NCWidgetProviding Delegate 

``` objc
- (void)widgetPerformUpdateWithCompletionHandler:(void (^)(NCUpdateResult))completionHandler
{
    //위젯이 꺼져 있어도 최신 상태를 유지하기 위해 뷰를 업데이트 하는 기능 지원.
    completionHandler(NCUpdateResultNewData); //새로운 데이타 존재 >> 뷰 업뎃
    completionHandler(NCUpdateResultNoData); // 데이타 없음 >> 뷰 업뎃 x
    completionHandler(NCUpdateResultFailed); //error >> 어떤 동작하는지 파악 못함.
}

- (void)widgetActiveDisplayModeDidChange:(NCWidgetDisplayMode)activeDisplayMode withMaximumSize:(CGSize)maxSize
{
    //iOS 10이상부터 펼치기/접기 기능을 위해 제공.
    //NCWidgetDisplayModeCompact : 접힌 상태 (디폴트 사이즈)
    //NCWidgetDisplayMode : 펼쳐진 상태
    
    //preferredContentSize 값이 달라져야지 펼치기/접기 애니메이션 실행
    self.preferredContentSize = maxSize;
}


- (UIEdgeInsets)widgetMarginInsetsForProposedMarginInsets:(UIEdgeInsets)defaultMarginInsets
{
    //iOS10부터 deprecated
    //위젯 content 마진
    return UIEdgeInsetsZero;
}
```

## iOS10 이상부터 접기/펼치기 기능 설정
* NSExtensionContext + NCWidgetAdditions
* widgetLargestAvailableDisplayMode : 접기/펼치기 기능 설정.
ㄴ NCWidgetDisplayModeCompact (`default`) : 고정 높이로 설정.
ㄴ NCWidgetDisplayModeExpanded : 동적으로 높이 제어.
* widgetActiveDisplayMode : today extesion 현재 활성화된 상태.
ㄴ NCWidgetDisplayModeCompact : 접혀져 있는 상태.
ㄴ NCWidgetDisplayModeExpanded (`default`) : 펼쳐져 있는 상태


``` objc
if (@available(iOS 10.0, *))
{
	[self.extensionContext setWidgetLargestAvailableDisplayMode:NCWidgetDisplayModeExpanded];
}
```

### 참고
* viewDidLoad에서 widgetLargestAvailableDisplayMode 값 셋팅해야지 정상 동작 함.
ㄴ [paycoapp-qa/4893 &#91;IOS_멤버십&#93; 위젯 &gt; 멤버십 설정 완료 후 위젯 접근 시 간략히보기 버튼이 노출됩니다.](dooray://1387695619080878080/tasks/2415048152803864759 "closed")
ㄴ [paycoapp-qa/4900 &#91;iOS_위젯&#93; 멤버십 위젯 미등록 상태에서 &#39;간략히 보기&#39; 문구가 일시적으로 노출되고 있습니다.](dooray://1387695619080878080/tasks/2415162064142232795 "closed")

## 라이프 사이클

### dealloc
1. 앱 실행.
2. 백그라운드 앱 -> 포그라운드 앱 으로 전환.
3. 편집 상태로 전환.(홈버튼 3번)
4. 등등등...


### init
1. viewDidLoad
2. widgetActiveDisplayModeDidChange
3. viewWillLayoutSubviews
4. viewWillAppear
5. viewDidAppear

### change displayMode
1. widgetActiveDisplayModeDidChange
2. viewWillLayoutSubviews
