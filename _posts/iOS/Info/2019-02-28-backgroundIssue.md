---
layout: post
title: Background 유효시간 정리
categories: [iOS]
tags: [iOS 정보]
description: 정리
comments: false

---

# 이슈

## 1. Timer 설정값과 OS > Background 유효시간 값이 다름

<img src="/assets/media/iOS/BackgroundIssue1.png">

### 이슈

- Timer 300초 설정
- 앱이 백그라운드로 진입
- OS에서는 180초의 백그라운드 동작시간만 보장 (OS마다 기기마다 최대시간은 다를 수 있음)
- 백그라운드에서 180초가 지나면 Timer는 120초에서 더이상 가지않고 멈춤
- 다시 앱이 포그라운드로 오면 Timer는 120초부터 시작하지만 실제서버의 유효시간은 끝난 상태

### 원인

- OS에서는제한된  백그라운드실행시간을 줌 
- 백그라운드동작시간이 바코드유효시간보다 짧을 수 있음 
- NSTImer는 invalidate를 호출하지않는 이상 해당 런루프에서 계속실행


```objc
- (void)startTimerWithTarget:(id)aTarget action:(SEL)aAction validTime:(NSInteger)aValidTime
{
mValidTimeSecond = aValidTime;
mRemaingTimeSecond = mValidTimeSecond;

mBgTaskId = [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
[[UIApplication sharedApplication] endBackgroundTask:mBgTaskId];
}];

mTimer = [NSTimer timerWithTimeInterval:1 target:aTarget selector:aAction userInfo:nil repeats:YES];
[[NSRunLoop mainRunLoop] addTimer:mTimer forMode:NSRunLoopCommonModes];
}
```

<img src="/assets/media/iOS/BackgroundIssue2.png">


### 해결 방안

- [[UIApplication sharedApplication] backgroundTimeRemaining]  메소드를통해 남은 시간 가져옴
- 남은시간이 없으면 타이머 종료 / UI 적절한상태로 업데이트?

``` objc
- (BOOL)isValidBackgroundRemainingTime
{
return [[UIApplication sharedApplication] backgroundTimeRemaining] > 10; 
}

- (void)didReceiveTimerEvent:(NSTimer *)aTimer
{
[self.barcodeDataController reduceOneSecond];

if([self isValidBackgroundRemainingTime] == NO)
{
[self executeTimeOutAction];
}
else if([self applicationIsForegroundState])
{
dispatch_async(dispatch_get_main_queue(), ^{
[UIView animateWithDuration:1 delay:0 options:UIViewAnimationOptionCurveLinear animations:^{
[self.certNumberView.progressView setProgress:[self.barcodeDataController currentPercent] animated:YES];
} completion:^(BOOL finished) {
}];
});

[self.certNumberView.timeLabel setText:[self.barcodeDataController remainingTimeText]];

if([self.barcodeDataController isTimeout])
{
[self executeTimeOutAction];
}
}
}
```

