---
layout: post
title: Jquery, Ajax 파일 업로드 만들기
categories: [개인]
tags: [정보]
description: 허접한 컴포넌트
comments: true
---

## 원하는 스펙

1. textArea에 드래그 & 드롭으로 파일 업로드 
2. 파일 추가 시 파일명 리스트 추가
2. 파일선택 버튼 변화 시 파일명 리스트업
3. 삭제 버튼 클릭 시 리스트 삭제
4. 입력 버튼 클릭 시 서버 전송

### 1. textArea에 드래그 & 드롭으로 파일 업로드 

> 
			 $(document).on("dragenter", ".dragAndDropDiv", function(e) {
                e.stopPropagation();
                e.preventDefault();
                $(this).css('background-color', 'yellow');
              });
              $(document).on("drop", ".dragAndDropDiv", function(e) {
                $(this).css('background-color', 'white');
                e.preventDefault();
                var files = e.originalEvent.dataTransfer.files; //get files
                appendFile(files, parent); 		//parent 밑에 추가
              });

### 2. 파일 추가 시 파일명 리스트 추가

> 
			  //파일 수 만큼 태그 만들어서 추가
              function appendFile(files, obj) {
                for (var i = 0; i < files.length; i++) {
                  var status = new createStatusbar(obj); //Using this we can set progress.
                  status.addList(files[i]);
                  status.setFileNameSize(files[i].name, files[i].size);
                  sendFileToServer(status);
                }
              }

              var rowCount = 0;
              function createStatusbar(obj) {
                rowCount++;

                //태그 만드는데 rowCount는 식별 id값 주기 위함
                //삭제 버튼 클릭 시 해당 div제거 해야하는데 parent에서 해당 statusbar의
                //index를 못가져와서 추가한건데 나중에 찾아보자
                this.statusbar = $("<div class='statusbar'  id='"+rowCount+"'></div>");
                this.filename = $("<div class='filename'></div>").appendTo(
                    this.statusbar);
                this.size = $("<div class='filesize'></div>").appendTo(
                    this.statusbar);
                this.abort = $("<div class='abort'>삭제</div>").appendTo(
                    this.statusbar);
                $("#parent").append(this.statusbar);

                this.setFileNameSize = function(name, size) {
                  var sizeStr = "";
                  var sizeKB = size / 1024;
                  if (parseInt(sizeKB) > 1024) {
                    var sizeMB = sizeKB / 1024;
                    sizeStr = sizeMB.toFixed(2) + " MB";
                  } else {
                    sizeStr = sizeKB.toFixed(2) + " KB";
                  }
                  this.filename.html(name);
                  this.size.html(sizeStr);
                }
                 this.addList = function(files) {		   //전송 시 최종적으로 추가,삭제된 파일리스트
                  filelist.push(files);
                }


### 3. 파일선택 버튼 변화 시 파일명 리스트업

> 
			  $("#selectFile").change(function(e) {    //파일 선택 시 
                e.preventDefault();
                var files = e.target.files;	            //여러 선택된 파일 정보가져옴
                appendFile(files, parent);		
              });


### 4. 삭제 버튼 클릭 시 리스트 삭제

>                
		    this.setAbort = function(jqxhr) { 	   //삭제버튼 이벤트
                  var sb = this.statusbar;
                  this.abort.click(function() {			   //this는 삭제버튼으로
                    var idx = $(this).parent().attr('id'); //하나의 리스트 전체를 담고있는 div get
                    $(this).parent().remove();             //삭제
                    //filelist.splice(idx-1,idx-1);
                    delete filelist[idx - 1];                 //전송을 위한 파일 리스트에서도 삭제
                    jqxhr.abort();
                  });
                }
              }

### 5. 입력 버튼 클릭 시 서버 전송
>
		    $("#send_btn").click(function(e) {
                e.preventDefault();
                //리스트에서 파일 데이터 formData에 연결 
                var formData = new FormData();
                for (var i = 0; i < filelist.length; i++) {
                  formData.append("file", filelist[i]);
                }
                formData.append("commnetIDX", "12");
                //서버로 전송
                $.ajax({
                  url : '/happyNews/file_callback',
                  processData : false,
                  contentType : false,
                  data : formData,
                  type : 'POST',
                  success : function(result) {
                    $(".parent").empty(); //자식노드 제거
                    filelist = []; 			//리스트 초기화
                    $("#selectFile").val(""); //파일선택 초기화
                  }
                });
              });


