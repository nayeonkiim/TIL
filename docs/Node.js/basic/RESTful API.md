---
id: Node6
title: RESTful API
---
_[Node.js 웹개발로 알아보는 백엔드 자바스크립트의 이해](https://www.inflearn.com/course/node-js-%EC%9B%B9%EA%B0%9C%EB%B0%9C)을 보고 학습한 내용을 정리한 것 입니다._

## RESTful API
- REST한 방식의API라는 건, 아래처럼 잘 설계된 API를 말한다.
    - 웹을 근간으로 하는 HTTP Protocol 기반이다.
    - 리소스는 URI(Uniform Resource Identifiers) 로 표현하며 말 그대로 고유해야 한다.
    - URI는 단순하고 직관적인 구조이어야 한다.
    - 리소스의 상태는 HTTP Methods를 활용해서 구분한다.
    - xml/json을 활용해서 데이터를 전송한다. (주로 json)

- CRUD  
    - 네트워크를 통해 리소스를 다루기 위한 행위들로 POST, GET, PUT, DELETE 가 있다.
        - Create (POST)
        - Retrieve (GET)
        - Update (PUT)
        - Delete (DELETE)

- Example
    - 영화 관련 예제
    - URL은 동일할 수 있고 Methods에 의해 달라진다.<br/><br/>
    - 
    |URL|Methods|설명|
    |------|---|---|
    |/movies|GET|모든 영화리스트 가져오기|
    |/movies|POST|영화 추가|
    |/movies/:title|GET|title 해당 영화 가져오기|
    |/movies/:title|DELETE|title 해당 영화 삭제|
    |/movies/:title|PUT|title 해당 영화 업데이트|
    |/movies?min=9|GET|영화 별점이 9점 이상인 것 가져오기|