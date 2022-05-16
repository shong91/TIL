# REST (Representational State Transfer)

## What is REST ?

- "웹의 장점을 최대한 활용할 수 있는 아키텍처"
- 자원을 이름(자원의 표현)으로 구분하여 해당 자원의 상태(정보)를 주고 받는 모든 것.

### 구성

- **Resource 자원**: HTTP URI를 통해 자원을 명시하고,

- **Verb 행위**: HTTP Method (GET/POST/PUT/DELETE) 를 통해

- **Representations 표현**: 해당 자원(URI)에 대한 CRUD Operation을 적용

"URI는 자원을 표현하는 데에 집중하고, 행위에 대한 정의는 HTTP METHOD를 통해 하는 것이 REST한 API를 설계하는 중심 규칙"

```
GET /members/delete/1  (X)
DELETE /members/1      (O)
```

### 장단점

- 장점

  - HTTP 표준 프로토콜 활용 => 웹의 장점을 최대한 활용할 수 있다
    - HTTP 프로토콜의 인프라를 그대로 사용: REST API 사용을 위한 별도 인프라 구축 필요 없음
    - 범용성 보장: HTTP 표준 프로토콜을 따르는 모든 플랫폼에서 사용 가능
  - REST API 메시지가 의도하는 바를 명확하게 나타내므로 의도하는 바를 쉽게 파악
  - 서버 - 클라이언트의 역할을 명확히 분리

- 단점

  - 표준 존재 X
  - HTTP Method 형태가 제한적
  - 구형 브라우저 사용 시 미지원되는 부분 존재 (PUT, DELETE 사용 불가)

### URI 설계 시 주의점

- 슬래시 구분자(/)는 계층 관계를 나타내는 데 사용

  - URI에 포함되는 모든 글자는 리소스의 유일한 식별자로 사용되어야 한다.
  - URI가 다르다는 것은 리소스가 다르다는 것이고, 역으로 리소스가 다르면 URI도 달라져야 함

- URI 마지막 문자로 슬래시(/)를 포함하지 않음

  - RESTful API 서버 인증 우회 공격 대처를 위함 (Spring Security Path Matching Inconsistency 취약점)
  - [RESTful API 서버를 위협하는 한 글자, 슬래시](https://netmarble.engineering/spring-security-path-matching-inconsistency-cve-2016-5007/)

- URI 가독성을 높이기 위해 하이픈(-)을 사용

  - 언더바(\_)는 사용하지 않음

- URI 경로에는 소문자 사용

  - 대소문자에 따라 다른 리소스로 인식되기 때문

- 파일 확장자는 URI에 포함시키지 않음

  - 파일 확장자는 Accept header를 사용

## HTTP 응답 상태 코드

- 2XX: 정상
- 4XX: 클라이언트 단의 에러
- 5XX: 서버 단의 에러

| 상태코드 | 설명                                                                                                                           |
| -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| 200      | 클라이언트의 요청을 정상적으로 수행함                                                                                          |
| 201      | 클라이언트의 요청을 정상적으로 수행함(POST Method - 리소스 생성)                                                               |
| 301      | 클라이언트가 요청한 리소스에 대한 URI가 변경 되었을 때 사용하는 응답 코드 (응답 시 Location header에 변경된 URI를 적어 줘야함) |
| 400      | 클라이언트의 요청이 부적절 할 경우 사용하는 응답 코드                                                                          |
| 401      | 클라이언트가 인증되지 않았거나, 유효한 인증 정보가 부족하여 요청이 거부되었음을 의미하는 응답 코드 (Unauthorized)              |
| 403      | 클라이언트가 인증되었고 서버가 해당 요청을 이해했지만, 권한이 없어 요청이 거부되었음을 의미하는 응답 코드 (Forbidden)          |
|          | (403 자체가 리소스가 존재한다는 뜻이기 때문에, 403 보다는 400이나 404를 사용할 것을 권고. )                                    |
| 405      | 클라이언트가 요청한 리소스에서는 사용 불가능한 Method를 이용했을 경우 사용하는 응답 코드                                       |
| 500      | 서버에 문제가 있을 경우 사용하는 응답 코드                                                                                     |

## References.

- https://meetup.toast.com/posts/92
- https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html
- https://netmarble.engineering/spring-security-path-matching-inconsistency-cve-2016-5007/
