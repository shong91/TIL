# CORS 에러는 프론트에서 확인해야 하는 거 아니에요?

![](https://i.imgflip.com/5f7ibr.jpg)

라고만 말하면 클납니다. (..)

솔직히 말하면 내가 그랬다. 여태까지 feature 개발을 주로 하고, 백엔드 프레임워크를 구성하는 일은 항상 시니어 개발자/타 부서에서 담당해 주었기 때문에, CORS 에러라는 걸 고민해본 적이 거의 없었다. 프론트엔드 개발자가 많이 마주하는 에러라고 알고 있었기에 그냥 그런줄만 알고 있었다. 이번에 CORS configure 관련 코드를 세팅해 두었음에도 계속해서 CORS 에러가 발생하는 이슈가 있었고, 내가 CORS 에 대하여 잘 모르고 있다 보니 이 에러가 프론트에서 나는 문제인지, 서버 쪽 문제인지 파악하는데 시간이 걸렸다. 이번 기회에 CORS 에 대해서 공부하게 되었고, 개념과 트러블슈팅 노트를 정리하며 알게 된 내용들을 정리해 보고자 한다.

## CORS 란?

웹 애플리케이션을 개발할 때, 백엔드와 클라이언트 서버를 분리하는 것이 요즘은 일반적이다.
React나 Vue 등을 사용하여 프론트 서버를 개발하게 된다면, 클라이언트 서버는 `localhost:3000`, API 서버는 `localhost:8080` 와 같이 같은 호스트에서 서로 다른 포트를 사용하게 될 것이다.

이 상황에서, 클라이언트 서버가 API 서버를 호출한다면 아래와 같은 CORS 에러가 발생한다.

**_왜 그럴까?_**

\<img>, \<video>, \<script> 등의 태그는 기본적으로 Cross-Origin 정책을 지원한다. **하지만 XMLHttpRequest, Fetch API 스크립트는 기본적으로 Same-Origin 정책을 따른다**.

즉, <u>자바스크립트에서 기본적으로 보안 상의 이유로 서로 다른 도메인에 대한 요청을 제한하기 때문에</u>, ajax, axios 요청으로 다른 도메인의 API 를 호출할 시 교차 출처 리소스 공유정책(Cross-Origin-Resource-Sharing; CORS) 을 위반하였다는 에러를 발생 시키는 것이다.

```
has been blocked by CORS policy: No ‘Access-Control-Allow-Origin’ header is present on the requested resource. If an opaque response serves your needs, set the request’s mode to ‘no-cors’ to fetch the resource with CORS disabled.
```

**_왜 기본적으로 Same-Origin policy을 따를까_?**

교차 출처에 대한 제약이 없다면, 해커가 CSRF(Cross-Site Request Forgery)나 XSS(Cross-Site Scripting) 등의 방법을 이용해서 악의적인 접근을 할 가능성이 있다. (ex 사용자가 모르게 악성 사이트에 접속하도록 하여 개인정보 등을 빼낸다거나..)

때문에 다른 출처의 스크립트가 실행되지 않도록, Same-Origin policy 를 적용하여 브라우저에서 사전에 방지하는 것이다.

## 동일 출처와 교차 출처는 어떻게 구분할까?

출처의 다름 유무는 **Protocol, Host, Port** 3가지로 판단한다. 이 세 가지가 동일하다면 동일 출처로 판단한다.

| URL                              | 동일 출처 여부 | 비고                      |
| -------------------------------- | -------------- | ------------------------- |
| https://www.sample.com:3000      | O              | Protocol, Host, Port 동일 |
| https://www.sample.com:3000/user | O              | Protocol, Host, Port 동일 |
| http://www.sample.com:3000       | X              | Protocol 다름             |
| https://www.example.com:3000     | X              | Host 다름                 |
| https://www.example.com:4000     | X              | Port 다름                 |

## CORS 에러, 어디까지가 클라이언트의 몫일까?

기본적으로 자바스크립트는 Cross-Origin 을 제한한다고 하였다. **출처를 비교하고 Cross-Origin 을 차단하는 일은 브라우저가 담당한다.**

교차 출처 리소스 공유정책(Cross-Origin-Resource-Sharing; CORS) 은 단어 그대로 다른 출처의 리소스 공유에 대한 허용/비허용 정책이다. 즉 클라이언트 서버에서는 API 서버에 대한 Cross-Origin 을 허용해주고, API 서버는 이 해당 응답이 신뢰할 수 있는 응답이라는 것을 알려주어야 한다.

**_브라우저의 CORS 기본 동작_**

1. 클라이언트에서 HTTP 요청의 헤더에 `Origin` 을 담아 전달
2. 서버는 응답 헤더에 `Access-Control-Allow-Origin` 을 담아 전달
3. 클라이언트의 `Origin` 과 서버의 `Access-Control-Allow-Origin` 을 비교
4. 유효하지 않다면 CORS 에러 발생

자...... 그래서 Origin 에 헤더를 담아 요청을 보냈는데, 여전히 CORS 에러가 발생한다. 이제부터는 CORS 에러 트러블슈팅 노트.

### 1. Preflight Request (예비 요청)

브라우저는 요청을 보낼 때 한 번에 바로 보내지 않고, 예비 요청(Preflight) 를 먼저 보내 서버와 통신이 되는지 확인한 뒤 본 요청을 보낸다.

즉, 예비 요청의 역할은 본 요청을 보내기 전에 브라우저 스스로 안전한 요청인지 미리 확인하는 것이다. 이 예비 요청은 `HTTPMethod.OPTIONS` 으로 보내진다.

**_Preflight 기본 동작_**

1. 브라우저는 서버로 예비 요청을 먼저 보낸다
2. 서버는 허용/비허용에 대한 헤더 정보를 응답한다
   - Access-Control-Allow-Origin, Access-Control-Allow-Method, Access-Control-Allow-Header, Access-Control-Max-Age
3. 브라우저는 요청과 응답의 정책을 비교하여, 해당 요청이 안전한지 확인한 뒤 본 요청을 보낸다
4. 서버는 본 요청에 대하여 응답한다

때문에, 백엔드 서버에서는 Preflight 요청이 들어올 수 있도록 `HTTPMethod.OPTIONS` 에 대한 Access-Control-Allow-Method 을 추가로 설정해 주어야 한다.

### 2. more-private address

```
has been blocked by CORS policy: the request client is not a secure context and the resource is in more-private address space `local`
```

develop 클라이언트 서버 - 로컬 API 서버 간 통신으로 테스트 하였을 때 위와 같은 에러가 발생하였다. origin 보다 더 낮은 수준(more-private)의 네크워크로 요청을 보내는 경우, 위와같이 에러를 발생한다.

- Local Address: 127.0.0.1, localhost 등 로컬 IP 주소
- Private Address: 사설망 주소. 공유기에서 각 기기마다 부여되는 IP 주소
- Public Address: 어느 인터넷 환경에서도 접속이 가능한 공인 IP 주소

위의 경우는 Public Address 에서 Local Address 로 (더 낮은 수준으로) 요청을 보냈기에 발생한 오류로, 클라이언트와 서버 모두 develop 서버로 띄웠을 때 해당 오류는 해결할 수 있었다.

### 3. credentials

```
has been blocked by CORS policy: The value of the 'Access-Control-Allow-Credentials' header in the response is '' which must be 'true' when the request's credentials mode is 'include'.
```

로그인이 성공한 유저는 쿠키나 Authorization 헤더에 토큰 정보 등을 싣어서 API 요청을 보낸다.

이때 요청에 인증과 관련된 정보를 담을 수 있도록 `credentials` 옵션을 추가해주어야 한다.

```
axios.post('API server url', { data },
    {
    withCredentials: true // 클라이언트와 서버가 통신할때 쿠키와 같은 인증 정보 값을 공유하겠다는 설정
})
```

## 그러면 백엔드는 뭘 해야하는데?

클라이언트 설정을 마쳤다면, 백엔드 서버에서 `Access-Control-Allow-Origin` 를 설정하여야 한다.
설정하지 않는다면 아래와 같은 CORS 에러를 뱉어낸다.

```
has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

### 1. CORS configuration in Web MVC

Spring MVC 기준으로 아래와 같이 CORS configuration 을 전역 설정한다.

```
// 전역으로 CORS 설정
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
        	.allowedOrigins("http://localhost:3000") // 허용할 출처
            .allowedMethods("*") // 허용할 HTTP method (OPTIONS 도 허용)
            .allowCredentials(true) // 쿠키 인증 요청 허용
            .maxAge(3000) // 원하는 시간만큼 pre-flight 리퀘스트를 캐싱
            ;
    }
}
```

### 2. CORS configuration in Spring Security

스프링 시큐리티를 적용하였다면, 전역으로 설정한 CORS configuration 보다 Spring security configuration 에서 CORS 설정을 추가할 수 있다.

```
@EnableWebSecurity
@Configuration
public class SecurityConfiguration {

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.cors();
    return http.build();
  }

  @Bean
  public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOrigins(Arrays.asList("http://localhost:3000"));
    configuration.setAllowCredentials(true);
    configuration.setAllowedHeaders(Arrays.asList("*"));
    configuration.setAllowedMethods(Arrays.asList("*"));
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", configuration);
    return source;
  }
}

```

## References.

- https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-CORS-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-%F0%9F%91%8F
