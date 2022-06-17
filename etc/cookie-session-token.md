# Cookie vs. Session vs. Token 개념 정리

### http 프로토콜의 비연결성(Connectionless) 과 무상태(Stateless):

- 서버로 가는 모든 요청은 이전 리퀘스트와 독립적으로 진행됨.
- 클라이언트 요청 - 서버 응답이 완료되면 바로 연결은 끊어지며, 연결이 해제됨과 동시에 서버와 클라이언트는 이전 요청/결과에 대해 잊어버린다.
- 요청 상태를 잊어버리게 된다면, '인증'은 어떻게 구현해야하나? 매번 클라이언트는 데이터베이스를 왕복하며 인증을 해야 하는 걸까?
- _cookie, session, token 의 차이점을 알고 인증 구현에 사용하자!_

# cookie

- 브라우저에서 요청 -> 서버에서 응답 시, 브라우저에 저장하고자 하는 내용을 쿠키에 [key:value] 형태로 담아 보냄.
- 쿠키에는 인증 외에도 여러가지 정보를 담아 보낼 수 있으며, 브라우저에서는 이 내용을 저장해두고 사용
- 유저 정보는 Client side에 저장됨
- 유효기간 있음
- _[단점]_ 쿠키는 네트워크를 통해 전달되기 때문에, 중간에 탈취할 수 있음 => 보안 취약점
- **이를 보완하기 위해 session 을 사용**

# session

- 웹 브라우저는 각각 별도의 세션(고유한 sessionID) 를 가진다.
- 서버와 클라이언트는 cookie를 통해 sessionID 만을 주고 받는다.
- 클라이언트에서 보낸 sessionID 를 통해 서버는 session DB 에 접근, 요청한 유저 정보를 확인한다. 유저 정보가 확인되면 서버는 브라우저에 sessionID를 전송한다.
- 유저 정보는 Server side에 저장됨
- 확인한 유저 정보로 로그인 인증 외의 추가 기능을 구현할 수 있음 (강제 로그아웃 처리 등)
- _[단점 1]_ 쿠키는 브라우저에만 존재하며, iOS, android 등 Native app 에는 존재하지 않음
- _[단점 2]_ 요청이 들어올 때 마다 서버는 sessionDB 를 조회하기 때문에, 서비스가 커지고 유저가 늘어남에 따라 DB 부하 발생 (때문에, 세션을 사용 시에는 이에 최적화된 redisDB 를 주로 사용)
- **이를 보완하기 위해 token 사용**

# token

- 대표적으로 JWT (JSON Web Token)
- token is a string: space 제약이 없어 매우 긴 string 이 가능
- 토큰 자체로 정보를 가지고 있어, 별도의 인증 서버가 필요 X (sessionDB 에 접근하지 않고, 유저 인증을 할 수 있음)
- jwt는 서버가 이중 삼중 여러대 일 때 강점을 가짐
  - 세션의 경우 로드밸런싱으로 유저가 처음 방문했을때는 1번 서버로 방문하여 세션이 남아 있지만, 두번째 방문했을때 1번이 아닌 2번으로 붙게되면 세션정보가 없어 인증 실패하게 된다.
  - 이를 방지하기 위해 redis 세션통합으로 개발하거나, 서버에 상관없는 jwt로 인증을 구현함.

## JWT 구조

- Header, Payload, Signature 각각 JSON 형태의 데이터를 base 64 인코딩 후 `.` 을 이용해 합친다. (`Header.Payload.Signature`)
- 최종적으로 만들어진 토큰은 HTTP 통신 간 이용되며, Authorization 이라는 key의 value로서 사용된다.

![img_01](/z.images/jwt-01.png)

### Header

JWT 웹 토큰의 헤더 정보.

```
{
  "alg" : "HS256", // 해싱 알고리즘. (HMAC SHA256 or RSA)
  "typ" : "JWT" // 토큰의 타입
}
```

### Payload

실제 토큰으로 사용하려는 데이터가 담기는 부분. 각 데이터를 Claim 이라 한다.

- Reserved claims : 이미 예약된 Claim. 필수는 아니지만 사용하길 권장. key 는 모두 3자리 String이다.
  - iss (String) : issuer, 토큰 발행자 정보
  - exp (Number) : expiration time, 만료일
  - sub (String) : subject, 제목
  - aud (String) : audience
- Public claims : 사용자 정의 Claim.
  - 공개용 정보
  - 충돌 방지를 위해 URI 포맷을 이용해 저장한다.
- Private claims : 사용자 정의 Claim
  - 사용자가 임의로 정한 정보

### Signature

- Header와 Payload의 데이터 무결성과 변조 방지를 위한 서명
- Header + Payload 를 합친 후, Secret 키와 함께 Header의 해싱 알고리즘으로 인코딩

## 인증 과정

1. 클라이언트에서 사용자 ID/PW 를 통해 인증 요청을 보냄
2. 서버에서 signed JWT 를 생성하여 해당 토큰을 클라이언트로 응답
3. 클라이언트는 응답받은 토큰을 Http Header에 넣어 (Bearer Token) 요청
4. 서버는 토큰 signed info의 유효성 여부 확인하여 유효할 시 응답

## References.

1. https://pronist.dev/143
2. https://sanghaklee.tistory.com/47
