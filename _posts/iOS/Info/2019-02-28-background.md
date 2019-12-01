---
layout: post
title: Bckground 동작 정리
categories: [iOS]
tags: [iOS 정보]
description: 정의 및 테스트방법 
comments: false

---

# iOS 백그라운드 동작 정리

https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html

### 백그라운드 상태 정의

- 사용자가 앱을 적극적으로 사용하지 않을 경우 시스템에서 앱을 백그라운드상태로 만듬
- 대부분 백그라운드상태에서 앱은 일시정지가되고 시스템에서 리소스관리 및 배터리관리에 도움이됨

<img src="/assets/media/iOS/Background1.png">


### 백그라운드 동작 종류

1. 제한된 시간의 백그라운드작업 수행
2. 백그라운드 다운로드
3. 백그라운드 모드에 따른 지원 (백그라운드패치, ble, 위치 등 다양함)



## 1. 제한된 시간의 백그라운드작업 수행

- 앱이 백그라운드에 들어가도 시스템에서 지원해주는 시간동안 작업수행 가능
- `beginBackgroundTaskWithName:expirationHandler:` or `beginBackgroundTaskWithExpirationHandler:`   메소드 사용
- 메소드호출 시 고유 토큰이생성되고 `endBackgroundTast:`메소드를 통해 해당 토큰의 테스크 종료해야함  (없을시 앱 종료)
- endBackgroundTast:호출은 앱에서 원하는 타이밍에 호출
- 최소한 expirationHandler 에서는 호출되어야함 


### 샘플코드

- https://github.com/chagjunyang/BackgroundTest > feature/backgroundExecute 브랜치

### 포그라운드 상태에서 실행

``` objc
__block UIBackgroundTaskIdentifier taskId = UIBackgroundTaskInvalid;
taskId = [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
NSLog(@"call expiration handler");

[[UIApplication sharedApplication] endBackgroundTask:taskId];
taskId = UIBackgroundTaskInvalid;
}];

//Do your work
```

### 백그라운드 상태 진입 시 수행

- didEnterBackground메소드안에서 오랜시간이 걸리는 동작수행은 보장하지못함  
- 애플에서는 긴 시간이걸리는 동작은 아래와같은 형식으로 사용하라고함

``` objc
- (void)applicationDidEnterBackground:(UIApplication *)application
{
bgTask = [application beginBackgroundTaskWithName:@"MyTask" expirationHandler:^{
[application endBackgroundTask:bgTask];
bgTask = UIBackgroundTaskInvalid;
}];

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
//Do your work

[application endBackgroundTask:bgTask];
bgTask = UIBackgroundTaskInvalid;
});
}
```

## 2. 백그라운드 다운로드

https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_in_the_background?language=objc

- 앱의 사이클과 별도로 진행
- 다운로드가 완료되면 잠시 앱 활성화 

### 백그라운드 다운로드 하는 방법

1. NSURLSessionConfiguration객체 생성
- backgroundSessionConfigurationWithIdentifier 이용
- Configureation 객체의 sessionSendsLaunchEvents 속성 YES로 설정
- 포그라운드상태에서 트랜지션이 시작된다면 discretionary속성을 YES로 설정

2. Download Task 생성
3. 다운로드 완료 이벤트 받기
- 백그라운드상태에서 다운로드실행하면 다른프로세스에서 다운로드가 실행될 동안 앱은 suspend 상태가 됨
- suspend, terminate 상태에서 다운로드 완료 이벤트를 받아와 백그라운드에서 UI를 갱신 할 기회를 가질 수 있음

### 샘플코드

https://github.com/chagjunyang/BackgroundTest.git   >  feature/backgroundDownload   브랜치

### terminate 상태에서 다운로드 완료 이벤트 받기

Appdelegate
``` objc
// 앱이 백그라운드에 있는 경우 앱이 중지될 수 있고, 이 경우 다운로드가 끝나면 시스템에서 아래 메소드 호출
// completionHandler를 들고있다가 호출하면 앱의 사용자 인터페이스가 업데이트되고 새 스냅샷을 가져올 수 있을음 시스템에게 알림
- (void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)(void))completionHandler
{
_backgroundURLSessionCompletionHandler = completionHandler;
}
```

ViewControlelr
``` objc
#pragma mark - NSURLSessionDelegate

- (void)URLSessionDidFinishEventsForBackgroundURLSession:(NSURLSession *)session API_AVAILABLE(ios(7.0), watchos(2.0), tvos(9.0)) API_UNAVAILABLE(macos)
{
dispatch_async(dispatch_get_main_queue(), ^{
AppDelegate *sAppDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];

if(sAppDelegate.backgroundURLSessionCompletionHandler != nil)
{
sAppDelegate.backgroundURLSessionCompletionHandler();
}
});
}
```

### 

## 3. 백그라운드 모드

- 다양한 백그라운드모드가 있지만 일단 backgroundFetch만 조사

<img src="/assets/media/iOS/Background2.png">


## 백그라운드 패치

### 정의
- OS에서 주기적으로 백그라운드 상태의 앱한테 업데이트하루 수 있는 기회를 주는 것  
- 사용자가 해당 앱을 자주 사용하는 시간대나 특정 패턴에 의해 자주불리기도, 가끔불리기도하는데 앱에서는 알지 못함
- fetch 후 앱의 활성화 시간은 OS에서 제어 (최대 30초)

### 구현 방법


``` objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
[[UIApplication sharedApplication] setMinimumBackgroundFetchInterval:UIApplicationBackgroundFetchIntervalMinimum];
return YES;
}


-(void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
[API fetchSomeData:^(NSError *aError){
if(aError != nil)
{
completionHandler(UIBackgroundFetchResultNewData);
}
else
{
completionHandler(UIBackgroundFetchResultNoData);
}
}];
}
```

1. capability에서 설정 ON

<img src="/assets/media/iOS/Background3.png">

2. UIApplecation의 setMinimumBackgroundFetchInterval 메서를 통해 최소 시간 지정
- 기본값은 UIApplicationBackgroundFetchIntervalNever 로 지정안하면 백그라운드패치수행 안됨
- 앱에 필요한 최소시간을 지정해줘도 OS에서 알아서 판단해서 수행 


3. 앱델리게이트에  application:performFetchWithCompletionHandler: 구현
- 앱에서 동작 다 하면 completionHandler 호출
- 호출이 너무늦어지면 OS에서 백그라운드패치 호출 주기를 늦출 수 있음
- handler에 전달하는 UIBackgroundFetchResult 값에따라 어떻게 동작하는지 차이점 설명안되어있음

### 디버깅 방법

- OS에서 제어하기때문에 언제 호출될지 몰라 디버깅을위한 방법이 존재
- 스킴 > 옵션에서 셋팅하고 실행하면됨
- 실행후에 다시 호출하고싶다면 상단메뉴 > debug > Simulate Background Fetch 클릭 

<img src="/assets/media/iOS/Background4.png">

<img src="/assets/media/iOS/Background5.png">
