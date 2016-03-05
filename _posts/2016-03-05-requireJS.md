---
layout: post
title: JavaScript RequireJS
categories: [개인]
tags: [정보]
description: 뭐하는놈이고..
comments: true
---
# RequireJS

- [nhnent-FE.javascript-github](https://github.com/nhnent/fe.javascript/wiki)
- 우리 회사의 FE개발팀의 github wiki를 보면 자바스크립트에 대해 여러 가이드가 나온다.
- 구경하던 중 의존성 관리라는 항목을 발견.
- 항목 설명은 다음과 같았다.
	- `자바스크립트의 의존성 관리를 좀 더 쉽게 해결할 수 있도록 모듈 간 의존성을 관리하는 방법을 가이드한다.`

- 글을 보자마자 내 프로젝트의 코드가 머리속에 떠오르게 되었고 당장 적용해보기로 결심했다.
- 쓰레기같은 내 코드를 꾸밀 생각에 이 떄 잠시나마 행복했다.

## 현재 내 상황
 
### articleDetail.jsp

	<script src="<c:url value='/js/jquery.min.js'/>"></script>
	<script src="<c:url value='/bootstrap/js/bootstrap.min.js'/>" charset="utf-8"></script>
	<script type="text/javascript" src="<c:url value='/toastui/pagination/js/code-snippet.js'/>"></script>
    <script type="text/javascript" src="<c:url value='/toastui/scrollend/js/scrollend.min.js'/>"></script>
	<script type="text/javascript" src="<c:url value='/js/article/search.js'/>" charset="utf-8"></script>
	<script type="text/javascript" src="<c:url value='/js/ejs/ejs.js' />"></script>
	<script type="text/javascript" src="<c:url value="/js/article/article.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/fileupload.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/comment.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/youtube.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/comments.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/eventHandler.js" />"></script>

### 문제점

- eventHandler.js는 위 /js/comment의 모든 js파일에 종속적이다. (4개파일)
- 따라서 eventHandler는 제일 하단에 선언되야 정상동작 한다.
- comments.js는 위 /js/comment의 3개 .js파일에 종속적이다. (3개 파일)
- 따라서 comment.js는 3개의 파일보다 나중에 선언되야 정상동작 한다.
- 나중에 파일이 확장되거나 의존성이 변경될 때 마다 선언 위치를 확인해야 하는문제가 발생
- 서로 참조하는 경우 아예 동작이 불가

### 해결책

- requirejs를 이용해 의존관계에 있는 js파일들의 관계를 정리한다.

## Require JS?

### 자바스크립트의 의존성 관리

- 자바스크립트는 기본적으로 코드 내에서 외부 코드를 불러오는 방법이 제공되지 않고 html의 script 태그에 의존하여 코드를 로드하게 된다.
- 기존의 방식은 HTML에서 의존관계에 따라 순서대로 스크립트를 로드하고, 코드내에서 네임스페이스를 정의하여 사용하는 방식이였다. (현재 내 코드!!)
- 하지만 이런 방식은 개발자가 그 의존관계와 순서를 일일이 신경 써야 하고, 어플리케이션의 규모가 커질수록 관리 비용이 심각하게 증가하는 문제가 있다.
- 이에 대한 대안으로 CommonJS와 AMD(Asynchronous Module Definition) 같은 표준이 등장하기 시작했다.
- 서버 사이드에서는 CommonJS 방식의 Node.js가 클라이언트에서는 AMD 방식의 RequireJS가 주로 사용되고 있었는데, 최근에는 클라이언트에서도 CommonJS 방식의 browserify가 많이 사용되고 있는 추세이다.

### RequireJS

<img src="/assets/media/requirejs/require1.png">

- [RequireJS](http://requirejs.org/)는 AMD 방식의 대표적인 자바스크립트 모듈 관리 라이브러리이다.
- CommonJS에 비해 사용 방식이 복잡한 편이지만 클라이언트단에서는 대안이 없을 정도로 안정화된 기능을 제공한다.
- 거의 모든 브라우저를 지원하며 다양한 플러그인으로 기능을 확장할 수 있다.


## RequireJS 시작하기

### require.js포함

- [다운로드](http://requirejs.org/docs/download.html)
- 사용하고자 하는 페이지에 다음과 같이 스크립트 파일을 포함 (경로는 각자 위치에 맞게)
- 나의 경우 `articleDetail.jsp`파일 안에 기존의 `<script>`로 선언된 js들 위에 선언했다.
- `<script type="text/javascript" data-main="/js/main" src="/js/requirejs.js"></script>`
- data-main은 require.js의 진입점을 뜻한다.
- `/js/main.js`파일을 만들고 그 안에 require설정과 진입점을 명시한다.

### 모듈화 하기 

- require.js에 있는 define()함수를 이용해 모듈을 선언한다.
- `define([], function(){})`의 형태이다.
- 첫 번째 인자로 의존성있는 모듈을 받는다.
- 두 번째 인자로 모듈에서 실행할 함수 단위 스코프로 작성한다.
- fucntion의 인자로 []배열의 인자가 순서대로 들어온다. arguments를 사용해 인자를 받아도 된다.
- 나의 경우 다음과 같이 사용했다.

### 1. 모듈 정의 및 반환
	define(function(){
		var exportModule = {};
		var test = function(){ console.log("test")};
	 	exportModule.test = test;

		return exportModule;
	});


### 2. define 첫 번째 인자로 사용하는 모듈 명시하기
	define(['comment', 'fileupload', 'youtube', 'common', 'jquery'], function(commentModule,fileuploadModule,youtubeModule,commonMoudle, $) {
	});

- 원래 대로라면 `'./comment'` 또는 `'/js/comment'`처럼 경로를 사용
- 나의 경우 뒤에 설명할 require.config에 네이밍?? 별칭? DB의 ansi같은 느낌으로 이름 설정
- 기본적으로 그냥 사용하면 .js가 저절로 붙어 해당 파일을 찾는다.


### 3. 함수 내에서 의존관계 모듈 받기

	define(function(require) {
  	var $ = require('jquery');
  	var exportModule = {};
  	var commentNamespace = require('comment');
  	var youtubeNamespace = require('youtube');
  	var fileUploadNamespace = require('fileupload');
  	var moment = require('locales');
  	});

- require('모듈명') 할 경우 requirejs에서 선언한 모듈만 딱 찾아 해당 모듈에 할당해준다.
- 정확한 원리는 아직 공부가 부족하여 모르겠으나 function파라미터로 넘겨받은 놈이 requirejs에서 주는 먼가 하는 놈이고
- 이놈을 이용해 사용할 모듈을 정의할 수 있다.
- 시간나면 공부해보자. require찍어보니 `function e(c,d,g)`함수인데 스코프, 프로토타입, 클로저 모르는 내용이 많다

### 4. RequireJS 설정하기

<img src="/assets/media/requirejs/require2.png">


### 5. 내 설정 파일 (/js/main.js)

	requirejs.config({
	  baseUrl:'/js',
	  paths : {
	    /*
	    * external js files
	    */
	    'jquery': 'jquery.min',
	    'bootstrap': '/bootstrap/js/bootstrap.min',
	    'scrollend': '/toastui/scrollend/js/scrollend.min',
	    'code-snippet': '/toastui/pagination/js/code-snippet',
	    'underscore': 'underscore/underscore',
	    'ejs': 'ejs/ejs',
	    'locales': 'moment-with-locales',
	    /*
	     * HappyNews module js files
	     */
	    'eventHandler': 'comment/eventHandler',
	    'comments': 'comment/comments',
	    'fileupload': 'comment/fileupload',
	    'youtube': 'comment/youtube',
	    'comment': 'comment/comment',
	    /*
	     * HappyNews unmodule js files
	     */
	    'search': 'article/search',
	    'article': 'article/article'
	  },
	  
	  shim : {
	    'bootstrap' : {
	      deps: ['jquery'],
	      exports: 'bootstrap'
	    },
	    'article': {
	      deps: ['jquery']
	    },
	    'scrollend': {
	      deps: ['code-snippet']
	    },
	    'ejs': {
	      deps: ['locales']
	    },
	    'comments': {
	      deps: ['scrollend', 'ejs', 'underscore']
	    },
	    'eventHandler': {
	      deps: ['search']
	    }
	  }
	});
	
	require( ['comments','eventHandler', 'article'], function (comment, test, comments) {
	});


- 기본경로를 /js로 주어 여러 파일들을 사용
- paths는 4번의 설명과 같고 모듈이름을 미리 다 정의했다.
- 현재 /js/comment/*.js 파일들은 모듈화를 완료했지만 article의 js파일들은 모듈화가 안된 상태이다.
- 따라서 shim에 모듈화가 안된 파일들을 정의하고 의존관계를 정의한다.
- 여기서 머리에 쥐가난다... 어떤놈이 어떤놈을 사용하는지 잘 파악해야 한다.
- 그리고 한가지 더 shim에서 'jquery' : { exports : $}로하면 전역에 $변수가 선언되므로 모든 곳에서 jquery사용이 가능할 줄 알았다.
- 하지만 jquery.js가 로딩되는 시점과 jquery를 사용하는 파일의 로딩시점이 달라 에러가 발생했다.
- 이럴 경우 shim에 의존관계를 명시하거나 해당 모듈에서 jquery 모듈을 받아와 사용하면 된다. 나의 경우 후자.

### 6. 결과

- 이전 articleDetail.jsp 코드
>
	<script src="<c:url value='/js/jquery.min.js'/>"></script>
	<script src="<c:url value='/bootstrap/js/bootstrap.min.js'/>" charset="utf-8"></script>
	<script type="text/javascript" src="<c:url value='/toastui/pagination/js/code-snippet.js'/>"></script>
    <script type="text/javascript" src="<c:url value='/toastui/scrollend/js/scrollend.min.js'/>"></script>
	<script type="text/javascript" src="<c:url value='/js/article/search.js'/>" charset="utf-8"></script>
	<script type="text/javascript" src="<c:url value='/js/ejs/ejs.js' />"></script>
	<script type="text/javascript" src="<c:url value="/js/article/article.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/fileupload.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/comment.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/youtube.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/comments.js" />"></script>
	<script type="text/javascript" src="<c:url value="/js/comment/eventHandler.js" />"></script>

- `<script type="text/javascript" data-main='/js/main' src="/js/require.js"></script>` 한 줄로 깔끔해 졌다.
- 하지만 `/js/main.js`의 require.config설정이 생겼고 여기가 좀 난해하다.


## 느낀점

- 기존에 javascript load 시 선언하는 위치가 매우 중요했다
- 팀원이 js파일 하나 추가하려면 모든 사람에게 위치를 물어보고 결정해야 했다.
- 지금은 설정파일에서 의존관계를 명시적으로 볼 수 있기 때문에 전보다는 훨 씬 이해가 쉬운 것 같다.
- 근데 지금 이렇게 적용하면서도 먼가 이상하다 먼가 잘못한 것 같은데...
- requireJS에 관해 자료를 찾다보면 백본이 같이 등장하는데 머하는 놈인지 모르겠다.
- 지금 내 프로젝트에서도 스코어보드랑 jquery를 사용하는데... 흠
- 시간이 되면 backbone.js도 공부해봐야 겠다
- 아니 그전에 프로토타입이랑 클로저? 상속? 책부터 보자!!!!!