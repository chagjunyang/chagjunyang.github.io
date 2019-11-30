---
layout: post
title: OAuth 1.0 (without OAuth API)
categories: [서버 기술 교육]
tags: [서버 기술 교육]
description: 트위터 uid, scree_name까지
comments: true
---

# OAuth 1.0 프로토콜을 이용한 사용자 인증 

참고 자료

[OAth anchor](http://oauth.net/core/1.0/#anchor9)

[OAuth RFC](https://tools.ietf.org/html/rfc5849#page-1)

[네이버자료](http://d2.naver.com/helloworld/24942) -> 정리 잘됨 (나중에 찾음...후)

RFC가 좀 더 상세하나 ANCHOR도 좋은 듯

## get requestToken 
- 여기서는 트위터를 대상으로 인증
- 페이스북, 페이코는 2.0 지원 (다 개발하고서 알았따...)
- 사용자 uid, profile_image_url, name 정보를 얻어오기 까지 과정들

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


### 1. Nonce 

- 각 응용프로그램이 고유한 요청을 생성해야 하는 토큰
- 이 토큰으로 여러변 요청이 제출됬는지 확인한다 하는데 왜 필요한지까지는 모름...
- 한 번 사용하고 버려지는 고유 키이다.
- 자바의 UUID.randomUUID()를 이용하여 생성
>    
	public String getNonce() {
        String uuid_string = UUID.randomUUID().toString();
        return uuid_string.replaceAll("-", "");
    }


### 2. TimeStamp

- [OAuth RFC](https://tools.ietf.org/html/rfc5849#page-17) 참고

>
	public String getTimeStamp() {
        Calendar tempcal = Calendar.getInstance();
        long ts = tempcal.getTimeInMillis();
        return new Long(ts / 1000).toString();
    }


### 3. signature

- 서명 알고리즘을 통해 발급된 값
- 트위터 요청이 전송 중 변경되지 않았나 확인 할 때 사용
- 이 애플리케이션이 사용자의 계정과 상호 작용할 수 있는 권한이 있는지 검증하는 목적
- [signature 생성법](https://dev.twitter.com/oauth/overview/creating-signatures)
- 1. 요청 시 들어가는 파라미터 (Request call example의 내용들 header부분)의 키, 값 [precent encode](https://dev.twitter.com/oauth/overview/percent-encoding-parameters) 인코딩
- 시그니처 생성법은 인터넷에 get oauth signature 등 검색하면 방법 있음
- 나의 signature계산 법은 다음과 같다.

>
	public String getSignature(String linkedParmsString, String url)
            throws UnsupportedEncodingException, GeneralSecurityException, MalformedURLException {
        String signature_base_string = "POST" + "&" + encode(url) + "&" + encode(linkedParmsString);
        return computeSignature(signature_base_string, ConsumerSecret + "&");
    }

>
	String authorization_header_string = "OAuth oauth_consumer_key=\"" + ConsumerKey + "\",oauth_signature_method=\"HMAC-SHA1\",oauth_timestamp=\"" + timeStamp + "\",oauth_nonce=\"" + Nonce + "\",oauth_version=\"1.0\",oauth_signature=\"" + encode(signature) + "\"";

>
	 public static String computeSignature(String baseString, String keyString)
            throws GeneralSecurityException, UnsupportedEncodingException {
        byte[] keyBytes = keyString.getBytes();
        SecretKey secretKey = new SecretKeySpec(keyBytes, "HmacSHA1");
        Mac mac = Mac.getInstance("HmacSHA1");
        mac.init(secretKey);
        byte[] text = baseString.getBytes();
        String result = Base64.getEncoder().encodeToString(mac.doFinal(text));
        return result.trim();
    }
          


### 4. encode

- request 시 oauth 1.0에서 원하는 포멧으로 encode해야함
- [OAuth RFC](https://tools.ietf.org/html/rfc5849#page-17)에 데이터 포멧 있음

> 
	 public String encode(String value) {
        String encoded = null;
        try {
            encoded = URLEncoder.encode(value, "UTF-8");
        } catch (UnsupportedEncodingException ignore) {
        }
        StringBuilder buf = new StringBuilder(encoded.length());
        char focus;
        for (int i = 0; i < encoded.length(); i++) {
            focus = encoded.charAt(i);
            if (focus == '*') {
                buf.append("%2A");
            } else if (focus == '+') {
                buf.append("%20");
            } else if (focus == '%' && (i + 1) < encoded.length() && encoded.charAt(i + 1) == '7'
                    && encoded.charAt(i + 2) == 'E') {
                buf.append('~');
                i += 2;
            } else {
                buf.append(focus);
            }
        }
        return buf.toString();
    }


### 5. send HTTP request

- 나는 spring 4.2.4, java 8, maven 기준으로 사용
- RestTemplate 사용
- 헤더에 파라미터 표시
- Authorization이 키 값이 되야 함

>
	Public String sendURL(String authorization_header_string, String url) {
        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.add("Authorization", authorization_header_string);
        HttpEntity<String> entity = new HttpEntity<String>("parameters", headers);

        ResponseEntity<String> response = restTemplate.exchange(url, HttpMethod.POST, entity, String.class);
        response.getBody();

        return response.getBody();
    }


### response에서 토큰 얻기

- response body부분 추출하면 다음과 같음
- `oauth_token=cVse9QAAAAAAj99fAAABUmp3Q4s&oauth_token_secret=RUdJt4t4PBXHrg0iUWXT9OYBp1r5CZLU&oauth_callback_confirmed=true`
- 여기서 oauth_token과 oauth_token_secret을 저장
> 
	 public void getOautToken(String responseBody) {
        // true or false의 값을 갖는다.
        String occ_val = responseBody.substring(responseBody.indexOf("oauth_callback_confirmed=") + 25);
        // 올바르게 받아 왔다면
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


<img src="/assets/media/twitter/oauth.png">


### 사용자 인증 받기

- c에서 사용자 인증을 받으려면 앞서 발급받은 oauth_token값을 전달
- 트위터 로그인창 사용자 id/pwd입력
- [자료](https://dev.twitter.com/oauth/reference/get/oauth/authenticate)
- parameter로 force_login (자동 로그인 할 것인지) / screen_name 등 옵션으로 있다


### Authorization Header

> 
				Authorization: OAuth realm="http://sp.example.com/",
                oauth_consumer_key="0685bd9184jfhq22",
                oauth_token="ad180jjd733klru7",
                oauth_signature_method="HMAC-SHA1",
                oauth_signature="wOJIO9A2W5mFwDgiDvZbTSMK%2FPY%3D",
                oauth_timestamp="137131200",
                oauth_nonce="4572616e48616d6d65724c61686176",
                oauth_version="1.0"


- 사용자 한테 ID/PWD받으 려면
- https://api.twitter.com/oauth/authorize?oauth_token=86-REQAAAAAAj99fAAABUm2vwBQ 이런 식으로 url요청 하면 앱 인증 화면으로 간다.
- 여기서 콜백으로 등록한 곳으로 앱 호출


### GetAccessToken

- 사용자 인증 후 받은 oauth_verifier와 requestToken을 이용해 accessToken을 얻는다.
- HTTP request 예시

> 
	 POST /request_token HTTP/1.1
     Host: server.example.com
     Authorization: OAuth realm="Example",
        oauth_consumer_key="jd83jd92dhsh93js",
        oauth_token="hdk48Djdsa",
        oauth_signature_method="PLAINTEXT",
        oauth_verifier="473f82d3",
        oauth_signature="ja893SD9%26xyz4992k83j47x0b"

- 위 적은 함수를 사용하여 get해보자
>
	 public String getAccessToken(String verifier, String token) throws Exception {
        String twitter_access_endpoint = "https://api.twitter.com/oauth/access_token";
        String Nonce = getNonce();
        String timeStamp = getTimeStamp();

        String linkedParmsString = "oauth_consumer_key=" + ConsumerKey + "&oauth_nonce=" + Nonce+ "&oauth_signature_method=" + signature_method + "&oauth_timestamp=" + timeStamp + "&oauth_token="+ token + "&oauth_verifier=" + verifier + "&oauth_version=1.0";

        String signature = getSignature(linkedParmsString, twitter_access_endpoint);

        System.out.println("oauth_signature=" + encode(signature));

        String authorization_header_string = "OAuth oauth_consumer_key=\"" + ConsumerKey+ "\",oauth_signature_method=\"HMAC-SHA1\",oauth_timestamp=\"" + timeStamp + "\",oauth_nonce=\"" + Nonce+ "\",oauth_version=\"1.0\",oauth_signature=\"" + encode(signature) + "\",oauth_token=\"" + token+ "\",oauth_verifier=\"" + verifier + "\"";

        String responseBody = sendURL(authorization_header_string, twitter_access_endpoint);

        System.out.println("accescc body");
        System.out.println(responseBody);
        return responseBody;
    }

- 다음은 응답의 결과로 access_token을 발급 받은 것 을 확인할 수 있다. 
- 아직 포스팅 못한 OAuth RFC분석에서 볼 수 있겠지만 기존에 라이브러리 사용할 때 쓰던
- RequestToken, AccessToken 등의 
- 는 문서에서 확인할 수 없었다.
- 하지만 개념이 대응되서 더 쉽게 이해할 수 있었음
- (나중에 찾아보니 2.0과 1.0의 용어 차이였음..)
 `oauth_token=4833688041-7mWgFX6sEHNappjRi0wfemrHMIXJpeFncdW8O7g&oauth_token_secret=Tj6vvv18wenIZqCzCFN19WLLGmv4IJBPJWo01vmgqUeAx&user_id=4833688041&screen_name=YangChangjun&x_auth_expires=0`


<img src="/assets/media/twitter/oauth.png">

- uid는 해당 oauth 서버에서 사용자를 식별하는 고유 값이다.
- screen_name은 string형 고유값이다. 트위터에서 계정 명에 해당한다.

### 사용자 정보 얻기

- 트위터 [REST API](https://dev.twitter.com/rest/public)
- request에 대해 인증된 클라이언트와 사용자면 json형식 응답
- 응답에는 요청에 따른 정보가 있다.

