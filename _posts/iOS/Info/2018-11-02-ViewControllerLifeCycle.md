---
layout: post
title: ViewController Life Cycle
categories: [iOS]
tags: [iOS 정보]
description: 메모용
comments: false

---

[참고](https://baked-corn.tistory.com/32)


### 1. init(coder:)

- storyboard 를 통해 View Controller들을 만들 경우 호출
- 객체를 Byte Stream으로 바꾸어 디스크에 저장하거나 네트워크를 통해 전송하는 직렬화 작업을 하지 않는 이상 매개변수로 넘어오는 NSCoder 는 무시하셔도 무방
-  아직 이 시점에서는 View Controller의 View가 생성된 것이 아니기 때문에 View의 요소들에 대한 접근을 시도한다면 에러를 발생

### 2. init(nibName:bundle:)
- storyboard가 아닌 분뢰된 .nib파일로 관리될 경우 init(coder:)대신 이 메소드를 초기화의 용도로 사용

### 3. loadView()

- 본격적으로 화면에 띄어질 View를 만드는 메소드
- storyboard 나 .nib파일로 만들어지는 경우가 아닌 모두 직접적으로 코딩하여 만드는 경우를 제외하고서는 override하지 않는 것이 좋음
- outlet들과 action들이 이 메소드에서 생성되고 연결됩니다.

### 4. viewDidLoad()

### 5. viewWillAppear()

### 6. viewDidAppear(_:)

### 7. didReceiveMemoryWarning()

이건 다른 두레이에 따로 정리

### 8. viewWillDisappear()


### 9. viewDidDisappear()

### 10. deinit()
