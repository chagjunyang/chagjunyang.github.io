---
layout: post
title: Voice Over
categories: [iOS]
tags: [iOS 정보]
description: 사용법 정리
comments: false
---

# Voice Over

* 애플에서 제공하는 Accessibility 중 하나.
* iOS 10 대 이상에서는 로터라는 것도 제공. 두 손가락 회전 시 나타남
* https://www.apple.com/kr/accessibility/iphone/vision/


## Accessibility API

* lightweight API
* UIKit framework에 포함되어 있음.

### UIAccessibility Protocol

* UIControl이랑 UIView에 기본으로 구현되어있다.
->> UIControl이랑 UIView에서만 사용할 수 있음
* label, frame, traits 등의 속성이 선언되어 있다.
* 그 외 오버라이드 메소드 들도 선언되어 있다. 
* UIAccessibility.h


### UIAccessibilityContainer Protocol

* UIControl나 UIView가 아닌, customView일 경우, Accessibility를 제공하기 위해 사용.(text나 도형, 이미지 draw 하는 경우)

### UIAccessibilityElement

* UIAccessibilityContainer Protocal 통해서만 return 가능
* customView를 표현하기 위해 생성
* 이를 사용해 UIView를 상속받지 않은 객체가 accessability를 지원하는 것도 가능하다.


### UIAccessibilityConstants.h

* Traits, accessibility 관련 노티피케이션이 정의되어 있는 헤더파일.
* 노티피케이션들을 이용해, 슬라이더의 값, 프로그레스 바의 변경 상태 등을 알려줄 수도 있다.


## 속성

<img src="/assets/media/iOS/VoiceOver1.png">

### 읽는 순서

label -> value -> trait -> hint 순으로 읽어줍니다. 각 사이에 1-2초간 텀이 좀 있는 편이니, 다음 속성을 들으시려면 조금 기다려야 합니다.

### 대표 속성들 (prefix인 accessibility는 생략)

<img src="/assets/media/iOS/VoiceOver2.png">

### 일반적인 Traits의 종류

UIView의 경우는 None이 기본값이다.
UILabel, UIButton, UISegmentedControl 등 일부 UI Components와 그 상속자들은 이미 Trait에 기본값이 들어가 있다.

* Button
* Link
* Search Field
* Keyboard Key
* Static Text
* Image
* Plays Sound
* Selected
* Summary Element
* Updates Frequently
* Not Enabled
* None(UIView의 기본값)

만약 커스텀 뷰라면, `accessibilityTraits` 메소드를 오버라이드 하거나 `setAccessibilityTraits`메소드로 원하는 UIAccessibilityTraits을 반환하는 것도 가능하다.

https://developer.apple.com/documentation/uikit/accessibility/uiaccessibility/accessibility_traits?language=objc

## UIKit에서 지원하는 기본 구현

### 기본값 isAccessibilityElement = NO 인 클래스들

* UIView
* UIImageView
* UIScrollView
* UITableView
* UICollectionView
* 아래 명기된 단일 컴포넌트 외 대부분 (전부 확인은 아직 못해봤습니다...)

### UIKit 내의 단일 컴포넌트들의 기본값

<img src="/assets/media/iOS/VoiceOver3.png">

### UIKit 내의 컨테이너(UITableView, UICollectionView 등)의 기본 동작

<img src="/assets/media/iOS/VoiceOver4.png">

## 구현 하기

* UIKit의 많은 컴포넌트들이 이미 구현하고 있음. (적어도 UILabel, UIButton은 신경을 거의 안 써도 됨)
* 기본적인 동작들은 위의 명기된 속성들을 설정하는 것 만으로도 가능!
* isAccessibilityElement = YES 



### 구현 예제

아래 두 예제는 같은 효과를 보입니다!


#### 뷰 내에서 오버라이드

```objc
#pragma mark - accessibility


- (BOOL)isAccessibilityElement
{
    return YES;
}


- (NSString *)accessibilityLabel
{
    return [NSString stringWithFormat:@"%@, %@", _storeNameLabel.text, _storeTypeLabel.text];
}


```



#### 뷰 외부에서 설정

