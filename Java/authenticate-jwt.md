# Spring Security + JWT 를 이용한 인증/인가

Spring Security + JWT 를 이용한 인증/인가 기능을 구현하였던 내용을 정리하고자 한다.
MSA 서비스를 염두하여 무상태성을 유지하는 어플리케이션 개발이 필요하였고, 자연스레 토큰 기반의 인증 방식을 채택하게 되었다.

1. Security filter 와 authority/role 을 사용하기 위해 Spring Security 를,
2. 무상태성을 유지하는 어플리케이션 개발을 위해 JWT 를,
3. 데이터에 빠르게 접근 가능하여 병목 현상을 방지하고, 데이터의 유효기간을 지정할 수 있는 in-memory 인 redis 를 사용하였다.

## JWT

![](/z.images/jwt-01.png)

JWT는 Json Web Token 의 약자로, Header, Payload, Signature 의 세 파트로 구분되며, 각각 JSON 형태의 데이터를 base 64 인코딩 후 `.` 을 이용해 합쳐 하나의 문자열로 생성한다.

1. `Header`에는 JWT 웹 토큰의 헤더 정보를,
2. `Payload`에는 실제 토큰으로 사용하려는 데이터를,
3. `Signature` 에는 Header와 Payload의 데이터 무결성과 변조 방지를 위한 서명을 입력한다.

payload 에 담기는 각 데이터를 claim 이라고 부르며, Header + Payload 를 합친 후, Signature에 입력된 Secret 키와 함께 Header의 해싱 알고리즘으로 인코딩하여 토큰을 생성한다.

## 인증/인가 프로세스

![](/z.images/authenticate.png)

1. 사용자 로그인 요청이 들어오면, 서버는 사용자의 인증 정보를 검증한다.
2. 유효한 인증 정보인 경우

- 2-1. 서버는 JWT Access Token과 Refresh Token을 생성하고
- 2-2. 생성된 Refresh Token을 redis 저장소에 저장한다.

3. 생성된 Access Token, Refresh Token을 사용자에게 전달하고, 클라이언트는 이를 메모리 등에 저장한다.
4. 클라이언트가 보유한 Access Token이 만료된 경우, 클라이언트는 서버에 Access Token, Refresh Token을 사용하여 새로운 Access Token을 요청한다.

- 4-1. 서버는 클라이언트로부터 받은 Access Token 의 조작 여부를 다시 확인한 뒤
- 4-2. Refresh Token을 저장소에 저장된 값과 비교하여 일치하는지 확인한다
  - 4-2-1. 유효한 경우에만 새로운 Access Token을 발급한다.
  - 4-2-2. Refresh Token이 유효하지 않다면(만료) 클라이언트는 사용자를 로그아웃 처리한다.

5. 클라이언트는 발급받은 새로운 Access Token을 사용하여 API 를 요청한다.
6. 사용자가 로그아웃하거나 Refresh Token이 만료된 경우, 저장소에서 해당 Refresh Token을 삭제하여 더 이상 사용되지 않도록 처리한다.

## 구현하기

[Spring Security + JWT 를 이용한 인증/인가 (2)](/Java/authenticate-jwt-2.md) 에서 계속...
