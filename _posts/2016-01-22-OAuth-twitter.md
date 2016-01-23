---
layout: post
title: Twitter 인증
categories: [개인]
tags: [정보]
description: spring mvc twitter login
comments: true
---

# 자바스크립트 트위터 연동

## 1. 트위터 연동 

### 트위터 API사용방식

###1. 웹용 컴포넌트를 가져다 쓰는 방식 (Twitter for Websites)

###2. 트위터 API를 직접 사용하는 방식 (REST/Stream API)

#### 트위터등록
- https://apps.twitter.com/app/ 에서 등록
- 핸드폰 등록 필요
- 현재 url http://127.0.0.1로 셋팅

#### 인증순서 [출처](https://ethdemor.wordpress.com/2012/01/28/%ED%8A%B8%EC%9C%84%ED%84%B0-api-%ED%94%8C%EB%9E%AB%ED%8F%BC%EA%B3%BC-oauth-access-token-%EC%96%BB%EA%B8%B0/)

1. 앱에 발급받은 Consumer Key와 시크릿을 이용해 request token을 발급
2. request token은 해당 컨슈머를 대변한다.
3. 이제 컨슈머는 준비완료 앞으로 사용자 키만 준비하면됨
4. 사용자는 컨슈머의 정보제공에 동의 하면 컨슈머는 verifier값과 Request Token을 트위터로 보내 최종 Access Token을 발급
5. 컨슈머는 에세스토큰으로 사용자를 대변해 트위터 정보 가져옴

- REQUEST TOKEN을 발급 받기위해  http://dev.twitter.com/oauth/request_token을 POST로 호출
- 사용자 리다이렉트 하기
- 리퀘스트 토큰 바급 후 사용자 인증을 받기 위해  https://api.twitter.com/oauth/authenticate리다이렉트
- 리퀘스트토큰과 Oauth_Verifier를가지고  https://api.twitter.com/oauth/access_token을 POST 방식으로 호출
- AccessToken을 가지고 트위터 api라이브러리 사용

#### 트위터 라이브러리

1. java코드 이용 spring에서 구현
2. pom.xml에 라이브러리 추가

>
	<dependency>
      <groupId>org.twitter4j</groupId>
      <artifactId>twitter4j-core</artifactId>
      <version>4.0.4</version>
    </dependency>

**동작순서**

	1. 트위터에 등록한 consumerKey, secret을 이용해 request토큰 발급
	2. requestToken 발급 시 callbackURL을 추가하여 내 컨트롤러로 동작하게
	3. requestToken.getAuthenticationURL()에 트위터 로그인 주소가 있고 주소에서 인증하고 나면 callbackURL로 돌아온다.
	4. callbackURL에서는 request를보면 oauth_token, oauth_verifier 2개의 파라미터가 있다.
	5. requestToken과 verifier값을 이용해 access토큰을 얻는다.

**이슈**

- 3번의 4번 에서 loginController에서 트위터 로그인화면 호출하고 callback으로 callbackController이동 시 세션유지 실패
- 아래 사진의 트위터 앱등록 시 호스트 주소와 callback을 정확히 입력해 주어야 했음... localhost대신 127.0.0.1사용
<img src="/assets/media/twitter/1.png">
- 페이스북과 트위터 앱 등록시 페이스북은 호스트방식이고 트위터는 url방식이라 같은 브라우저에서 현재 사용불가
- host파일 수정하던가 도메인서버 받던가 해결책 찾아야함


** 트위터 callback url 해결 **

1. 트위터 개발자 사이트의 block url을 체크해제
2. 사이트에 등록한 callback url은 default url로 됨
3. 내 웹에서 request토큰 발급 받을 때 callback url이 우선순위
4. 내 소스의 callback url에 localhost로 주면 facebook과 같이 이용 가능

###소스코드

#### twitterController.java
>
	@Controller
	public class TwitterController {
    private String consumerKey = "cHCWNbHCjmRPa1T0V2mk0zvbu";
    private String consumerSecret = "yeNnxuOZLEhgZ5kKMn2CSYdWrPT61P0LtwNDjon0c0JjPZ4U32";
    private static final Logger logger = LoggerFactory.getLogger(TwitterController.class);

    @Resource(name = "MemberService")
    private MemberService memberService;

    /*
     * (1/22 양창준 수정) 트위터 로그인
     */
    @RequestMapping(value = "/twitter/login", method = RequestMethod.GET)
    public String home(Locale locale, Model model, HttpServletRequest request) throws TwitterException {
        logger.info("Welcome home! The client locale is {}.", locale);

        Twitter twitter = new TwitterFactory().getInstance(); // 트위터 라이브러리 객체생성
        request.getSession().setAttribute("twitter", twitter);
        twitter.setOAuthConsumer(consumerKey, consumerSecret); // 앱에 등록된 키, 시크릿
        // 입력
        RequestToken requestToken = twitter.getOAuthRequestToken("http://localhost:8080/happyNews/twitter/callback");
        // requestToken값 세션에 저장 후 콜백 호출해서 에세스토큰 get
        request.getSession().setAttribute("requestToken", requestToken);

        // 사용자에게 id/pwd입력 받은 다음 콜백으로 등록한 콘트롤러로 이동
        return "redirect:" + requestToken.getAuthenticationURL();
    }

    /*
     * (1/22 양창준 수정) 트위터 로그인
     */
    @RequestMapping(value = "/twitter/callback")
    public String twitter_callback(Model model, HttpServletRequest request) throws Exception {
        HttpSession session = request.getSession();
        Twitter twitter = (Twitter) session.getAttribute("twitter");

        RequestToken requestToken = (RequestToken) session.getAttribute("requestToken");
        String verifier = request.getParameter("oauth_verifier");
        AccessToken accessToken = twitter.getOAuthAccessToken(requestToken, verifier);

        session.removeAttribute("requestToken");

        Member member = new Member();
        member.setUid("" + twitter.showUser(accessToken.getUserId()).getId());
        member.setName(twitter.showUser(accessToken.getUserId()).getName());
        member.setFlag(OAuthProvider.twitter);
        member.setImg(twitter.showUser(accessToken.getUserId()).getProfileImageURL());

        System.out.println(member.getImg());
        System.out.println(member.getName());
        System.out.println(member.getFlag());

        // Insert DB
        if (!memberService.isMember(member))
            memberService.insertMember(member);

        session.setAttribute(OAuthProvider.session, member);

        return "redirect:" + session.getAttribute("prePath");
    }
}