```objc
@implementation NPStoreHomeStoreListCellModel

//중략

- (void)bindCellModelWithCell:(UICollectionViewCell<NPStoreHomeStoreListCellInterface> *)aCell
{
    NPStoreHomeStoreListCell *sCell = (NPStoreHomeStoreListCell *)aCell;
    
    //중략
    
    //setting accessibility
    
    [sCell setIsAccessibilityElement:YES];
    [sCell setAccessibilityLabel:[NSString stringWithFormat:@"%@, %@", _storeName, _storeTypeName]];
    
}


@end
```

## 디버깅 하기


### Voice Over 디버깅 하기

<span style="color:#e11d21">**주의 : Voice Over는 오직 실 기기에서만 실행 가능하다**</span>

#### Voice Over 켜기


##### 모든 상황에서 Voice Over 켜기
* 설정 > 일반 > 손쉬운 사용 > Voice Over 켜기

##### 손쉬운 사용 단축키 이용하기 (추천)

* 설정 > 일반 > 손쉬운 사용 > (쭉 아래로 스크롤) 손쉬운 사용 단축키 > Voice Over 선택
* 설정 이후, 홈 버튼을 3번 탭하면 Voice Over를 On/Off 할 수 있음
* **주의** 기본값은 지정 단축키가 없음이다. 단축키를 사용해서 Voice Over를 사용하려면 직접 단축키에서 지정해줘야 한다.


##### 참고 링크
https://support.apple.com/ko-kr/HT204390


#### Voice Over 동작

##### 손쉬운 사용에서 Voice Over를 선택했을 경우 On/Off

* 아이폰 X 이외의 기기에서는 홈 버튼 3번 탭
* 아이폰 X는 측면 버튼 3번 탭

##### 탭 동작

* 한 번 탭 : 선택
* 두번 탭 : **선택된 뷰에 한해서** 일반적인 탭 동작 (너무 많이 누르면 그것도 안됨)
* 세 손가락 세번 탭 : 화면 커튼 (화면 끄기) On/Off
* 세 손가락 2번 탭 : 말하기 On/Off (효과음만 들림)
* 두 손가락 3번 탭 : 화면 상의 모든 accessibilityElement의 리스트. accessiibilityLabel이 설정되어 있는 경우에만 리스트에 노출됨.
* 두 손가락 2번 탭 : 매직 탭. 앱의 구현에 따라 동작이 다릅니다. 보통 숏컷으로 사용하거나, 앱의 상태를 변경하는 데 사용합니다. (카메라 - 사진 촬영)

##### 스와이프 동작

* 세 손가락 스와이프 - 스크롤 동작. 스크롤 하는 손가락이 해당 뷰 내에 있어야 함 + 선택되어 있어야 함. 스크롤 될 뷰가 너무 작으면 잘 안됨...ㅠㅠㅠㅠㅠ (ex 가맹점 홈)
* 한 손가락 스와이프 - 다음 뷰 읽음 (보통 상-하, 좌-우 순으로 읽는 거 같은데.... 조금 더 알아보겠습니다. 특정 순서가 있는 듯 합니다.)



#### Voice Over Curtain

실제로 화면을 꺼서, 보이스 오버만 사용해서 앱을 사용할 수 있게 해주는 기능.

Voice Over를 사용 하는 상태에서, 세 손가락으로 세번 화면을 탭하면 on/off 할 수 있다.


### 시뮬레이터

<span style="color:#e11d21">**시뮬레이터에서는 Voice Over를 테스트할 수 없다.**</span>
대신에, Accessibility Inspector라는 것을 사용해서 보이스 오버를 제외한 전반적인 분석이 가능하다.

#### Accessbility Inspector

Xcode > Open Developer Tool > Accessibility Inspector 로 접근 가능하다.

* 특정한 뷰의 Accessibility 관련 프로퍼티들을 볼 수 있다.
* audit (심사) 기능으로 전반적인 분석도 가능하다. 안타깝게도 보이스 오버로 어떻게 들리는지 알 수 가 없다!
* 그래도 Voice Over가 인식하는 frame은 테스트가 가능하다.
* 글자 크기, 투명도 제거 등의 시각적인 테스트는 가능하다.

