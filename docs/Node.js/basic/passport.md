---
id: Node5
title: PASSPORT
---
_[Node.js 웹개발로 알아보는 백엔드 자바스크립트의 이해](https://www.inflearn.com/course/node-js-%EC%9B%B9%EA%B0%9C%EB%B0%9C)을 보고 학습한 내용을 정리한 것 입니다._

### passport
- 인증을 편하게 관리하기 위한 패키지이다.
- passport 가 실제로 하는 일은 session 객체 내부에 passport 프로퍼티를 만들고 값으로 쿠키와 식별자를 매칭해서 저장한다.(serialize)
- 이 후 매 요청 시에 세션에 저장된 식별자를 이용해 유저의 데이터를 찾아 express 라우터 콜백함수의 request.user에 해당 데이터를 저장한다.(deserialize)
- username과 password 기반 사용자 인증을 위해 사용된다.
- Strategy를 정해줘야 하며 local 로그인의 경우는 LocalStrategy이다.

### 설치하기
- `npm install passport passport-local express-session connect-flash --save-dev`
    - passport : 인증 
    - passport-local : 로컬 로그인 처리
    - express-session : 로그인 후 user 정보를 세션에 저장처리
    - connect-flash : 에러 메시지 redirect 처리

### 설정
- 아래의 설정을 모두 app.js에 해준다.

- app.js에 passport 사용 선언을 해준다.
    ```js
    var passport = require('passport')
    var LocalStrategy = require('passport-local').Strategy
    var session = require('express-session')
    var flash = require('connect-flash')
    ```
- session 활성화
    ```js
    app.use(session({
        resave: false,
        saveUninitialized: true,
        secret: 'keyboard cat',
    }))
    ```
- passport 구동 : passport를 미들웨어로 사용하겠다. 
    - `app.use(passport.initialize());`
- 세션 연결 : req.session에 passport 정보를 저장하겠다.
    - `app.use(passport.session());`
- 에러메시지 redirect 모듈 사용하겠다.
    - `app.use(flash());`
    

### 회원가입 로직 구성하기
```java
var passport = require('passport')
var LocalStrategy = require('passport-local').Strategy;

passport.use(new LocalStrategy({
    usernameField: 'email',
    passwordField: 'pw',
    passReqToCallback: true
}, function (req, email, pw, done) {
    var query = connection.query('select * from user where email=?', [email], function (err, rows) {
        if (err) return done(err);

        if (rows.length) {
            console.log('exisited user')
            console.log(rows);
            return done(null, false, { message: 'your email is already used' })
        } else {
            var sql = { email: email, pw: pw };
            var query = connection.query('insert into user set ?', sql, function (err, rows) {
                if (err) throw err

                return done(null, { 'email': email, 'id': rows.insertId })
            })
        }
    })
}
));
```
- passport와 LocalStrategy 사용을 위해 선언해준다.
- passport.use() 
    - 함수에서 Strategy와 설정들을 정의한다.
    - `usernameField` 과 `passwordField` 는 어떤 폼 필드로 부터 아이디와 비밀번호를 전달받을 지 에 대한 설정
    - passReqToCallback 이 true일 경우 뒤의 콜백의 req 매개변수를 통해 express의 req 에 접근할 수 있게 된다.<br/>
    `function (req, email, pw, done) {`
    - 아이디와 비밀번호 값이 들어와서 함수가 실행되고
    위의 쿼리는 아이디값으로 사용된 이메일과 동일한 이메일을 db에서 찾아서 select 해준다.<br/>
    `var query = connection.query('select * ~ )`
    - 비교과정에서 서버에 에러가 나면 첫번째 if문의 `done(err);`를 리턴한다.
    - `if (rows.length)` 의미는 이미 처리된 결과가 있는 경우로 이미 해당 email을 가진 회원이 존재하므로 `done(null, false, { message: 'your email is already used' })` 으로 실패의 의미인 false와 message를 넘겨준다.
    - err도 없고 email을 가진 회원도 없는 경우 마지막 else문으로 들어와서 쿼리를 통해 회원으로 등록을 시켜주고 `done(null, { 'email': email, 'id': rows.insertId })` 을 리턴한다.

    - +) 아이디와 비밀번호 이외의 이름과 같은 값도 함께 insert 하고 싶으면
        - req.body.name, req.query.name 으로 해당 값을 불러와서 넣어주면 된다.
        ```js
        var paramName = req.body.name || req.query.name;

        var sql = { email: email, pw: password, name: paramName };
        var query = connection.query('insert into user set ?', sql, 
        ...
        ```