### 6. 전체 코드
>
	 <%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
	<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
	<%@ page session="false"%>
	<html>
	<head>
	<title>Test</title>
	<script type="text/javascript"
	src="http://code.jquery.com/jquery-1.11.3.js"></script>
	<style>
	.dragAndDropDiv {
	  border: 0;
	  width: 650px;
	  height: 200px;
	  color: #92AAB0;
	  text-align: center;
	  vertical-align: middle;
	  padding: 10px 0px 10px 10px;
	  font-size: 200%;
	  display: table-cell;
	}	
	.progressBar {
	  width: 200px;
	  height: 22px;
	  border: 0px solid #ddd;
	  border-radius: 5px;
	  overflow: hidden;
	  display: inline-block;
	  margin: 0px 10px 5px 5px;
	  vertical-align: top;
	}
	.progressBar div {
	  height: 100%;
	  color: #fff;
	  text-align: right;
	  line-height: 22px;
	  /* same as #progressBar height if we want text middle aligned */
	  width: 0;
	  background-color: #0ba1b5;
	  border-radius: 3px;
	}
	.statusbar {
	  border-top: 0px solid #A9CCD1;
	  min-height: 25px;
	  width: 99%;
	  padding: 10px 10px 0px 10px;
	  vertical-align: top;
	}
	.statusbar:nth-child(odd) {
	  background: #EBEFF0;
	}
	.filename {
	  display: inline-block;
	  vertical-align: top;
	  width: 250px;
	}
	.filesize {
	  display: inline-block;
	  vertical-align: top;
	  color: #30693D;
	  width: 100px;
	  margin-left: 10px;
	  margin-right: 5px;
	}
	.abort {
	  background-color: #A8352F;
	  -moz-border-radius: 4px;
	  -webkit-border-radius: 4px;
	  border-radius: 4px;
	  display: inline-block;
	  color: #fff;
	  font-family: arial;
	  font-size: 13px;
	  font-weight: normal;
	  padding: 4px 15px;
	  cursor: pointer;
	  vertical-align: top
	}
	</style>
	</head>
	<body>
	<textarea id="fileUpload" class="dragAndDropDiv"></textarea>
	<form class="parent" id="parent"></form>
	<input type="file" id="selectFile" multiple="multiple" name="selectFile"
		required="required" />
	<button id="send_btn" id="send_btn">입력</button>
	<script type="text/javascript">
    $(document)
        .ready(
            function() {
              var filelist = [];
              var objDragAndDrop = $(".dragAndDropDiv");
              var parent = $(".parent");
              $(document).on("dragenter", ".dragAndDropDiv", function(e) {
                e.stopPropagation();
                e.preventDefault();
                $(this).css('background-color', 'yellow');
              });
              $(document).on("drop", ".dragAndDropDiv", function(e) {
                $(this).css('background-color', 'white');
                e.preventDefault();
                var files = e.originalEvent.dataTransfer.files; //get files
                appendFile(files, parent); 		//parent 밑에 추가
              });
              $("#selectFile").change(function(e) { //파일 선택 시 
                e.preventDefault();
                var files = e.target.files;	            //여러 선택된 파일 정보가져옴
                appendFile(files, parent);		
              });
              $("#send_btn").click(function(e) {
                e.preventDefault();
                //리스트에서 파일 데이터 formData에 연결 
                var formData = new FormData();
                for (var i = 0; i < filelist.length; i++) {
                  formData.append("file", filelist[i]);
                }
                formData.append("commnetIDX", "12");
                //서버로 전송
                $.ajax({
                  url : '/happyNews/file_callback',
                  processData : false,
                  contentType : false,
                  data : formData,
                  type : 'POST',
                  success : function(result) {
                    $(".parent").empty(); //자식노드 제거
                    filelist = []; 			//리스트 초기화
                    $("#selectFile").val(""); //파일선택 초기화
                  }
                });
              });
              //파일 수 만큼 태그 만들어서 추가
              function appendFile(files, obj) {
                for (var i = 0; i < files.length; i++) {
                  var status = new createStatusbar(obj); //Using this we can set progress.
                  status.addList(files[i]);
                  status.setFileNameSize(files[i].name, files[i].size);
                  sendFileToServer(status);
                }
              }
              var rowCount = 0;
              function createStatusbar(obj) {
                rowCount++;
                //태그 만드는데 rowCount는 식별 id값 주기 위함
                //삭제 버튼 클릭 시 해당 div제거 해야하는데 parent에서 해당 statusbar의
                //index를 못가져와서 추가한건데 나중에 찾아보자
                this.statusbar = $("<div class='statusbar'  id='"+rowCount+"'></div>");
                this.filename = $("<div class='filename'></div>").appendTo(
                    this.statusbar);
                this.size = $("<div class='filesize'></div>").appendTo(
                    this.statusbar);
                this.abort = $("<div class='abort'>삭제</div>").appendTo(
                    this.statusbar);
                $("#parent").append(this.statusbar);
                this.setFileNameSize = function(name, size) {
                  var sizeStr = "";
                  var sizeKB = size / 1024;
                  if (parseInt(sizeKB) > 1024) {
                    var sizeMB = sizeKB / 1024;
                    sizeStr = sizeMB.toFixed(2) + " MB";
                  } else {
                    sizeStr = sizeKB.toFixed(2) + " KB";
                  }
                  this.filename.html(name);
                  this.size.html(sizeStr);
                }
                this.addList = function(files) {		   //전송 시 최종적으로 추가,삭제된 파일리스트
                  filelist.push(files);
                }
                this.setAbort = function(jqxhr) { 	   //삭제버튼 이벤트
                  var sb = this.statusbar;
                  this.abort.click(function() {			   //this는 삭제버튼으로
                    var idx = $(this).parent().attr('id'); //하나의 리스트 전체를 담고있는 div get
                    $(this).parent().remove();             //삭제
                    //filelist.splice(idx-1,idx-1);
                    delete filelist[idx - 1];                 //전송을 위한 파일 리스트에서도 삭제
                    jqxhr.abort();
                  });
                }
              }
              function sendFileToServer(status) {
                var uploadURL = "/happyNews/fileUpload_status/"; //Upload URL
                var extraData = {}; //Extra Data.
                var jqXHR = $.ajax({
                  xhr : function() {
                    var xhrobj = $.ajaxSettings.xhr();
                    return xhrobj;
                  },
                  url : uploadURL,
                  //data: formData,
                  success : function(data) {
                    status.setAbort(jqXHR);
                    console.log(filelist);
                  }
                });
              }
            });
	  </script>
	</body>
	</html>

