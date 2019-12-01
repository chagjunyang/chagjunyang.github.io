---
layout: post
title: WKWebView
categories: [iOS]
tags: [iOS UIKit]
description: UIWebView vs WKWebview
comments: false

---

# UIWebView .vs. WKWebView

<img src="/assets/media/iOS/WKWebView11.png">

<img src="/assets/media/iOS/WKWebView2.png">

<img src="/assets/media/iOS/WKWebView3.png">

<img src="/assets/media/iOS/WKWebView4.png">

<img src="/assets/media/iOS/WKWebView5.png">


# WKWebView 분석


### init

```objc

- (instancetype)initWithFrame:(CGRect)frame configuration:(WKWebViewConfiguration *)configuration NS_DESIGNATED_INITIALIZER;
- (nullable instancetype)initWithCoder:(NSCoder *)coder NS_DESIGNATED_INITIALIZER;

```

### load & stop

``` objc

@property (nonatomic, readonly, getter=isLoading) BOOL loading;

- (nullable WKNavigation *)loadRequest:(NSURLRequest *)request;
- (nullable WKNavigation *)loadFileURL:(NSURL *)URL allowingReadAccessToURL:(NSURL *)readAccessURL API_AVAILABLE(macosx(10.11), ios(9.0));
- (nullable WKNavigation *)loadHTMLString:(NSString *)string baseURL:(nullable NSURL *)baseURL;
- (nullable WKNavigation *)loadData:(NSData *)data MIMEType:(NSString *)MIMEType characterEncodingName:(NSString *)characterEncodingName baseURL:(NSURL *)baseURL API_AVAILABLE(macosx(10.11), ios(9.0));

- (nullable WKNavigation *)reload;
- (nullable WKNavigation *)reloadFromOrigin; //캐싱된 정보를 기준으로 리로드할지 판단하여 리로드

- (void)stopLoading;
```

### navigation 

``` objc
//네비게이션의 완료도. 0부터 1의 값을 가짐.
@property (nonatomic, readonly) double estimatedProgress;

@property (nonatomic, readonly) BOOL canGoBack;
@property (nonatomic, readonly) BOOL canGoForward;

@property (nonatomic) BOOL allowsBackForwardNavigationGestures; //YES일경우 엣지스와이프액션으로 goBack, goForward동작가능
@property (nonatomic) BOOL allowsLinkPreview API_AVAILABLE(macosx(10.11), ios(9.0)); //3D Touch시 프리뷰 화면 제공

- (nullable WKNavigation *)goBack;
- (nullable WKNavigation *)goForward;

```


### scale


``` objc

@property (nonatomic) BOOL allowsMagnification;
@property (nonatomic) CGFloat magnification;

- (void)setMagnification:(CGFloat)magnification centeredAtPoint:(CGPoint)point;

```


# WKNavigationDelegate 분석


``` objc
optional

//1. Decides whether to allow or cancel a navigation. (shouldRequest)
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler;

//4. Decides whether to allow or cancel a navigation after its response is known.
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler;

//2. Called when web content begins to load in a web view.  (didStartLoad)
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(null_unspecified WKNavigation *)navigation;

//3. Called when a web view receives a server redirect.
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(null_unspecified WKNavigation *)navigation;

// Called when an error occurs while the web view is loading content.
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error;

//5. Called when the web view begins to receive web content.
- (void)webView:(WKWebView *)webView didCommitNavigation:(null_unspecified WKNavigation *)navigation;

//6. Called when the navigation is complete. (didFinishLoad)
- (void)webView:(WKWebView *)webView didFinishNavigation:(null_unspecified WKNavigation *)navigation;

// Called when an error occurs during navigation.
- (void)webView:(WKWebView *)webView didFailNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error;

```


## 델리게이트 호출 순서

### 최초 화면 진입

```
2017-05-29 17:53:05.592642+0900 PAYCO[18647:3550727] decidePolicyForNavigationAction
2017-05-29 17:53:05.594526+0900 PAYCO[18647:3550727] didStartProvisionalNavigation
2017-05-29 17:53:05.695049+0900 PAYCO[18647:3550727] decidePolicyForNavigationAction
2017-05-29 17:53:05.698461+0900 PAYCO[18647:3550727] didReceiveServerRedirectForProvisionalNavigation
2017-05-29 17:53:05.779858+0900 PAYCO[18647:3550727] decidePolicyForNavigationResponse
2017-05-29 17:53:05.788819+0900 PAYCO[18647:3550727] didCommitNavigation
2017-05-29 17:53:06.605984+0900 PAYCO[18647:3550727] didFinishNavigation


```


### 화면 > 화면 이동


```
2017-05-29 14:13:08.213894+0900 PAYCO[18563:3537307] decidePolicyForNavigationAction
2017-05-29 14:13:08.220277+0900 PAYCO[18563:3537307] didStartProvisionalNavigation
2017-05-29 14:13:08.311146+0900 PAYCO[18563:3537307] decidePolicyForNavigationResponse
2017-05-29 14:13:08.324191+0900 PAYCO[18563:3537307] didCommitNavigation
2017-05-29 14:13:08.622047+0900 PAYCO[18563:3537307] didFinishNavigation

```


###  화면 < 화면 이동 (히스토리백)

