---
id: Node2
title: Request, Response 처리
---

## Post 방식으로 서버에서 처리히기
* form.html
```
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
    - req.body 로 post로 들어온 정보이다. Object 타입 
        - 즉 form으로부터 들어온 정보
```
var bodyParser = require('body-parser')

app.use(bodyParser.json())
app.use(bodyParser.urlencoded({ extended: true }))

app.post('/email_post', function (req, res) {
    console.log(req.body);
    res.send("<h1>welcome " + req.body.email + "</h1>");
})
```

