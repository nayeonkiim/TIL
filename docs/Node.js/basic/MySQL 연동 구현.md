---
id: Node3
title: MySQL 연동 구현
---
_[Node.js 웹개발로 알아보는 백엔드 자바스크립트의 이해](https://www.inflearn.com/course/node-js-%EC%9B%B9%EA%B0%9C%EB%B0%9C)을 보고 학습한 내용을 정리한 것 입니다._

## MySQL과 연동
1. MySQL 모듈 설치
`npm install mysql`

2. MySQL 데이터베이스 생성, 테이블 생성
- 데이터베이스 생성 : `create database jsman`
- 테이블 생성 
    ```sql
    create table user(
        uid int not null primary key auto_increment,
        email varchar(255) not null,
        name varchar(50) not null,
        pw varchar(100) not null
        )
    ```
- 데이터 추가 : `insert into user values ('adb@naver.com','haha', '1133')`

3. MySQL 연동
```js
//app.js
var mysql = require('mysql');
var connection = mysql.createConnection({
  host     : 'localhost',
  port     : 3306,
  user     : 'root',
  password : '1111',
  database : 'pract'
});

connection.connect();
```

4. MySQL에서 데이터 찾기
```js
//emai.js
app.post('/ajax_send_email', function (req, res){
    var email = req.body.email;
    var responseData = {};

    var query = connection.query('select name from user where email = "' + email +'"', function(err, rows) {
        if(err) throw err;
        if(rows[0]) {
            console.log(rows[0].name)
            responseData.result = "ok";
            responseData.name = rows[0].name;
        }else {
            responseData.result="none";
            responseData.name="";
        }
        res.json(responseData);
    })
})
```
- client로 부터 들어온 데이터인 email에 대한 name을 select
- client로 부터 들어온 데이터의 name이 존재하는 경우 rows[0] 이 true
- ajax 통신으로 값을 responseDate에 넣어 다시 client로 보내준다.

- [쿼리 단순화 URL](https://github.com/mysqljs/mysql/#escaping-query-values)
    - 첫 번째 코드를 두 번째 코드로 쿼리를 간단하게 만드는 방법

    ```js
    var sql= "INSERT INTO user (email,name,pw) VALUES (" + email + '","' + name + '","' +   passwd + '")"';
    var query = connection.query(sql, function (err, row){
        if(err) { throw err;}
        console.log("ok db insert");
    })
    ```

    ```js
    var sql = {email : email, name : name, pw : passwd};
    var query = connection.query('insert into user set ?', sql, function (err, row){
        if(err) { throw err;}
        console.log("ok db insert");
    })
    ```
      
    
