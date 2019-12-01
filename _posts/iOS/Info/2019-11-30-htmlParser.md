---
layout: post
title: HTML Parser 만들기
categories: [iOS]
tags: [iOS 정보]
description: 간단한 Dom, Sax 방식 파서
comments: false
---

[Git](https://github.com/chagjunyang/SimpleHTMLParser)

### 개요

- `<font color='#FA2828'/> 안녕하세요</font>` 와 같이 html string을 서버에서 받아와 화면에 표시하는 니즈 있음
- NSAttributedString을 이용해 파싱할 경우 메인스레드에서 동작하는데 많은 데이터를 파싱하면 크래시
- 파싱코드를 메인스레드가 아닌곳에서 돌리고 완성되면 메인스레드에서 다시 호출해주면 크래시현상은 피할 수 있었으나 속도가 느림
- Rx도 도입하지않았고, 셀프사이징셀도 사용하지 않은곳이 많아 비동기적으로 처리하기에는 이슈가 있음
- Payco에서는 간단한 font, br, b, u 4개정도의 태그만 사용하고 속성값도 몇개 없어서 파서의 성능개선이 가능
- 결과적으로 짧은 문자열 여러번 파싱의 속도는 평균 500배이상, 긴 문자열 파싱은 1.3~1.5배 빨라짐

# 돔 파서

파싱과정
1. 어휘분석
2. 구문분석 -> 트리구성
3. 트리로부터 attrbutedSting 구성

### 어휘 분석

- htmlString으로부터 의미있는 단위의 문자를 찾는 행위
- token을 open, close, text, br 4가지로 구분

1. 문자열에서 '<' 찾아 좌측 있는지 확인
- 좌측이 있다면 text항목 토큰
- 파스트리에 text토큰 삽입
- nextHTMLString = '<'우측
- 2번 상태로 진입

2. '<' 우측
- '>' 만날때까지가 tag에 대한 string
- tagString 파싱
- nextHTMLString = '>' 우측
- 1번 상태로 진입

3. tagString 파싱
- 트림
- 기본 open속성의 토큰
- tagString에 "</" "/>" 포함되어있으면 close토큰속성으로 변경
- 공백을 기준으로 splite : 첫번째 splite항목 = tagName, 우측 = 속성 list
- tagName == 'br' 이면 br토큰으로 속성 변경
- 트리에 toeken삽입

### 구문 분석

- 어휘분석을 통해 만들어진 파스트리를 순회하면서 attributedString을 만드는 과정

1. 바깥에서 전달받은 속성으로 attributedDict만듬
2. root부터 시작해 자식노드들에게 attributedDict전달

노트 종류에 따른 동작
case. open :
- 전달받은 attributedDict에 속성 update
- 자식 토큰에게 업데이트된 attributedDict전달  (재귀적)
- 만들어진 attributedString 반환

case .text:
- 전달받은 attributedDict으로부터 attributedString만들어서 반환

case .br:
- 전달받은 attributedDict으로부터 줄바꿈문자 attributedString만들어서 반환

### 기존 애플코드와 성능 비교

#### 1. 짧은 문자 여러번 파싱

- `기본<br/><font color='#FA2828'>양창<font color='#0accac'>준</font></font>입<b>니다</b>라아아아아아아아아아아아기본`

| 횟수 | new | old |
| --- | --- | --- |
| 100 | 0.012초 | 1.25초 |
| 1000 | 0.08초 | 15초 |
| 10000 | 0.8초 | 15초 |

#### 2. 큰 문자 1번 파싱

- `기본<br/><font color='#FA2828'>양창<font color='#0accac'>준</font></font>입<b>니다</b>라아아아아아아아아아아아기본.... ...`

|  | new | old |
| --- | --- | --- |
|  | 0.6초 | 0.2초 |

### 문제점

- br예외처리에 대한로직이 깔끔하지 않음
- 모든 문자열에서부터 어휘분석을하기때문에  전체 문자에서 '<'찾고 나머지우측에대해 반복하는 과정이 비효율적
- 어휘분석을 통해 만들어진 파스트리를 다시 순회하면서 attributedString을 만드는 과정이 비효율적


### 개선 방향

#### 1. 파싱 순서

- 어휘분석 -> 구분분석 -> 파스트리 -> attributedString 에서 파스트리가 없어도 가능
- 어휘분석 -> 구분분석 -> attributedString 구조로 진행

#### 2. 어휘분석

- 전체 문자열에서 토큰을 구하는게 아닌 입력받은 문자는 Strem으로 넣고 필요할때 Strem에서 정해진 크기단위로 버퍼로 꺼내와 파싱하는 방식으로 변경
- 오토마타를 설계하여 각각의 문자에대해 1번만 순회하면서 파싱

#### 3. 구문 분석

- 어휘분석으로부터 이벤트를 전달받을때마다 attributedString을 만들어 최종 result에 append
- 파싱끝나면 result 반환


# TO - BE

### 참고 자료

### 오토마타를 이용한 XML파서 만들기
http://ddmix.blogspot.com/2015/06/cppalgo-30-automata-xml-parser.html
https://drive.google.com/file/d/0BzNvN1OCjO-fNzRrdU9aWmNWaWc/view

<img src="/assets/media/iOS/parser1.png">
<img src="/assets/media/iOS/parser2.png">

HTML tag EBNF
```
tag-open := '<' tag-name ws* attr-list? ws* '>'
tag-empty := '<' tag-name ws* attr-list? ws* '/>'
tag-close := '</' tag-name ws* '>'


attr-list := (ws+ attr)*
attr := attr-empty | attr-unquoted | attr-single-quoted | attr-double-quoted

attr-empty := attr-name
attr-unquoted := attr-name ws* = ws* attr-unquoted-value
attr-single-quoted := attr-name ws* = ws* ' attr-single-quoted-value '
attr-double-quoted := attr-name ws* = ws* " attr-double-quoted-value "

tag-name := (alphabets | digits)+                      # Can digits become first letter?
attr-name := /[^\s"'>/=\p{Control}]+/

# These three items should not contain 'ambiguous ampersand'...
attr-unquoted-value := /[^\s"'=<>`]+/
attr-single-quoted-value := /[^']*/
attr-double-quoted-value := /[^"]*/

alphabets := [a-zA-Z]
digits := /[0-9]/
ws := /\s/
```


## Simple HTML Parser ActionTable


htmlToken
- tagName: String
- attributes: Dict

callBackMethod
- startTagWithToken(htmlToken)
- parsedText(text)
- endTagWithToken(htmlToken)

attributeBuilder
- startTagWithToken(htmlToken)
- stack = root는 defaultFont AttrDict
- stackItem = (htmlToken : attributeDict)
- stack에 푸쉬  / stackItem = (htmlToken : attributeDict)
- endText(text)
- stack.topItem.attr 입혀서 text에 적용 후 result에 반영
- endTagWithToken(htmlToken)
- stack.pop / tagName일치하는지 유효성 검사

- 스트링 끝난순간 stack의 값 확인해서 유효성검사 > text의 경우 전달


// 전역변수
- text: String
- token: htmlToken
- attribute: (String : String)

| State | Input | Action | Trans |
| --- | --- | --- | --- |
| Initial | < | endText(text) <br/>  text.clear() | StartOpenTag |
| Initial | * | text += ch |Initial | 
| StartOpenTag | / |  | CollectCloseTag |
| StartOpenTag | * | token.tagName += ch | CollectOpenTag |
| CollectOpenTag | * <br/> (font,br,b,u) | token.tagName += ch | CollectOpenTag |
| CollectOpenTag | > | delegate.startTagWithToken(token) <br/> token.clear() <br/>  | Initial |
| CollectOpenTag | /> | delegate.startTagWithToken(token) <br/> delegate.endTagWithToken(token) <br/> token.clear() | Initial |
| CollectOpenTag | ws |  | CollectAttributes |
| CollectCloseTag | * <br/> (font,br,b,u) | token.tagName += ch | CollectCloseTag |
| CollectCloseTag | > | delegate.endTagWithToken(token) <br/> token.clear() | Initial |
| CollectAttributes | = |  | StartAttrValue |
| CollectAttributes | * <br/> (color, size) | attribute.key += ch | CollectAttributes |
| CollectAttributes | > | delegate.startTagWithToken(token) <br/> token.clear() <br/> attribute.clear() | Initial |
| CollectAttributes | /> | delegate.startTagWithToken(token) <br/> token.clear() <br/> attribute.clear() | Initial |
| StartAttrValue | ' |  | CollectAttrValue |
| StartAttrValue | " |  | CollectAttrValue |
| CollectAttrValue | *<br/> (skip#, digit+) | attribute.value += ch | CollectAttrValue |
| CollectAttrValue | ' | token.addAttribute(attribute) <br/> attribute.clear() | RemoveWhiteSpace |
| CollectAttrValue | " | token.addAttribute(attribute) <br/> attribute.clear()| RemoveWhiteSpace |
| RemoveWhiteSpace | \s |  | RemoveWhiteSpace |
| RemoveWhiteSpace | * | attribute.key += ch | CollectAttributes |
|  |  |  |  |

### Action Table 코드 매핑

- switch문이아닌 dictionary를 이용해 액션테이블을 코드로 그대로 옮기는 행위
- 리더빌리디 향상
- 예를들어 아래  collectTagNameFontF 같은 경우  Font의 'F'이고, 다음으로 올 수 있는 문자는 대소문자 'O' 뿐이다.
- 리더빌리티를 위해 lowercase같은 작업을하지않고 매핑테이블에 대소문자를 넣었음


``` swift 
typealias TableType = [String : SimpleHTMLParserStateV2]

static let collectText: TableType = ["<" : SimpleHTMLParserStateV2.collectStartingTagName]

static let collectStartingTagName: TableType = ["/" : SimpleHTMLParserStateV2.collectCloseTagName,
"f" : SimpleHTMLParserStateV2.collectTagNameFontF,
"F" : SimpleHTMLParserStateV2.collectTagNameFontF,
"b" : SimpleHTMLParserStateV2.collectTagNameB,
"B" : SimpleHTMLParserStateV2.collectTagNameB,
"u" : SimpleHTMLParserStateV2.collectTagNameU,
"U" : SimpleHTMLParserStateV2.collectTagNameU,
" " : SimpleHTMLParserStateV2.collectStartingTagName]

static let collectTagNameFontF: TableType = ["o" : SimpleHTMLParserStateV2.collectTagNameFontO,
"O" : SimpleHTMLParserStateV2.collectTagNameFontO]

static let collectTagNameFontO: TableType = ["n" : SimpleHTMLParserStateV2.collectTagNameFontN,
"N" : SimpleHTMLParserStateV2.collectTagNameFontN]

static let collectTagNameFontN: TableType = ["t" : SimpleHTMLParserStateV2.collectTagNameFontT,
"T" : SimpleHTMLParserStateV2.collectTagNameFontT]
```

## Stream 리더

https://ko.wikipedia.org/wiki/UTF-8

Sax파서를 만들기위해서는 Stream단위 문자열 처리가 필수

* Stream만들기
* String -> Data -> Stream
* UTF8 인코딩
* Stream 읽기
* Stream -> Uint8 Array (스트림은 무조건 uint8 Array로만 받아올 수 있음)
* UInt8 : (0~255) 를 가짐
* UTF8 은 최대 UInt8 4개의 조합으로 구성
* UInt8 -> 비트로 바꾸어서 아래 조건에 따라 범위 산출
* 뒤에올 문자열 길이 정해지면 얽어드림

<img src="/assets/media/iOS/parser3.png">


#### 이슈

* 코드의 스트링을 인코딩해서 Stream에 저장할경우`\'`, `\"` 등 문자열에 자동으로 들어가 `\`를 구분해줘야함

### hex to utf8 logic

<img src="/assets/media/iOS/parser4.png">


## 성능 비교

### 기존애플 vs Dom파서 vs Sax파서

#### 1\. 짧은 문자 여러번 파싱

* `기본<br/><font color='#FA2828' size='20'>양창<font color='#0accac'>준</font></font>입<b>니다</b>라아아아아<u>아아아아아아아</u>기본`

단위. 초

| 반복 | apple | Dom | Sax | SaxV2 (매핑테이블방식) |
| --- | ----- | --- | --- | --------------- |
| 100 | 0.9 | 0.036 | 0.03 | 0.035 |
| 1000 | 15 | 0.35 | 0.29 | 0.33 |
| 10000 | - | 3.5 | 2.9 | 3.3 |

#### 2\. 긴 문자 1번 파싱

* 위으 짧은문자열 반복하는 긴 스트링

|  | apple | Dom | Sax | SaxV2 (매핑테이블방식) |
| --- | ----- | --- | --- | --------------- |
|  | 0.36 | 2.8 | 0.22 | 0.26 |


## TestCase

정상 케이스
* [x] nil
* [x] ```""```
* [x] ```양창준abc❌❍❎``` : 특수문자
* [x] ```a <br/> b```, ```a <br> b```, ```a </br> b``` : br태그 2가지형태
* [x] ```양 <u> 창준 </u>``` : U태그 > 기본
* [x] ```양 <U> 창준 </u>``` : U태그 > 대소문자
* [x] ```양 <u> <u>창준 </u></u>``` : U태그 > 중첩
* [x] ```양창 <u></u>``` : U 태그 > 빈문자
* [x] ```양 <b> 창준 </b>``` : B태그 > 기본
* [x] ```양 <b><b> 창준 </b></b>``` :  B태그 > 중첩
* [x] ```양창 <b></b>``` :  B태그 > 빈문자
* [x] ```양 <B> 창준 </b>``` : B태그 > 대소문자
* [x] ```abc <font> 양창준 </font>``` : Font 태그 > 기본
* [x] ```abc <font></font>``` Font 태그 > 빈문자
* [x] ```abc <font color='#FA2828'> 양창준 </font>``` Font 태그 > color속성
* [x] ```abc <font COLor='#FA2828'> 양창준 </fONt>``` : 대소문자 Font 태그 > 대소문자
* [x] ```abc <font color='#FA2828' size='15'> 양창준 </font>``` : Font 태그 > color, size 속성
* [x] ```abc <font color='#FA2828' size='15'> 양창준abc❌<br>❍❎ </font>``` : Font, 특수문자,  br
* [x] ```abc<br/><font color='#FA2828' size='20'>양창<font color='#0accac'>준</font></font>입<b>니다</b>라아아아아<u>아아아아아아아</u>기본``` : Font 태그 > 중첩


오류케이스

* [x] ``` a <u>bcd</u></u> ``` : 닫는 태그가 더 많음
* [x] ``` a<b><b>b</b> ``` : 열림태그가 더 많음
* [x] ``` a<u color='123123'> b</u> ``` : u에 속성 사용됨
* [x] ``` a <b size='21'>bc</b> ```: b에 속성 사용됨
* [x] ``` a <b> bc </u> ``` : 태그네임다름
* [x] ``` a<font family='normal'>abc</font> ```: 정의안된 속성 사용
