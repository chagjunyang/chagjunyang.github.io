---
layout: post
title: OAuth 인증
categories: [개인]
tags: [정보]
description: OAuth Test
comments: true
---

# OAuth

## 개념

- 특정 사이트에 권한만 주어 ID/PWD를 직접 사이트에서 입력하지 않고 해당 인증기관을 거쳐 권한만 주기위함
- 이런 작업을 표준화 하고 싶어 생긴 것 이 OAuth (비밀번호 인증방식의 문제점 해결하기 위함)

<img src="/assets/media/oauth_1.png">


- Resource Server (Service Provider)
	- 자원(protected resource)를 제공하는 서버다. Client는 access token을 이용해서 자원에 접근할 수 있다.

- Resource Owner
	- Resource server에 있는 자원의 소유자. 사람이 소유자가 될 필요는 없다. 애플리케이션이나 기계가 자원의 소유자가 될 수도 있다.

- Client
	- Cleint는 유저 계정에 접근을 시도하는 애플리케이션이다. 애플리케이션의 사용을 위해서 로그인이 필요한 경우도 있다.

- Authorization Server
	- 인증을 수행하는 서버로 로그인 및 접근허가 여부를 검사해서 Authorization code과 access token을 발행하는 일을 한다..


- Protected resource
	- 접근허가를 필요로 하는 자원이다. 자원에 접근하기 위해서는 Authorization server를 통해서 access token을 받아야 한다. 자원은 접근제어를 필요로하는 모든 인터넷 객체다. 즉 문서파일, 음악, 오디오, 사용자 정보, API들이다.

- Authorization grant
	- Authorization grant는 client가 획득한 access token에 부여된 리소스 소유자의 권한을 의미한다.

- Authorization code
	- Authorization code는 authorization server가 발급해주는 문자열로 client와 resource owner를 중개하기 위해서 사용한다. Client는 Authorization code를 이용해서 Access token을 얻을 수 있다.

- Access token
	- Access token은 자원(protected resources)에 접근하기 위해서 사용하는 증명서로 접근을 요청하는 client의 권한과 범위(scope)를 알아낼 수 있는 일련의 문자열로 구성된다.

	- 서버/클라이언트 인증 모델의 경우 3rd-pary 애플리케이션이 "아이디/패스워드"를 가짐으로써 보안문제를 가지고 있다. OAuth2›는 아이디/패스워드 대신에 access token을 교환함으로 보안성을 높일 수 있다.

- Refresh token
	- 유저가 접근승인을 받으면 access token을 발급받는다. 이 access token은 만료기간을 가지고 있어서, 만료기간을 넘어가게 되면 더 이상 사용할 수 없게 된다. 이 경우 access token을 발급받기 위한 과정을 다시 진행해야 한다.
	- 많은 경우 token endpoint(아래 설명한다)는 access token과 refresh token을 함께 제공한다. 유저는 refresh token을 이용해서 access token을 재 발급받을 수 있다. 대략 다음과 같은 방식이다.
- Scope
	- 접근하고자 하는 데이터셋 영역 (정보 어디까지 가져올지 설정)
