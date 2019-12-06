---
layout: post
title: FileSystem
categories: [iOS]
tags: [iOS 저장소]
description: 어디에 어떤 데이터를 저장해야하나
comments: false

---

[File System Programming Guide](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html)

[참고](https://medium.com/@Alpaca_iOSStudy/ios-파일-시스템-35e61a85a3f8)


### APFS(Apple File System)

- 디스크 파일 시스템의 
- 기존 HFS+라는 파일시스템에서 iOS10.3부터 APFS사용
-  플래시 메모리에 최적화, 암호화 전반적 지원 등 발전된 파일 시스템


### iOS Standard Directories

- 보안상의 이유로 파일 시스템은 앱의 샌드박스 디렉토리로 제한
- 샌드박스 안에 각 컨테이너는 목적에 따라 역할이 있음
- 앱번들은 앱정보만 가지며, 데이터는 디렉토리는 하위 디렉토리로 나뉠 수 있음

<img src="/assets/media/iOS/FileSystem1.png">

- 앱은 컨테이너 외부에있는 파일에 엑세스할 수 없음
- 예외로 연락처와 같이 공개 시스템 인터페이스를 사용하여 접근하는경우가 있음 



| Directory         | Description                                                  |
| :---------------- | :----------------------------------------------------------- |
| *AppName*`.app`   | This is the app’s bundle. This directory contains the app and all of its resources.You cannot write to this directory. To prevent tampering, the bundle directory is signed at installation time. Writing to this directory changes the signature and prevents your app from launching. You can, however, gain read-only access to any resources stored in the apps bundle. For more information, see the *[Resource Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/LoadingResources/Introduction/Introduction.html#//apple_ref/doc/uid/10000051i)*The contents of this directory are not backed up by iTunes or iCloud. However, iTunes does perform an initial sync of any apps purchased from the App Store. |
| `Documents/`      | Use this directory to store user-generated content. The contents of this directory can be made available to the user through file sharing; therefore, this directory should only contain files that you may wish to expose to the user.The contents of this directory are backed up by iTunes and iCloud. |
| `Documents/Inbox` | Use this directory to access files that your app was asked to open by outside entities. Specifically, the Mail program places email attachments associated with your app in this directory. Document interaction controllers may also place files in it.Your app can read and delete files in this directory but cannot create new files or write to existing files. If the user tries to edit a file in this directory, your app must silently move it out of the directory before making any changes.The contents of this directory are backed up by iTunes and iCloud. |
| `Library/`        | This is the top-level directory for any files that are not user data files. You typically put files in one of several standard subdirectories. iOS apps commonly use the `Application Support` and `Caches` subdirectories; however, you can create custom subdirectories.Use the `Library` subdirectories for any files you don’t want exposed to the user. Your app should not use these directories for user data files.The contents of the `Library` directory (with the exception of the `Caches` subdirectory) are backed up by iTunes and iCloud.For additional information about the Library directory and its commonly used subdirectories, see [The Library Directory Stores App-Specific Files](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html#//apple_ref/doc/uid/TP40010672-CH2-SW1). |
| `tmp/`            | Use this directory to write temporary files that do not need to persist between launches of your app. Your app should remove files from this directory when they are no longer needed; however, the system may purge this directory when your app is not running.The contents of this directory are not backed up by iTunes or iCloud. |



### Library

- Application Support: 사용자의 documents를 제외한 나머지 앱의 데이터 파일들을 저장하는데 사용한다. 앱이 생성하고 관리하는 데이터, 설정, 리소스 등이 저장되며 이 디렉토리의 모든 컨텐트들은 앱의 bundle identifier 나 회사 이름의 서브 디렉토리에 위치한다. iTunes와 iCloud에 백업이 가능하다.
- Caches: 앱이 쉽게 재생성 할 수 있는 파일, 쉽게 다운로드 받을 수 있는 파일들이 저장된다. 앱의 성능을 위한 목적으로 존재하며 캐시로는 데이터 베이스 파일이나 다운로드 파일 등이 고려된다. 앱이 실행 중에는 삭제되지 않는 것이 보장되며 백업은 되지 않는다. Caches/Snapshots 디렉토리에는 앱이 백그라운드로 넘어갈 때 저장되는 앱의 스냅샷이 저장된다.
- Preferences: 앱의 중요 설정이 담겨 있는 디렉토리로 NSUserDefaults 클래스를 사용해 파일을 만들어 저장할 수 있다. 이 유저디폴트 파일은 iTunes와 iCloud에 저장이 된다.


### 어디에 앱의 파일을 넣어야 할까?
- 유저 데이터는 Documents 에 저장.
- 앱 생성 데이터는 Library/Application support 에 저장.
- 용량이 큰 파일(예: 미디어 파일) 같은 경우 NSURLIsExcludedFromBackupKey 를NSURL 인스턴스의 메소드인setResourceValue(_:forKey:) 에 전달해 백업이 되지 않도록 한다.
- 임시 파일은 tmp 에 저장.
- 데이터 캐시 파일은 Library/Caches 에 저장

```
Where You Should Put Your App’s Files
To prevent the syncing and backup processes on iOS devices from taking a long time, be selective about where you place files. Apps that store large files can slow down the process of backing up to iTunes or iCloud. These apps can also consume a large amount of a user's available storage, which may encourage the user to delete the app or disable backup of that app's data to iCloud. With this in mind, you should store app data according to the following guidelines:

Put user data in Documents/. User data generally includes any files you might want to expose to the user—anything you might want the user to create, import, delete or edit. For a drawing app, user data includes any graphic files the user might create. For a text editor, it includes the text files. Video and audio apps may even include files that the user has downloaded to watch or listen to later.
Put app-created support files in the Library/Application support/ directory. In general, this directory includes files that the app uses to run but that should remain hidden from the user. This directory can also include data files, configuration files, templates and modified versions of resources loaded from the app bundle.
Remember that files in Documents/ and Application Support/ are backed up by default. You can exclude files from the backup by calling -[NSURL setResourceValue:forKey:error:] using the NSURLIsExcludedFromBackupKey key. Any file that can be re-created or downloaded must be excluded from the backup. This is particularly important for large media files. If your application downloads video or audio files, make sure they are not included in the backup.
Put temporary data in the tmp/ directory. Temporary data comprises any data that you do not need to persist for an extended period of time. Remember to delete those files when you are done with them so that they do not continue to consume space on the user’s device. The system will periodically purge these files when your app is not running; therefore, you cannot rely on these files persisting after your app terminates.
Put data cache files in the Library/Caches/ directory. Cache data can be used for any data that needs to persist longer than temporary data, but not as long as a support file. Generally speaking, the application does not require cache data to operate properly, but it can use cache data to improve performance. Examples of cache data include (but are not limited to) database cache files and transient, downloadable content. Note that the system may delete the Caches/ directory to free up disk space, so your app must be able to re-create or download these files as needed.
```
