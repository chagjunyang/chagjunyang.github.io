---
layout: post
title: TextArea youtube embed
categories: [서버 기술 교육]
tags: [서버 기술 교육] 
description: 동적 iframe제공
comments: true
---

# Youtube

## 개요

- 덧글 입력 창인 TextArea에서 사용자 입력 이벤트 발생 시
- 유튜브 url이 있다면 iframe으로 실시간 보여주는 기능을 제공하고자 함

### 입력 가능 URL종류

1. https://www.youtube.com/watch?v=m_njhhMH6gM 브라우저 URL
2. https://youtu.be/m_njhhMH6gM 공유 URL
3. `<iframe width="560" height="315" src="https://www.youtube.com/embed/m_njhhMH6gM" frameborder="0" allowfullscreen></iframe>` 아이프레임

### 알고리즘

- 사용자 입력을 실시간으로 체크하여 유효성 검사
- iframe, https://youtu.be, https://www.youtube.com 3가지 경우의 유효성 검사
- 유효하다면 youtube영상의 고유값(key)을 추출
- 배열에 해당 key값 저장
- iframe태그 만들어 화면에 출력 -> 이때 삭제를 위해 각 iframe태그의 id값을 배열.length로 한다.
- 입력 이벤트 발생시 출력한 iframe이 텍스트창에 삭제되었나 확인해서 html에서 제거
- 덧글 입력 완료 시 iframe제거
- 현재 상당히 지저분하다... 

>
	$("#commentArea").keyup(function(e) { //파일 선택 시 
        e.preventDefault();
        var keyed = $(this).val().replace(/\n|\"/g, " "); //줄바꿈제거
        var keyString, iframe;
        isDeleteYoutube(urlArr, keyed);  //텍스트에서 지워졌다면 iframe제거
        if(keyed != 0){
          while(  ((keyString = getYoutubeURL(keyed)) != -1)  && keyString.key != -1 )//url 끝 지점과 유튜브 id값 반환
          {
            if( $.inArray(keyString.key, urlArr) == -1) //중복된 youtube가 아니라면
            {
              urlArr.push(keyString.key);
              this.iframe = getIframe(keyString.key, urlArr.length);
              iframe_parent.append(this.iframe);
            }
            keyed = keyed.substring(keyString.idx, keyed.length); //첫 번째 http끝 부터 다시 검사
          }
        }
      }); 

### 유효성 검사

> 
	function validateYouTubeUrl(url)
	{
	  if (url != undefined || url != '') {
	    var regExp = /^.*(youtu.be\/|v\/|u\/\w\/|embed\/|watch\?v=|\&v=|\?v=)([^#\&\?]*).*/;
	    var match = url.match(regExp);
	    if (match && match[2].length == 11) {
	      return match[2];
	    }
	    else {
	      return -1;
	    }
	  }
	}


### key값으로 iframe만들

>
	function getIframe(url, id)
	{
	  return '<iframe id=youtube'+id+' src="http://www.youtube.com/embed/'+url+'?rel=0" height="150" width="180" allowfullscreen="" frameborder="1"></iframe>';;
	}

### 자바스크립트 큰 따옴표 및 줄바꿈 제거

- [정규식 가이드](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/%EC%A0%95%EA%B7%9C%EC%8B%9D)
- $(this).val().replace(/\n|\"/g, " ");  

### 실행화면

<img src="/assets/media/youtube.png">


<img src="/assets/media/youtube2.png">

1. https://www.youtube.com/watch?v=m_njhhMH6gM 형식 가능
2. https://youtu.be/m_njhhMH6gM 형식 가능
3. iframe형식 가능
4. 실시간 입력에 따른 iframe출력, 텍스트 제거에 따른 iframe제거
5. 다음에 시간이 된다면 슬라이드쉐어 api써서 해보자  
