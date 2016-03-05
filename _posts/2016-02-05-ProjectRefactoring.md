
# 프로젝트 리팩토링 (구조 및 컨벤션)

## CommentController

### 레스트 구조

삽입 post   /comment/					매개변수
삭제 delete /comment/{commentId}
리스트 GET  /comments/{articleId}
좋아요	   /comments/{commentId}/likeup

## CommentFileController