[출처](http://www.joinc.co.kr/modules/moniwiki/wiki.php/man/12/oAuth2/About)



## Facebook 인증 
### facebook api key 발급

1. https://developers.facebook.com/접속 후 로그인

2. 우측 상단의 Register클릭
 
<br/>

	<img src="/assets/media/facebook/facebook_1.png">

3. 등록 완료되면 아래와 같은 메시지 출력

<br/>

	<img src="/assets/media/facebook/facebook_2.png">

4. 우측 상단의 App ID 만들기 클릭

<br/>

	<img src="/assets/media/facebook/facebook_3.png">

5. 프로젝트 명 입력 (네임스페이스는 옵션으로 페북안에 내 여러 프로젝트 중 구분하는 것 인지 아니면 특정 네임스페이스로 api쓸 떄 제공되는건지... 몰라서 그냥 스킵합)

<br/>

	<img src="/assets/media/facebook/facebook_4.png">

6. 완료되면 다음과 같이 App ID와 secret발급

<br/>

	<img src="/assets/media/facebook/facebook_5.png">

### javascript facebook 연동 

[웹 페이스북 가이드](https://developers.facebook.com/docs/facebook-login/web)
<br/>

1. 버튼 모양 생성
	`<div class="fb-login-button" data-max-rows="1" data-size="medium" data-show-faces="false" data-auto-logout-link="false"></div>`

<br/>

2. 페이스북 사용을 위한 등록

	> 
	 	<script>
		(function(d, s, id) {
			var js, fjs = d.getElementsByTagName(s)[0];
			if (d.getElementById(id))
				return;
			js = d.createElement(s);
			js.id = id;
			js.src = "//connect.facebook.net/ko_KR/sdk.js#xfbml=1&version=v2.5&appId=168276353537938";
			fjs.parentNode.insertBefore(js, fjs);
		}(document, 'script', 'facebook-jssdk'));
	  	</script>

3. 로그인 확인

<br/>

	<img src="/assets/media/facebook/facebook_6.png">

<br/>

4. 페이스북 개발사이트에 내 앱 URL명시

<br/>

	<img src="/assets/media/facebook/facebook_8.png">

<br/>

	<img src="/assets/media/facebook/facebook_7.png">

<br/>

5. 사용자 정보 얻어오기
- 처음 연결에서 response객체는 status, authResponse를 가지며 userID, accessToken을 가짐
- `scope="public_profile,email"` 설정 안하니 response에 name정보만 있음
- [scope 설정자료](https://developers.facebook.com/docs/facebook-login/permissions/#reference-public_profile)
- [Graph API](https://developers.facebook.com/docs/javascript/reference/FB.api)
- api문서 보고 원하는 동작 하자 (내가 원하는건 프로필사진, 이름, 이메일, 성별, 나이, 지역 정도? 추후 결정)
	
6. 내가 원하는 버튼에 페이스북 함수 연결 기본 모형 소스코드

> 
	<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="EUC-KR"%>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
	<title>Insert title here</title>
	</head>
	<body>
	<input type="button" id="button1" onclick="myFacebookLogin();"
		value="login" />
	<input type="button" id="button1" onclick="fb_logout();" value="logout" />
	<script type="text/javascript">
		window.fbAsyncInit = function() {
			FB.init({
				appId : '168276353537938',
				xfbml : true,
				version : 'v2.5'
			});
		};
		(function(d, s, id) {
			var js, fjs = d.getElementsByTagName(s)[0];
			if (d.getElementById(id)) {
				return;
			}
			js = d.createElement(s);
			js.id = id;
			js.src = "//connect.facebook.net/en_US/sdk.js";
			fjs.parentNode.insertBefore(js, fjs);
		}(document, 'script', 'facebook-jssdk'));
		function myFacebookLogin() {
			FB.login(function(response) {
				statusChangeCallback(response);
			}, {
				scope : 'public_profile, email'
			});
		}
		function statusChangeCallback(response) {
			// The response object is returned with a status field that lets the
			// app know the current login status of the person.
			// Full docs on the response object can be found in the documentation
			// for FB.getLoginStatus().
			if (response.status === 'connected') {
				// Logged into your app and Facebook.
				var uid = response.authResponse.userID;
				var accessToken = response.authResponse.accessToken;
				testAPI(uid);
			} else if (response.status === 'not_authorized') {
				// The person is logged into Facebook, but not your app.
				alert("login faile");
			} else {
				// The person is not logged into Facebook, so we're not sure if
				// they are logged into this app or not.
				alert("please input text");
			}
		}
		function testAPI(uid) {
			console.log("uid = " + uid);
			var fieldarray = [ 'bio', 'gender', 'name', 'timezone', 'email',
					'picture' ];
			//FB.api('/me', {fields: fieldarray}, function(response) {
			FB.api('/me/picture', function(response) {
				console.log(response);
			});
		}
		function fb_logout() {
			FB.logout(function(response) {
				console.log("logout");
				console.log(response.status);
				console.log(response);
			});
		}
	</script>
	</body>
	</html>

 <img src="/assets/media/facebook/facebook_9.png">

<br/>

**위 코드를 실행 했을 때 현재는 성별,이름,프로필사진,타임존 가져올게 설정**
**로그아웃은 이미지로 첨부하지 않았지만 동작함** 

7. 원하는 정보 가져오기

	- Graph api 문서의 내용에 사용자 정보 가져오려면 `FB.api('/{user_id}')`라 되있음
	- 근데 `user_id`하면 `/me`와 같은 동작을 하며 사용자 이름만 가져옴
	- 그래서 /me, fields로 원하는 속성 직접 입력함
	- 현재 필요한 정보 중 이메일 정보를 가져오지 못해 찾아보자
	- email정보는 사용자가 입력 안했을 경우 가져오지 못하므로 제외 하자

<br/>

 <img src="/assets/media/facebook/facebook_10.png">

<br/>

8. 가져올 데이터 확정
	- 사용자 이름
	- uid
	- 프로필사진 url
	- 페이스북인지, 트위터인지, 페이코인지 식별 키 삽입
 
9. 프로젝트 꾸미기
	- 권한 수신 확인 시  나타나는 우리 프로젝트 [아이콘](https://developers.facebook.com/apps/168276353537938/app-details/)여기서 등록
	- 로그인 및 로그아웃 버튼 이미지로 변경
 
## twitter 인증

## payco 인증 

