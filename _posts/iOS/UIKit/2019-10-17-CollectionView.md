---
layout: post
title: UICollectionView Custom 
categories: [iOS]
tags: [iOS UIKit]
description: override할 메소드 정리
comments: false
---

- prepareLayout
- 컬렉션뷰 사이즈 가져올 수 있음
- 다음 레이아웃 업데이트를 위한 계산 여기서 가능

- prepareForCollectionViewUpdates:
-  UICollectionViewUpdateItem array를 통해 변경될 레이아웃 어레이를 받아옴
- 삽입,삭제, 이동, 리로드 등을 알려줌


- finalizeCollectionViewUpdates
- 컬렉션 뷰 업데이트 중에 필요한 추가 애니메이션 또는 정리를 수행
- 레이아웃 업데이트 가장 마지막에 호출되는 메소드

- prepareForAnimatedBoundsChange:
- bounds변화, 삽입, 삭제에 대한 계산 준비


- finalizeAnimatedBoundsChange
- 크기변경이나, 삽입삭제 후 초기화를위해 사용

- shouldInvalidateLayoutForBoundsChange:
- 파라미터 boudns영역의 레이아웃이 바뀌어야되면 yes, 아니면 no
- no가 디폴트
- yes이면 invalidLayout호출해서 레이아웃 다시 그리기 내부적으로 진행


---


- initialLayoutAttributesForAppearingItemAtIndexPath
- 삽입할 때 첫 포지션 정보 전달

- finalLayoutAttributesForDisappearingItemAtIndexPath
- prepareForCollectionViewUpdates 호출 후에 불림ㅐ
- 구현 시 항목의 최종 형태 및 목적지에 대해 알려줘야함

- layoutAttributesForItemAtIndexPath
- indexPath의 attributes를  반환
- 이 때도 커스텀하게 계산 가능
- 서플라이, 데코레이션에는 사용하지 마라

- targetContentOffsetForProposedContentOffset
- 스크롤 위치를 반환
- 페이징시 필요할 수도


--- 


- UICollectionViewLayoutAttributes
- 오버라이드할때 카피3개 isEqual, hash, cpoywithzone .. 3개 필수 (찾아보자)
