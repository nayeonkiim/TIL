---
id: Others2
title: ResponseEntity,HttpEntity란?
---

<p>
Spring Security 를 공부하던 중, test 코드 작성을 중, 헤더에 token 값을 담아주기 위해 ResponseEntity와 HttpEntity를 알게 되었다.
</p>

## HttpEntity 란?
- Represents an HTTP request or response entity, consisting of headers and body 라고 설명이 되어 있다.
- 즉, HTTP 요청 또는 응답에 해당하는 **HttpHeader**와 **HttpBody**를 포함하는 클래스이다.
- 아래의 코드처럼 body와 header 값을 사용할 수 있다.
    ```java
    HttpEntity<String> entity = template.getForEntity("https://example.com", String.class);
    String body = entity.getBody();
    MediaType contentType = entity.getHeaders().getContentType();
    ```

## ResponseEntity 란?
- HttpEntity 클래스를 상속받아 구현한 클래스이다.
    `public class RequestEntity<T> extends HttpEntity<T> {...}`
- HttpStatus, HttpHeaders, HttpBody를 포함한다.

```java
ResponseEntity<String> entity = template.getForEntity("https://example.com", String.class);
String body = entity.getBody();
MediaType contentType = entity.getHeaders().getContentType();
HttpStatus statusCode = entity.getStatusCode();
```

## 사용한 test 코드
```java
@DisplayName("2. 인증 성공")
@Test
void test_2() {
    HttpHeaders headers = new HttpHeaders();
    headers.add(HttpHeaders.AUTHORIZATION, "Basic + Base64.getEncoder().encodeToString(
        "user1:1111", getBytes()
    ));
    HttpEntity entity = new HttpEntity(null, headers);
    ResponseEntity<String> resp = client.exchange(greetingUrl(), HttpMethod.GET, entity, String.class);

    System.out.println(resp.getBody());
}
``` 

## [참고]
- https://devlog-wjdrbs96.tistory.com/182
- fast campus java/spring 웹 개발 마스터