### 인증 콜백 -  done()
- 인증이 유효한 경우
    - `return done(null, user);`
- 인증이 유효하지 않은 경우
    - ex ) 이미 존재하는 email인 경우
    - `return done(null, false);`
    - `return done(null, false, { message: 'Incorrect password.' });`
        - 실패 이유를 알려주는 추가적인 메세지 정보도 적을 수 있다.
        - 실패 시 redirect 되는 페이지에서 <%=message%> 를 통해 해당 정보를 출력할 수 있다.<br/><br/>
        <img src="https://github.com/nayeonkiim/TIL/blob/master/docs/Node.js/basic/img/inuse.JPG?raw=true" width="350px" height="200px" title="table1" alt="값타입"></img><br/> 

- 인증을 판별할 때 exception이 발생한 경우
    - `return done(err);`

### 인증
```js
router.post('/', passport.authenticate('local', {
    successRedirect: '/main',
    failureRedirect: '/join',
    failureFlash: true
}))

module.exports = router;
```
- post로 '/'로 들어온 경우 LocalStrategy 를 사용해서 passport에서 local에 대한 인증 작업을 시작한다.
- 위에서 정의해둔 `passport.use(new LocalStrategy({ ... ` 를 이용해 인증 작업을 한다.
- 인증 작업이 성공할 경우 '/main'으로 redirect
- 인증 작업이 실패할 경우 '/join'으로 redirect
- `failureFlash: true` 를 통해 위의 Strategy 인증 callback 에서의 error 메시지를 redirect 하는 곳으로 보낸다. 
- `module.exports = router;`  : module로 등록도 마지막에 해준다.

### 세션에 저장처리
- **serializeUser** 
    - 회원가입 성공 시 실행되었던<br/>
    done(null, { 'email': email, 'id': rows.insertId }) 에서 객체형태의 값을 전달받아 세션(**req.session.passport.user**)에 저장한다.
    - done(null, user.id)
        - 첫 번째 인수는 에러 발생 시 사용하는 것이고, 두 번째 인수는 저장하고 싶은 데이터를 넣는다.
        - 즉, 사용자의 아이디만 저장하라고 명령한 것이다.
    
        ```js
        passport.serializeUser(function(user, done) {
            done(null, user.id);
        });
        ```
- **deserializeUser**
    - 실제 서버로 들어오는 **요청마다** 세션 정보를 실제 DB의 데이터와 비교한다.
    - 해당하는 유저 정보가 있으면 done의 두번째 인자를 **req.user**에 저장하고 요청을 처리할 때 유저의 정보를 **req.user**를 통해서 넘겨준다.
    - User를 통해 findById를 해주는 세션 정보를 실제 DB의 데이터와 비교하려면 db에 대한 정의를 해줘야한다. 아래의 코드는 mongoDB의 경우이다.
    - **serializeUser 에서 done으로 넘겨주는 두번째 인자(user.id)가 deserializeUser의 첫번째 매개변수(id)로 전달된다.**<br/><br/>
    ```js
    passport.deserializeUser(function(id, done) {
        User.findById(id, function(err, user) {
            done(err, user);
        });
    });
    ```
    - db 데이터 비교 없이 그냥 유저 정보를 넘겨주도록 구현했다.
    ```js
    //join/index.js
    passport.deserializeUser(function (id, done) {
        console.log('passport session get id : ', id)
        done(null, id);
    });
    ```
    ```js
    //main.js
    router.get('/', function (req, res) {
        console.log('main js loaded', req.user)
        res.sendFile(path.join(__dirname, "../../public/main.html"))
    });
    ```
    - id는 사용자를 찾는 데 사용되며 **req.user**로 접근 가능하다.
        - 위에서 인증 성공시 main 으로 redirect를 했으므로 (`successRedirect: '/main'`)<br/>
        main.js에서 `console.log('main js loaded', req.user)` 로 콘솔에 출력
        - `[main js loaded 20]` : id(20)가 출력되는 것을 알 수 있다.
- 정리
    - serializeUser 는 사용자 정보를 세션에 저장하는 것
    - deserializeUser 는 세션에 저장한 정보를 통해 사용자 정보 객체 불러오는 것이다.

### 로그인 구현
- 로그인 기능도 위와 유사하게 구현하면 된다.
-  페이지로 전달시 **AJax를 통해 JSON 형태로 응답**을 해줄때 **Custom Callback**을 사용한다. 
- 위의 경우 redirect 기반이다.