### 7. 서버 코드

>
	@RequestMapping(value = "/file_callback")
    public void file_callback(CommandMap map, HttpServletRequest request) throws Exception {
        MultipartHttpServletRequest multipartHttpServletRequest = (MultipartHttpServletRequest) request;
        Iterator<String> iter = multipartHttpServletRequest.getFileNames();
        MultipartFile multipartFile = null;
        String uploadFileName = null, saveFileName;
        String comment_idx = request.getParameter("commnetIDX");
        System.out.println(comment_idx);
        List<MultipartFile> list = multipartHttpServletRequest.getFiles("file");
        for (int i = 0; i < list.size(); i++) {
            multipartFile = list.get(i);
            uploadFileName = multipartFile.getOriginalFilename();
            saveFileName = comment_idx + "_" + uploadFileName;
            // 중복될 경우 덮어쓰기
            if (saveFileName != null && !saveFileName.equals("")) {
                try {
                    multipartFile.transferTo(new File(uploadPath + saveFileName));
                } catch (IllegalStateException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            } 
        }
    }


### 실행 화면

**드래그로 textArea에 파일 추가**

<img src="/assets/media/fileupload/1.png">

- html5에서 지원하는 속성으로 편하게 구현 가능
- 자신이 원하는 타이밍의 이벤트 발생하면됨

**동적으로 화면에 추가된 파일 리스트 출력**

<img src="/assets/media/fileupload/2.png">

- <div id=parent/> 최상위 노드에 자식으로 div만듬
- 삭제 시 해당 div삭제 

**파일선택 버튼 클릭**

<img src="/assets/media/fileupload/3.png">

- input type을 file로 함
- multiple="multiple" 속성을 주면 여러 파일 업로드 가능
- 클리 시 가 아닌 파일선택 후 이벤트 발생 하려면 change이용

###결론

- 여기저기 참고를 많이해서 참고링크는 없지만 style외 거의 직접 짠거라 문제될 시 삭제하겠습니다.
- 파일 업로드 진행하면서 많은 공부가 되었음
- 이런 허접한거 만드는데 15시간 이상 투자한듯
- div로 파일명 리스트 추가하는데 삭제 시 부모로부터 속한 자식 `<div>`에서 삭제버튼이 속한 `<div>`의 index를 얻어오지 못해서 생성 시 id를 추가해줌 
- 코드 중 sendfileToServer() 부분만 수정하면 좋을듯... 이부분은 아직 시간이 부족해서..
- 배포되는 js파일 처럼 깔끔하고 interface화 해서 코드 수정해야 하는데 개념 부족 (나중에 꼭 해보자)
- TUI를 써야하는데 넘사벽... 백본은 무엇인가 