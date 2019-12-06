---
layout: post
title: Story Board
categories: [iOS]
tags: [iOS UIKit]
description: Segue정리 + unwind segue + custom segue 
comments: false

---

### StoryBoard

- Scene과 Segue 2개로 표현
- Scene은 화면을 구성하고 Segue를 통해 화면간 혹은 스토리보드간 이동을 명세한다
- Scene을 구성한는 방법은 너무 많아서 Segue에 대해서만 정리


# Segue

- Scene을 연결하는 객체
- source -> destination  연결
- 이동 시 애니메이션 효과 등 커스텀할 수 있음

### 기본 Segue 종류

- show : push
- show detail : replace
- fullScreen single Viewcontorller에서 사용시 present와 같은 효과
- present modal
- present as popover

## Segue 사용 방법

1. 스토리브도에서 특정 Action을 Segue로 바로 연결
2. 스토리보드에서 Segue만 생성 -> 코드로 호출
- 생성한 Segue의 identifier를 설정하고 이값을 performSegue로 호출해줘야함

3. Segue를 사용하지 않고 StoryBoard의 뷰컨 호출하기

``` swift
let storyboard = UIStoryboard(name: "Main", bundle: nil)
let nextViewController = storyboard.instantiateViewControllerWithIdentifier("NextView") as NextViewController
self.presentViewController(nextViewController, animated: true, completion: nil)
```


## Unwind Segue

[참고](https://medium.com/@kyeahen/ios-unwind-segue-in-swift-e8ff0e7fbbcd)

- Exit와 연결되는 세그
- A -> B -> C 에서 C -> B로의 돌아가기나,   C -> A로 돌아가는 등의 세그 정의 가능
- 사용은 일반 Segue와 똑같지만 생성 방법이 다름

<img src="/assets/media/iOS/storyBaord1.png">
![image.png](/files/2628244829572970086)

## 사용법


- UIVIewController > unwind 메소드 정의 
-  메소드네이밍을 다르게해서 되돌아갈 타겟을 구분할 수 있게 하는게 중요
- 스토리보드에서 Exit로 특정 액션 직접 연결하던가 
- ViewController -> Exit로 드래그하면 정의한 unwind메소드가 나옴
- 스토리보드 > ViewController > Exit > UnwindSegue 정의 > identifier 명시 > 프로그램에서 performSegue로 호출 가능

### 메모

- 같은 이름의 unwind 메소드를 다른 viewController에서 중복된 네이밍으로 생성했따면, 스토리보드에서는 1개의 네이밍만 뜸. 
- 사리지는 viewController와 가장 가까운 presentingVieContorller 기준으로 unwind가 호출됨
- prepareSegue는 호출되지않고 필요하면 unwind 메소드에서 동작하면됨.
- push, present가 섞여도 사용가능

<img src="/assets/media/iOS/storyBaord2.png">


### 코드


``` swift
class RedViewController: UIViewController {
@IBAction func unwindToRed(for unwindSegue: UIStoryboardSegue, towards subsequentVC: UIViewController) {
print("unwind")
}
}


class GreenViewController: UIViewController {
@IBAction func unwindToGreen(for unwindSegue: UIStoryboardSegue, towards subsequentVC: UIViewController) {
print("unwind")
}
}


class BlueViewController: UIViewController {
@IBAction func tappedAction(_ sender: Any) {
performSegue(withIdentifier: "unwindBlueToRed", sender: self)
}
}
```

---

## Custom Segue

- UIStoryboardSegue 상속
- perform method 오벌라이드
- custom transiion도 이런식으로 segue에 감싸서 만들면 좋을 것 같음

ex )

``` objc
- (void)perform
{
[[self sourceViewController] addChildViewController:[self destinationViewController]];
}
```
