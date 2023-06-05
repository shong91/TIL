# 헤더에는 왜 언더스코어를 쓰면 안되나요? (feat. 제 로컬에서는 되는데요)

프로젝트에서 커스텀 헤더를 사용하여야 하는 건이 있어, API 호출 시 커스텀 헤더 값을 넣어 요청을 보내는 기능을 개발하였다.

커스텀 등록 헤더에 'X-' 를 prefix 로 붙이는 관례가 폐기된 지 오래라 하여, 'X-' 를 붙이지 않고 커스텀 헤더를 만들었다.

_커스텀 등록 헤더는 'X-'를 앞에 붙여 추가될 수 있지만, 이 관례는 RFC 6648에서 비표준 필드가 표준이 되었을때 불편함을 유발하는 이유로 2012년 6월에 폐기되었습니다. 다른것들은 IANA 레지스트리에 나열되어 있으며, 원본 컨텐츠는 RFC 4229에서 정의되었습니다. IANA는 또한 제안된 새로운 메시지 헤더의 레지스트리도 관리합니다._

_Custom proprietary headers have historically been used with an X- prefix, but this convention was deprecated in June 2012 because of the inconveniences it caused when nonstandard fields became standard in RFC 6648; others are listed in an IANA registry, whose original content was defined in RFC 4229. IANA also maintains a registry of proposed new HTTP headers._

관성적으로 언더스코어(\_) 를 사용하였는데, 로컬에서는 잘 동작하는 것이 개발 서버에 올리니 커스텀 헤더를 읽지 못하는 이슈가 있었다. 제 로컬에서는 동작하는데요?!

### 원인

개발 서버 로그를 까 보니 진짜 언더스코어 헤더만 쏙 빼고 요청을 받아오고 있었다. 원인은 Nginx의 기본 설정에 있었다.

개발 서버를 Nginx 를 이용해 띄우고 있었는데, Nginx 는 기본값으로 `underscores_in_headers=off`로 설정되어 있기 때문이다. 명시적으로 underscores_in_headers를 설정하지 않으면, nginx는 밑줄이 있는 헤더를 자동으로 삭제한다.

_If you do not explicitly set underscores_in_headers on;, nginx will silently drop HTTP headers with underscores (which are perfectly valid according to the HTTP standard)._

```
CUSTOM_HEADER_KEY: 1234 // 읽어오지 못함
CUSTOM-HEADER-KEY: 1234 // 정상 동작
```

### 해결

이를 해결하기 위해서는 nginx 설정에서 옵션을 켜거나, 헤더의 언더스코어를 대시(-) 로 수정하는 두 가지 방법이 있다.

**_1. `.ebextensions > nginx > conf.` 의 nginx 설정 파일 변경_**

```
client_max_body_size 20M;
underscores_in_headers on;
```

nginx 에 따르면, 밑줄을 사용하는 것은 HTTP 표준에 따라 유효하지만, 헤더를 CGI 변수에 매핑하는 과정에서 대시와 밑줄이 모두 밑줄에 매핑되기 때문에, 모호함을 방지하기 위해 `underscores_in_headers=off` 를 디폴트 설정으로 하였다고 한다.

_This is done in order to prevent ambiguities when mapping headers to CGI variables as both dashes and underscores are mapped to underscores during that process._

하여, 기본값을 바꾸는 것 보다는 언더스코어를 대시로 수정하는 방법을 택하였다.

**_2. 헤더의 언더스코어를 대시(-) 로 수정_**

앞서 두 가지 버전을 테스트하여, 대시(-) 가 들어간 버전은 정상적으로 요청 헤더 값을 받아오는 것을 확인하였다.

```
CUSTOM_HEADER_KEY: 1234 // 읽어오지 못함
CUSTOM-HEADER-KEY: 1234 // 정상 동작
```

이와 같이 헤더를 변경하여 문제를 해결하였다.

## References.

- https://developer.mozilla.org/ko/docs/Web/HTTP/Headers
- https://dev.to/thesameeric/dont-use-underscores-in-your-http-headers-gfp
