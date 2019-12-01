---
layout: post
title: UIResponder
categories: [iOS]
tags: [iOS 정보]
description: Responder Chain 구조 파악
comments: false

---

[참고](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events?language=objc)


- 앱은 UIResponder 객체를 통헤 이벤트를 받는다.
- UIResponder객체는 UIVIew, UIViewController , UIResponder, UIApplication 등 다양하다
- responder는 이벤트를 받으면 이벤트를 처리하던가 다른 responder에게 전파해야만 한다
- 앱이이벤트를 받으면 UIkit은 자동적으로 firstResponder에게 이벤트를 전달한다
- 처리하지 않은 이벤트들은 responder chain의 다음으로 전파된다.

<img src="/assets/media/iOS/UIResponder1.png">

- 제스쳐레코그나이져는 터치나 프레스 이벤트를 뷰보다 먼저 받는다.
- 제스쳐레코그나이져가 처리를 안하면 UIKit은 뷰에게 전파한다.
- 해당뷰가 이벤트처리를안하면 위에 언급한것처럼 리스폰터체이닝시작

#### 어떤 리스폰더가 터치이벤트를 받을지 결정하는 로직

- UIKit은 view-based hit testing을 통해 어디서 터치이벤트가 발생했는지 결정
- 뷰계층구조속의 영역을 계산
- hitTest:withEvent:  메세지를 뷰에게 보냅니다.
- 이런식으로 지정된 터치를 포함하는 가장 깊은 하위 뷰를 찾고 뷰 계층을 탐색합니다. 이 하위 뷰는 터치 이벤트의 첫 번째 응답자가됩니다.

### 리스폰더 체인 변경

- nextResponder속성을 변경하여 체인구조변경가능
- UIVIew, UIViewContorller 등 많은 오브젝트에 있음 


## 테스트


### CASE : UIBUtton -> UIBUtton

<img src="/assets/media/iOS/UIResponder2.png">

- 초록색선택 : 최상위 버튼 한군대에만 hitTest하고 바로 종료
- 빨간색 선택 : 초록색, 빨간색 순으로 힛테스트 / 초록색버튼에서는 힛테스트에서 nil을 반환
- 이런식으로 위에서부터 체이닝

### CASE : gesture -> gesture

### CASE : gesture -> UIBUtton

- 버튼의 힛테스트가 불리고 제스쳐 불림
- 제스쳐가 성공하면 버튼의 액션은 동작안함
- 제스쳐가 실패하면 버튼의 액션 동작
- 버튼위로도 제스처는 동립적으로 받는다.
### CASE : gesture -> gesture -> UIBUtton

### CASE : 버튼에 제스쳐를 입히면?
