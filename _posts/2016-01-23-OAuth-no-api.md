---
layout: post
title: OAuth not using API
categories: [개인]
tags: [정보]
description: test
comments: true
---

# Spring Oauth without libaries
## get requestToken
[예제](https://twittercommunity.com/t/solved-java-oauth-request-token-flow-example-without-libraries/1440/9)
[OAth anchor](http://oauth.net/core/1.0/#anchor9)
### Requset call example 
>
	Example Result
	Request URL:
	POST https://api.twitter.com/oauth/request_token
	Request POST Body:
	N/A
	Authorization Header:
	OAuth oauth_nonce="K7ny27JTpKVsTgdyLdDfmQQWVLERj2zAK5BslRsqyw",
	oauth_callback="http%3A%2F%2Fmyapp.com%3A3005%2Ftwitter%2Fprocess_callback", 
	oauth_signature_method="HMAC-SHA1", 
	oauth_timestamp="1300228849",
	oauth_consumer_key="OqEqJeafRSF11jBMStrZz", 
	oauth_signature="Pc%2BMLdv028fxCErFyi8KXFM%2BddU%3D", 
	oauth_version="1.0"
>
	Response:
	oauth_token=Z6eEdO8MOmk394WozF5oKyuAv855l4Mlqo7hhlSLik&oauth_token_secret=Kd75W4OQfb2oJTV0vzGzeXftVAwgMnEK9MumzYcM&oauth_callback_confirmed=true

### Collecting parameters [자료](https://dev.twitter.com/oauth/overview/authorizing-requests)

#### 1. Nonce 
- 각 응용프로그램이 고유한 요청을 생성해야 하는 토큰
- 이 토큰으로 여러변 요청이 제출됬는지 확인한다 하는데 왜 필요한지까지는 모름...
- 한 번 사용하고 버려지는 고유 키이다.
- 자바의 UUID.randomUUID()를 이용하여 생성
>    
	public String getNonce() {
        // generate any fairly random alphanumeric string as the "nonce". Nonce
        // = Number used ONCE.
        String uuid_string = UUID.randomUUID().toString();
        return uuid_string.replaceAll("-", "");
    }

#### signature
- 서명 알고리즘을 통해 발급된 값
- 트위터 요청이 전송 중 변경되지 않았나 확인 할 때 사용
- 이 애플리케이션이 사용자의 계정과 상호 작용할 수 있는 권한이 있는지 검증하는 목적
- [signature 생성법](https://dev.twitter.com/oauth/overview/creating-signatures)
- 1. 요청 시 들어가는 파라미터 (Request call example의 내용들 header부분)의 키, 값 [precent encode](https://dev.twitter.com/oauth/overview/percent-encoding-parameters) 인코딩


#### 


### sendURL

### response에서 토큰 얻기
- response body부분 추출하면 다음과 같음
- `oauth_token=cVse9QAAAAAAj99fAAABUmp3Q4s&oauth_token_secret=RUdJt4t4PBXHrg0iUWXT9OYBp1r5CZLU&oauth_callback_confirmed=true`
- 여기서 oauth_token과 oauth_token_secret을 저장
> 
	 public void getOautToken(String responseBody) {
        // true or false의 값을 갖는다.
        String occ_val = responseBody.substring(responseBody.indexOf("oauth_callback_confirmed=") + 25);
        // 옳바르게 받아 왔다면
        if (occ_val.equals("true")) {
            StringTokenizer st = new StringTokenizer(responseBody, "&");
            String currenttoken = "";
            while (st.hasMoreTokens()) {
                currenttoken = st.nextToken();
                if (currenttoken.startsWith("oauth_token="))
                    oauth_token = currenttoken.substring(currenttoken.indexOf("=") + 1);
                else if (currenttoken.startsWith("oauth_token_secret="))
                    oauth_token_secret = currenttoken.substring(currenttoken.indexOf("=") + 1);
                else if (currenttoken.startsWith("oauth_callback_confirmed=")) {
                } else {
                    System.out.println("트위터에서 키를 받아오지 못했습니다.");
                }
            }
        }

- 아래 이미지는 트위터사이트에서 가져온 동작 순서이다

<br/>

<img src="/assets/media/twitter/oauth.png">

<br/>

- 현재 B까지 구현

### 사용자 인증 받기
- c에서 사용자 인증을 받으려면 앞서 발급받은 oauth_token값을 전달
- 트위터 로그인창 사용자 id/pwd입력
- [자료](https://dev.twitter.com/oauth/reference/get/oauth/authenticate)
- parameter로 force_login (자동 로그인 할 것인지) / screen_name 등 옵션으로 있다

#### Authorization Header
> 
				Authorization: OAuth realm="http://sp.example.com/",
                oauth_consumer_key="0685bd9184jfhq22",
                oauth_token="ad180jjd733klru7",
                oauth_signature_method="HMAC-SHA1",
                oauth_signature="wOJIO9A2W5mFwDgiDvZbTSMK%2FPY%3D",
                oauth_timestamp="137131200",
                oauth_nonce="4572616e48616d6d65724c61686176",
                oauth_version="1.0"

