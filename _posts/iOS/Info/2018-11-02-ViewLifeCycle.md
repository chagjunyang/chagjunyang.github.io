---
layout: post
title: UIView > layoutSubViews
categories: [iOS]
tags: [iOS 정보]
description: setNeedsLayout vs layoutIfNeed
comments: false

---

[애플.View](https://developer.apple.com/documentation/uikit/uiview)

<img src="/assets/media/iOS/LifeCycle1.png">

## [layoutSubViews](https://developer.apple.com/documentation/uikit/uiview/1622482-layoutsubviews)

- 모든 subView의 layoutSubViews 호출
- 오버라이드해서 나만의 레이아웃을 여기서 동작하게끔 추가 가능
- 직접 호출하지말고 setNeedsLayout호출, 즉시업데이트하려면 layoutIfNeed

### layoutSubViews가 자동으로 불리는 시점

- subView를 추가할 때
- 크기를 조정 했을 때 
- 스크롤했을 때
- 화면이 회전되었을 때
- 제약조건이 변경되었을 떄
- frame기반 작업할때 여기서 포지션 잡았음

### setNeedLayout

- 다음번 update cycle에서 layoutSubViews가 호출되게 함. 
- 비용적으로 가장 효율

### layoutIfNeed

- 호출 즉시 layoutSubViews가 동작되게 수행
- 즉시 값이 변경되어야흐는 애니메이션에서 자주 사용

### updateConstraints

- layoutIfNeed와 마찬가지로 직접호출하면 안됨
- 제약조건이 옳바르게 들어갔는지 확인 가능한 부분

### setNeedsUpdateConstraints

- 다음 사이클에 updateConstraints를 호출

### updateConstraintsIfNeeded

- 호출즉시 updateConstraints 호출
- 이 메소드는 오버라이드하지 마세요
- 시스템에 의해 자동으로 불립니다. 



