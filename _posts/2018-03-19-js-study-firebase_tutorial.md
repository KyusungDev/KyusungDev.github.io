---
layout: post
title: "Firebase 사용방법"
tags: [firebase]
---

2018.03.19
firebase Realtime Database 사용방법을 정리한 글입니다.

- firebase 가입
[firebase](https://firebase.google.com)

- firebase realtime DB 생성  
자신의 콘솔로 이동하여 새 프로젝트를 추가하고, 프로젝트에 들어가서 Database 메뉴를 선택한다.  

**규칙 메뉴에서 모든 사용자가 접근 가능하도록 변경한다.**
```js
{
  "rules": {
    ".read": true, // true로 변경
    ".write": true // true로 변경
  }
}
```

- 데이터 메뉴에 하위 항목 추가
  - 추가버튼을 선택하여 하위 항목을 한다 
    ```js
        lists: "" // 이름 lists, 값 ""
    ```

    lists 항목을 클릭하면 화면 상단에 아래와 같은 URL이 표시된다.
    ```
    https://js-study.firebaseio.com/lists
    ```

    URL을 복사하고 해당 URL 끝에 **.json**을 붙이면 해당 데이터를 조회하거나 수정할 수 있다.  
    ```
    https://js-study.firebaseio.com/lists.json
    ```

- 해당 DB에 대해서 테스트
  - Postman으로 테스트하기
    크롬 앱에서 Postman 설치 [Postman](https://www.getpostman.com/apps)  
    + GET  
      GET 선택 후 URL 입력후 Send

    + PUT  
      PUT 선택 후 URL 입력 후 Headers 메뉴에서 Content-Type을 **application/json** 으로 변경  
      Body 메뉴에서 raw 선택 후 type을 **JSON(application/json)**으로 변경  
      json 형식으로 데이터 작성 후 Send  
      
  - 코드로 테스트하기  
    XMLHttpReqeust를 이용해 GET/PUT하는 예제 참고.  
    추가로 구현하려면  GET/POST/PUT/DELETE 예제가 작성된 코드를 참고.  
    + [코드로 테스트하기](https://jsbin.com/sugipawuyu/edit?html,js,console,output)   
    + [GET/POST/PUT/DELETE 코드 구조 참조](https://gist.github.com/EtienneR/2f3ab345df502bd3d13e)  

***
- [JSON 사용하기](https://www.w3schools.com/js/js_json_intro.asp)
- [Firebase REST API 사용 방법](https://firebase.google.com/docs/reference/rest/database/)  
- [AJAX 사용하기](https://developer.mozilla.org/ko/docs/Web/Guide/AJAX/Getting_Started)
- [REST API란 무엇인가](http://blog.naver.com/PostView.nhn?blogId=complusblog&logNo=220986337770)
- [REST API 제대로 알고 사용하기](http://meetup.toast.com/posts/92)
  