```
2017-05-29 14:14:13.655658+0900 PAYCO[18563:3537307] decidePolicyForNavigationAction
2017-05-29 14:14:13.661791+0900 PAYCO[18563:3537307] didStartProvisionalNavigation
2017-05-29 14:14:13.672985+0900 PAYCO[18563:3537307] didCommitNavigation
2017-05-29 14:14:13.712415+0900 PAYCO[18563:3537307] didFailNavigation

```

# WKUIDelegate 분석

``` objc

//웹페이지 -> 새로운 페이지 생성 (window.open()) 시 호출. 구현 안하면 웹페이지 동작 x
- (nullable WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures;

- (void)webViewDidClose:(WKWebView *)webView 

- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler;

- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL result))completionHandler;

- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(nullable NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable result))completionHandler;


```

### 샘플코드
``` objc
#pragma mark - WKUIDelegate

//window.alert(), alert() ...
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler
{
if (completionHandler)
{
UIAlertController *alertController = [UIAlertController alertControllerWithTitle:nil message:message preferredStyle:UIAlertControllerStyleAlert];

[alertController addAction:[UIAlertAction actionWithTitle:NSLocalizedString(@"확인", @"")
style:UIAlertActionStyleDefault
handler:^(UIAlertAction * _Nonnull action) {
completionHandler();
}]];

[self presentViewController:alertController animated:YES completion:nil];
}
}

//confirm()
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL))completionHandler
{
if (completionHandler)
{
UIAlertController *alertController = [UIAlertController alertControllerWithTitle:nil message:message preferredStyle:UIAlertControllerStyleAlert];

[alertController addAction:[UIAlertAction actionWithTitle:NSLocalizedString(@"확인", @"")
style:UIAlertActionStyleDefault
handler:^(UIAlertAction * _Nonnull action) {
completionHandler(YES);
}]];

[alertController addAction:[UIAlertAction actionWithTitle:NSLocalizedString(@"취소", @"")
style:UIAlertActionStyleCancel
handler:^(UIAlertAction * _Nonnull action) {
completionHandler(NO);
}]];

[self presentViewController:alertController animated:YES completion:nil];
}

}

//window.open()
- (WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures
{
WKWebView *sWebView = [[WKWebView alloc] initWithFrame:webView.frame configuration:configuration];
[sWebView setUIDelegate:self];
[sWebView setNavigationDelegate:self];
[sWebView setAutoresizingMask:(UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight)];

[self.view addSubview:sWebView];

return sWebView;
}

//window.close()
- (void)webViewDidClose:(WKWebView *)webView
{
[webView removeFromSuperview];

webView = nil;
}
```

# 자바스크립트 실행 방법

### 1. webview에 script 실행 방식

``` objc
- (void)evalTest
{
[self.webView evaluateJavaScript:@"document.title" completionHandler:^(id _Nullable result, NSError * _Nullable error) {
NSLog(@"result is %@", result);
}];
}
```

### 2. WKWebViewConfiguration을 이용한 방식 (웹과 네이티브 통신)


#### objective - c


``` objc
- (void)setupWebView
{
WKWebViewConfiguration *sConfiguration  = [[WKWebViewConfiguration alloc] init];
WKUserContentController *sController    = [[WKUserContentController alloc] init];
NSString *scriptSource = @"window.webkit.messageHandlers.observe.postMessage('test')";

WKUserScript *sUserScript = [[WKUserScript alloc]
initWithSource:scriptSource
injectionTime:WKUserScriptInjectionTimeAtDocumentStart
forMainFrameOnly:YES];

[sController addScriptMessageHandler:self name:@"observe"];
[sController addUserScript:sUserScript];

[sConfiguration setUserContentController:sController];

self.webView = [[WKWebView alloc] initWithFrame:self.view.frame configuration:sConfiguration];

[self.webView setNavigationDelegate:self];
[self.webView setUIDelegate:self];
[self.webView setAllowsLinkPreview:YES];
[self.webView setAllowsBackForwardNavigationGestures:YES];

[self.view addSubview:self.webView];

NSURLRequest *sRequest = [[NSURLRequest alloc] initWithURL:[NSURL URLWithString:@"http://10.77.108.24:8080"]];

[self.webView loadRequest:sRequest];
}


#pragma mark - WKScriptMessageHandler


- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
NSLog(@"message is %@", message.body);
}
```

#### web page

``` javascript
$btn1.click(function(e) {
window.webkit.messageHandlers.observe.postMessage('test'); 
});
```



### WKWebViewConfiguration


-  A collection of properties used to initialize a web view.


### WKUserContentController

- A WKUserContentController object provides a way for JavaScript to post messages and inject user scripts to a web view.


### WKScriptMessageHandler

- A class conforming to the WKScriptMessageHandler protocol provides a method for receiving messages from JavaScript running in a webpage.

### WKScriptMessage

- A WKScriptMessage object contains information about a message sent from a webpage.

### WKUserScript

- A WKUserScript object represents a script that can be injected into a webpage.


### 3. 결론

- 웹뷰 로드 후 특정 스크립트를 웹페이지에 실행시키고 싶다면 eval방식 사용
- 웹뷰 로드 전,후 바로 수행될 특정 스크립트가 있거나, 웹뷰에서 앱으로 전달되는 특정 약속들은 2번 방식 사용.
