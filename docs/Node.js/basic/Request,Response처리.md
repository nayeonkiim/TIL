---
id: Node2
title: Request, Response 처리
---
_[Node.js 웹개발로 알아보는 백엔드 자바스크립트의 이해](https://www.inflearn.com/course/node-js-%EC%9B%B9%EA%B0%9C%EB%B0%9C)을 보고 학습한 내용을 정리한 것 입니다._

## Post 방식으로 서버에서 처리하기
- form method="post" 로 post 전송을 한다.
* form.html
```html
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <form action="/email_post" method="post">
        email : <input type="text" name="email"><br />
        <input type="submit">
    </form>
</body>
</html>
```

* app.js
    - post 방식인 경우 req.param('email') 처럼 url로 간 데이터를 가져다 쓸 수 없다.
    - **bodyParser** 필요
        - ```app.use(bodyParser.json())```
            - 클라이언트로부터 오는 응답이 json인 경우
        - ```app.use(bodyParser.urlencoded({ extended: true }))```
            - json 아니면 인코딩으로 처리한다.
    - **req.body** 는 post로 들어온 정보이다. Object 타입 
        - 즉 form으로부터 들어온 정보
        ```js
        var bodyParser = require('body-parser')

        app.use(bodyParser.json())
        app.use(bodyParser.urlencoded({ extended: true }))

        app.post('/email_post', function (req, res) {
            console.log(req.body.email);
            res.send("<h1>welcome " + req.body.email + "</h1>");
        })
        ```
        - { email: 'wkdrnsldk98@naver.com' } 
            - form.html 의 email 입력창에 'wkdrnsldk98@naver.com'을 입력한 경우 console에 위의 내용이 출력된다.
    

## View engine을 활용한 응답처리
- **ejs** 라는 View engine을 통해 html에서 data를 활용할 수 있다.<br/><br/>
- +) ejs 설치하기
    - npm install ejs --save
- form.html로 들어온 data html에서 출력해보기
    - app.js
    ```js
    //view engine으로 ejs를 사용하겠다.
    app.set('view engine', 'ejs')

    app.post('/email_post', function (req, res) {
        console.log(req.body);
        //email.ejs 에 키 email에 req.body.email 값 객체를 넘겨준다.
        res.render('email.ejs', { 'email': req.body.email })
    });
    ```
    - email.ejs
    ```html
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <title>email ejs template</title>
    </head>

    <body>
    <h1>Welcome !! <%= email %></h1>
    <p>정말로 반가워요</p>
    </body>

    </html>
    ```
    <img src="/img/34.JPG" width="500px" height="100px" title="table1" alt="상속매핑"></img><br/>

## JSON 활용한 Ajax 처리
- 브라우저 새로고침 없이 xml httpServlet request로 데이터를 보낼 수 있다.<br/><br/>

* XMLHttpRequest
    - 서버와 상호작용하기 위하여 사용된다.
    - 전체 페이지의 새로고침 없이도 URL로 부터 데이터를 받아올 수 있다.
    - AJAX 프로그래밍에 주로 사용된다.

* 브라우저에 보낼 때도 json, 서버로 응답 받을 때도 json으로 실습 해본다.    
- form.html
    - xhr.send(data); : 서버로 요청을 보냄
    - xhr.addEventListener('load', function () { 부터 서버로부터 응답을 받은 후를 의미한다.
    ```html
    <body>
        <form action="/email_post" method="post">
            email : <input type="text" name="email"><br />
            <input type="submit">
        </form>

        <button class="ajaxsend">ajaxsend</button>

        <div class="result"></div>

        <script>
            //제공한 선택자 또는 선택자 뭉치와 일치하는 문서 내 첫번째 Element 반환
            document.querySelector('.ajaxsend').addEventListener('click', function (){
                //첫번째 form의 첫 input에 value값
                var inputdata = document.forms[0].elements[0].value;
                sendAjax('http://localhost:3000/ajax_send_email', inputdata);
            })

            function sendAjax(url, data) {
                var xhr = new XMLHttpRequest();
                var data = { "email" : data };

                data = JSON.stringify(data)

                //요청을 초기화함.
                xhr.open('POST', url);
                //HTTP 요청 헤더의 값을 설정함.
                xhr.setRequestHeader('Content-Type', "application/json");
                //요청을 보낸다. 요청이 비동기인 경우 이 메소드는 요청이 보내진 즉시 반환한다.
                xhr.send(data);

                //'load','progress','error','abort' 가 존재.
                //'load' : 요청에 의한 응답이 끝난 경우.
                xhr.addEventListener('load', function () {
                    var result = JSON.parse(xhr.responseText);
                    if(result.result !== "ok") return;
                    //document.querySelector(".result") : class="result" 인 Element return
                    document.querySelector(".result").innerHTML = result.email;
                });
            }
        </script>
    </body>
    ```
- app.js
    - res.json(responseData); 으로 html로 다시 responseData 값을 보낸다.
    ```js
    ...
    app.post('/ajax_send_email', function (req, res){
        //check validation about input value => select db
        var responseData = {'result' : 'ok', 'email' : req.body.email};
        res.json(responseData);
    });
    ```
- 정상적으로 email 값이 나온다.
    <img src="/img/35.JPG" width="200px" height="100px" title="table1" alt="상속매핑"></img><br/>