<img src="/assets/media/iOS/VoiceOver5.png">
<img src="/assets/media/iOS/VoiceOver6.png">


## 이어폰에서만 소리나게 하기


### 기본 동작
* 스피커와 이어폰의 볼륨 수준, 음소거 여부는 분리가 되어 있음. (스피커 음량 != 이어폰 음량)
* 이어폰이 꽂혀있을 경우, 이어폰에서만 나고 있음. 이어폰이 뽑혔을 때, 스피커 음량으로 전환
* 이어폰에서만 나야하는 경우는 오디오 출력 기기 확인 후, 안내, 출력 소스 전환이 필요할 것 같음. (예를 들어 결제 비밀번호 입력)

### 오디오 출력 기기 확인

AVAudioSession의 currentRoute를 이용해서 확인 가능합니다.
https://developer.apple.com/documentation/avfoundation/avaudiosession?language=objc

##### 오디오 출력 기기가 헤드폰일 경우에만 내부 원소에 접근 가능하게 하기 

```objc
#import <AVFoundation/AVFoundation.h> //필수

//NPCommonPasswordConfirmKeyPadView.m

AVAudioSession *sSession = [AVAudioSession sharedInstance];
NSError *sError;


@try {
    
    [sSession setActive:YES error:&sError]; //가져온 세션 활성화
    
    BOOL isSoundsPlaysThroughSpeaker = NO;
    
    for (AVAudioSessionPortDescription *sDesc in sSession.currentRoute.outputs) // 현재 오디오 세션에 동작하고 있는 모든 출력 포트를 가져온다.
    {
        if ([sDesc.portType isEqualToString:AVAudioSessionPortHeadphones] == NO) //헤드폰이 아니라면 다 스피커...
        {
            isSoundsPlaysThroughSpeaker = YES;
        }
        
    }
    
    if (isSoundsPlaysThroughSpeaker)
    {
        UIAccessibilityPostNotification(UIAccessibilityAnnouncementNotification, @"스피커로 소리가 재생되고 있습니다. 보안을 위해 반드시 헤드셋을 착용해 주세요.");
        
    }
    else
    {
        
        [self setIsAccessibilityElement:YES];
        UIAccessibilityPostNotification(UIAccessibilityScreenChangedNotification,  self);
    }
    
    
} @catch (NSException *exception) {
    
} @finally {
    
}

//후략
```


상수들.
AVAudioSessionPortHeadphones //헤드폰
AVAudioSessionPortBuiltInSpeaker //일반 스피커
AVAudioSessionPortBuiltInReceiver //가장 사용자에게 가까운 스피커. 전화 받을 때 쓰임.


## 참고 문서

* 애플의 가이드
https://developer.apple.com/accessibility/ios/

* 애플의 뷰컨트롤러 작성 시 접근성 적용 가이드
https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/SupportingAccessibility.html#//apple_ref/doc/uid/TP40007457-CH12-SW1

* capital one이라는 앱에서 voice over를 적용한 사례 
https://medium.com/capital-one-developers/ios-accessibility-best-practices-for-the-voiceover-user-experience-dc08112ef16

* 시뮬 / 기기에서 테스트하기 
https://developer.apple.com/library/content/technotes/TestingAccessibilityOfiOSApps/TestingtheAccessibilityofiOSApps/TestingtheAccessibilityofiOSApps.html#//apple_ref/doc/uid/TP40012619-CH2-SW2

* iOS Accessibility Introduction
https://www.slideshare.net/aimeemaree/the-good-the-bad-the-voiceover-ios-accessiblity
https://revealapp.com/blog/exploring-accessibility.html

* 국내 모바일 접근성 2.0 지침에 따른 iOS 가이드
http://www.wah.or.kr/_Upload/etc/20160323%20IOS%20모바일%20애플리케이션%20접근성%20제작기법.pdf

## 참고하면 좋은 영상들

### WWDC 접근성 관련 영상 (사파리만 재생 가능)

* 애플에서 제공하는 전반적인 접근성 설명과 실제 적용 예제
https://developer.apple.com/videos/play/wwdc2015/201/