```js
passport.use('local-login', new LocalStrategy({
    usernameField: 'email',
    passwordField: 'password',
    passReqToCallback: true
}, function (req, email, password, done) {
    var query = connection.query('select * from user where email=?', [email], function (err, rows) {
        if (err) return done(err);

        if (rows.length) {
            return done(null, { 'email': email, 'id': rows[0].uid })
        } else {
            return done(null, false, { 'message': 'your login info is not found' })
        }
    })
}
));

router.post('/', function (req, res, next) {
    passport.authenticate('local-login', function (err, user, info) {
        if (err) res.status(500).json(err);
        if (!user) return res.status(401).json(info.message);

        req.logIn(user, function (err) {
            if (err) { return next(err); }
            return res.json(user);
        });
    })(req, res, next);
});
```
- client로 부터 넘어온 데이터가 user로 들어온다.
- authenticate() 가 미들웨어로 사용되지 않고 경로 핸들러 내에서 호출된다. -> 클로저를 통해 req, res 객체에 콜백 접근 가능해진다.
- 인증에 실패하면 user가 false로 셋팅되고 exception이 발생하면 err가 셋팅된다.
- `info` : strategy의 입증 콜백에 의해 제공받은 추가적인 사항을 포함하는 인수이다.
- req.logIn() 처리 후 serializeUser, deserializeUser로 이어진다.
- `(req, res, next);` : authentication에서 반환하는 메서드에 req, res, next 인자를 넣어주어 추가적인 처리를 한다. -> 콜백<br/><br/>

- +) view 부분은 [ajax 처리](http://localhost:3000/docs/Node.js/basic/Node2#json-%ED%99%9C%EC%9A%A9%ED%95%9C-ajax-%EC%B2%98%EB%A6%AC) 참고

### 로그아웃 구현
```js
//logout/index.js
var express = require('express');
var app = express();
var router = express.Router();

router.get('/', function (req, res) {
    req.logout();
    res.redirect('/login');
});

module.exports = router;
```
- 세션 정보가 지워지고 login으로 redirect 된다.<br/><br/>

## 로그인과 로그인 된 이후 과정
- 로그인 
1. 로그인 요청이 들어옴
2. 라우터에서 **passport.authenticate** 메서드 호출
3. 로그인 전략 수행
4. 로그인 성공 시 사용자 정보 객체와 함께 **req.login** 호출
5. req.login 메서드가 **passport.serializeUser** 호출
6. **req.session** 에 사용자 아이디만 저장
7. 로그인 완료

- 로그인 된 이후 과정
1. 요청이 들어옴
2. 라우터에 요청이 도달하기 전에 **passport.session** 미들웨어가 **passport.deserializeUser** 메서드 호출
3. req.session 에 저장된 아이디로 데이터베이스에서 사용자 조회
4. 조회된 사용자 정보를 **req.user**에 저장
5. 라우터에서 req.user 객체 사용 가능


## 권한 설정
- 로그인한 사용자는 회원가입과 로그인 라우터에 접근하지 못하도록 한다.
- 로그인이 안된 사용자 역시 로그아웃 라우터에 접근하지 못하도록 한다.
- 위와 같은 설정을 위해 권한 설정이 필요하다. 접근 권한을 제어하는 미들웨어가 필요하다.
    - Passport에서 req 객체에 추가해주는 **isAuthenticated**메서드
    - 로그인 중이면 req.isAuthenticated() 가 true
    - 그렇지 않으면 req.isAuthenticated() 가 false

    ```js
    //middlewares.js
    exports.isLoggedIn = (req, res, next) => {
    if (req.isAuthenticated()) {
            next();
        } else {
            res.status(403).send('로그인 필요');
        }
    };

    exports.isNotLoggedIn = (req, res, next) => {
        if (!req.isAuthenticated()) {
            next();
        } else {
            const message = encodeURIComponent('로그인 한 상태입니다.');
            res.redirect(`/?error=${message}`);
        }
    }
    ```
    - 사용
        - 로그인이 안되었을 때만 로그인 라우터에 접근 가능하다.
        ```js
        router.post('/login', isNotLoggedIn, (req, res, next) => {
            ...
        }
        ```

### 참고)
- [passport document](http://www.passportjs.org/docs/authenticate/)
- https://www.zerocho.com/category/NodeJS/post/57b7101ecfbef617003bf457
- [Node.js 교과서](http://www.yes24.com/Product/Goods/62597864)