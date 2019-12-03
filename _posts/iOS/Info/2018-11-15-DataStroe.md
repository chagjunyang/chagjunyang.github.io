---
layout: post
title: iOS 저장공간
categories: [iOS]
tags: [iOS 저장소]
description: SQLite, PList, CoreData, NSUserDefaults, KeyChain...
comments: false

---

[파일 시스템]()


## NSUserDefaults

- 로컬 데이터 저장을 위한 가장 일반적인 방법 중 하나
- 앱 삭제시 같이 삭제됨
- 저장될 기본객체는 Plist와 같음
- 커스텀 모델같은 데이터를 넣고싶으면 NSData로 아카이브해서 저장
- NSUserDefaultsDidChangeNotification, key-value observing 등을 통해 값 변화를 알 수 있음
- 쓰레드 세이프함

---

## Plist

- propertyList
- NSArray, NSDate, NSString, NSDictionary 등 사용 가능
- Boolean, Integer 는 number로, 운수도원에서는 string으로 했음
- key-value list가 나열된 파일로 아카이브되어 저장


---

## CachedData (NSCache)

- file system중 캐시 디렉토리에 접근해서 사용
- NSSearchPathForDirectoriesInDomains(NSCachesDirectory)

### NSDocumentDirectory

- 아이튠즈, 아이클라우드에서 백업 가능 


### NSLibraryDirectory

- 아이튠즈, 아이클라우드에 공유 불가

### NSCachesDirectory

- 캐시용 디렉토리
- 메모리가 부족하면 제거될 수 있음
- 아이튠즈, 아이클라우드에 공유 불가

---

## KeyChain

[링크](http://chagjunyang.github.io/ios/2018/11/15/KeyChain.html))

---

## CoreData

[링크](http://chagjunyang.github.io/ios/2018/11/15/CoreData.html)

---

## 저장소 선택

- 비밀번호와같이 민감한 정보는 KeyChain
- 사용자 설정값과 같은 정보는 UserDefaults
- 적은 텍스트, 숫자, 배열과 같은 데이터는 UserDefault
- 큰 데이터와 복잡한 모델은 CoreData
