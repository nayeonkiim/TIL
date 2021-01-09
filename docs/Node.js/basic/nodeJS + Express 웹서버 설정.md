---
id: Node1
title: nodeJS + Express 웹서버 설정
---
_[Node.js 웹개발로 알아보는 백엔드 자바스크립트의 이해](https://www.inflearn.com/course/node-js-%EC%9B%B9%EA%B0%9C%EB%B0%9C)을 보고 학습한 내용을 정리한 것 입니다._

## NPM
- npm은 Node Package Manager의 약자로 자바스크립트 패키지 매니저이다.
- Node.js 에서 사용 가능한 패키지를 설치 및 배포할 수 있다.

### package.json
- $ npm init을 통해서 생성한다.
- npm을 이용해 패키지를 설치하기 위해서는 package.json 파일이 필요하다.
- package.json은 프로젝트의 정보와 프로젝트에서 사용 중인 패키지의 의존성을 관리하게 된다.
```
{
  "name": "nodeserver",
  "version": "1.0.0",
  "description": "nodeserver test",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "body-parser": "^1.19.0",
    "ejs": "^3.1.5",
    "express": "^4.17.1"
  }
}
```

## Express
- express란 Node.js의 핵심 모듈인 http와 Connect 컴포넌트를 기반으로 하는 웹 프레임워크다.
- 설치 : ```npm install express --save```
    - --save 는 프로젝트가 의존하고 있는 외부 라이브러리 정보를 package.json에 저장한다.

- port 3000으로 서버 띄우기
```
var express = require('express')
var app = express()

app.listen(3000, function () {
    console.log("start!!! express server on port 3000");
});
```
- ```node app.js``` : 애플리케이션 실행
 

## URL 라우팅 처리

```
app.get('/', function (req, res) {
    res.sendFile(__dirname + "/public/main.html")
})
```
- /로 get 요청 들어오면 설정된 파일을 보낸다.

- html 파일에 js파일 포함된 경우
    - js 파일도 url 처리를 해줘야 한다.
    - js와 같은 static 파일을 서버에서 요청 받는데로 처리해 주면 편리해진다.
        - ```app.use(express.static('static파일 디렉토리'))```
        - static 디렉토리 등록시켜 주기

    
