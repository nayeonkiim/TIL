---
id: Node5
title: PASSPORT
---

### passport
- 인증을 편하게 관리하기 위한 패키지이다.
- passport 가 실제로 하는 일은 session 객체 내부에 passport 프로퍼티를 만들고 값으로 쿠키와 식별자를 매칭해서 저장한다.(serialize)
- 이 후 매 요청 시에 세션에 저장된 식별자를 이용해 유저의 데이터를 찾아 express 라우터 콜백함수의 request.user에 해당 데이터를 저장한다.(deserialize)
- username과 password 기반 사용자 인증을 위해 사용된다.
- Strategy를 정해줘야 하며 local 로그인의 경우는 LocalStrategy이다.

### 설치하기
- `npm i passport passport-local bcrypt`
    - passport : 인증 
    - passport-local : 로컬 로그인 처리
    - bcrypt : 비밀번호 암호화

### 설정
- app.js에 아래의 설정을 해준다.
    - passport 사용 선언을 해준다.
        ```js
        const session = require('express-session')
        const passport = require('passport');    

        const passportConfig = require('./passport');

        passportConfig();  //패스포트 설정
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

- passport 폴더의 index.js에 **Passport 관련 코드**를 넣어준다. 
    ```js
    const passport = require('passport');
    const local = require('./localStrategy');
    const User = require('../models/user);

    module.exports = () => {
        passport.serializeUser((user, done) => {
            done(null, user.id);
        });

        passport.deserializeUser((id, done) => {
            User.findOne({ where: {id} })
            .catch(err => done(err));
        });

        local();
    };
    ```
    - **serializeUser**
        - **로그인 시 실행**되며, **req.session** 객체에 어떤 데이터를 저장할지 정하는 메서드이다.
        - done 함수의 첫 번째 인수는 **에러 발생** 시 사용하는 것이고, 두 번째 인수에는 **저장하고 싶은 데이터**를 넣는다.
        - 즉, done(null, user.id); 는 세션에 사용자의 아이디만 저장하라고 명령한 것이다.

    - **deserializeUser**
        - 매 요청시 실행된다.
        - passport.session 미들웨어가 이 메서드를 호출한다.
        - serializeUser의 done 의 두 번째 인수로 넣었던 데이터가 deserializeUser의 매개변수가 된다.
        - 그리고 여기서 serializeUser에서 세션에 저장했던 아이디를 받아 데이터베이스에서 사용자 정보를 조회한다.
        - 조회한 정보를 **req.user**에 저장하여 req.user를 통해 로그인한 사용자의 정보를 가져올 수 있다.

## 로그인과 로그인 된 이후 과정
- 로그인 
1. 로그인 요청이 들어옴
2. 라우터에서 **passport.authenticate** 메서드 호출
3. 로그인 전략 수행
4. 로그인 성공 시 사용자 정보 객체와 함께 **req.login** 호출
5. req.login 메서드가 **passport.serializeUser** 호출
6. **req.session** 에 사용자 아이디만 저장
7. 로그인 완료<br/><br/>

- 로그인 된 이후 과정
1. 요청이 들어옴
2. 라우터에 요청이 도달하기 전에 **passport.session** 미들웨어가 **passport.deserializeUser** 메서드 호출
3. req.session 에 저장된 아이디로 데이터베이스에서 사용자 조회
4. 조회된 사용자 정보를 **req.user**에 저장
5. 라우터에서 req.user 객체 사용 가능


## 로컬 로그인 구현
- 로컷 로그인이란 자체적으로 회원가입 후 로그인하는 것을 의미한다.
- 로컬 로그인을 구현하기 위해서는 passport-local 모듈이 필요하다.

### 권한 설정
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

### 로그인, 회원가입, 로그아웃 라우터 구성하기
1. 회원가입 라우터(/join)
    - isNotLoggedIn : 로그인이 안된 사용자만 접근 가능하다.
    - 기존에 같은 이메일로 가입한 사용자가 있는지 조회한 뒤 (User.findOne), 있다면 회원가입 페이지로 되돌려보낸다.
    - 없다면 비밀번호를 bcrypt 모듈로 암호화해서 저장한다.
    - 프로미스를 지원하는 함수이므로 await을 사용했다.
2. 로그인 라우터(/login)
    - isNotLoggedIn : 로그인이 안된 사용자만 접근 가능하다.
    - 로그인 요청이 들어오면 **passport.authenticate('local')** 미들웨어가 로컬 로그인 전략을 수행한다.
    - 위의 passport.authenticate('local') 는 미들웨어 내의 미들웨어 이므로 사용자 정의 기능을 추가하고 싶을 때 내부 미들웨어에 (req, res, next)를 인수로 제공해서 호출하면 된다.
    - 전략이 성공하거나 실패하면 authenticate 메서드의 콜백 함수가 실행된다.
        - 콜백 함수의 첫 번째 매개변수 **authError 값이 있다면 실패**한 것이다.
        - **두 번째 매개변수 값이 있다면 성공**한 것이고 **req.login** 메서드를 호출한다. 
    - Passport 는 req 객체에 login과 logout 메서드를 추가한다. **req.login은 passport.serializeUser 를 호출**한다.
3. 로그아웃 라우터(/logout)
    - req.logout 메서드는 req.user 객체를 제거하고, res.session.destroy는 req.session 객체의 내용을 제거한다.
    - 세션 정보를 지운 후 메인 페이지로 되돌아 간다.<br/><br/>
    ```java
    // routes/auth.js
    const passport = require('passport');
    const bcrypt = require('bcrypt');
    const { isLoggedIn, isNotLoggedIn } = require('./middlewares');

    router.post('/join', isNotLoggedIn, async (req, res, next) => {
    const { email, nick, password } = req.body;
    try {
        const exUser = await User.findOne({ where: { email } });
        if (exUser) {
        return res.redirect('/join?error=exist');
        }
        const hash = await bcrypt.hash(password, 12);
        await User.create({
        email,
        nick,
        password: hash,
        });
        return res.redirect('/');
    } catch (error) {
        console.error(error);
        return next(error);
    }
    });

    router.post('/login', isNotLoggedIn, (req, res, next) => {
    passport.authenticate('local', (authError, user, info) => {
        if (authError) {
        console.error(authError);
        return next(authError);
        }
        if (!user) {
        return res.redirect(`/?loginError=${info.message}`);
        }
        return req.login(user, (loginError) => {
        if (loginError) {
            console.error(loginError);
            return next(loginError);
        }
        return res.redirect('/');
        });
    })(req, res, next); // 미들웨어 내의 미들웨어에는 (req, res, next)를 붙입니다.
    });

    router.get('/logout', isLoggedIn, (req, res) => {
    req.logout();
    req.session.destroy();
    res.redirect('/');
    });

    ```


### 로그인 전략 구현
1. LocalStrategy 생성자의 첫 번째 인수
    - **전략에 관한 설정을 하는 곳**
    - usernameField와 passwordField 에는 일치하는 로그인 라우터의 req.body 속성명을 적으면 된다.
2. LocalStrategy 생성자의 두 번째 인수
    - **실제 전략을 수행**하는 async 함수
    - 첫 번째 인수에서 넣어준 email과 password는 각각 async 함수의 첫 번째와 두 번째 매개변수가 된다. 
    - 세 번째 매개변수인 done 함수는 passport.authenticate의 콜백 함수이다.
- 전략 : 데이터베이스에서 이메일과 일치하는 이메일 있는지 조회 후 있다면 bcrypt의 compare 함수로 비밀번호 비교한다. 비밀번호도 동일하면 done 콜백함수에 조회한 user 정보를 넣어 준다.<br/><br/>
    ```java
    // passport/localStrategy.js
    const passport = require('passport');
    const LocalStrategy = require('passport-local').Strategy;
    const bcrypt = require('bcrypt');

    const User = require('../models/user');

    module.exports = () => {
    passport.use(new LocalStrategy({
        usernameField: 'email',
        passwordField: 'password',
    }, async (email, password, done) => {
        try {
        const exUser = await User.findOne({ where: { email } });
        if (exUser) {
            const result = await bcrypt.compare(password, exUser.password);
            if (result) {
            done(null, exUser);
            } else {
            done(null, false, { message: '비밀번호가 일치하지 않습니다.' });
            }
        } else {
            done(null, false, { message: '가입되지 않은 회원입니다.' });
        }
        } catch (error) {
        console.error(error);
        done(error);
        }
    }));
    };
    ```


## 참고)
- [passport document](http://www.passportjs.org/docs/authenticate/)
- https://www.zerocho.com/category/NodeJS/post/57b7101ecfbef617003bf457
- [Node.js 교과서](http://www.yes24.com/Product/Goods/62597864)