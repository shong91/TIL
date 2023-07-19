# Shorten URL (단축 URL) 생성하기

회원 서비스를 개발하면서, 회원을 초대하여 가입시키는 프로세스를 진행하였다.

초대 메일 발송 시에 회원 가입 링크를 보내, 유저가 링크를 클릭하여 나머지 프로세스를 진행할 수 있도록 하는 것이었다.

모든 프로세스가 완료 되어야만 회원 테이블에 저장되도록 구조를 잡았는데, 그렇기 때문에 2가지 문제가 발생했다.

1. 초대할 회원 정보를 모두 메일 링크에 포함하게 되면, 링크가 길어질 뿐 아니라 중요 정보들이 모두 노출되게 된다.
2. 메일 발송 내용을 DB 에 저장하고 저장한 시퀀스를 메일 링크에 포함하게 되면, 시퀀스를 통해 메일 발송 순서를 추정할 수 있게 된다.

위 두 가지 문제점을 해결하기 위해, 메일 발송 시퀀스를 암호화 한 shorten URL 을 생성하여 메일 링크가 길어지는 문제점과, 시퀀스 번호가 그대로 노출되는 문제점을 해결하고자 하였다.

대표적인 단축URL 생성 서비스로는 https://bitly.com/ 가 있고, youTube 등 많은 서비스에서도 공유 링크를 단축URL 형태로 제공하고 있다.

## 구현하기

### 1. 원본 URL을 저장할 테이블 생성

기본적으로 id(auto increment) 와 origin_url 컬럼이 필요하며, 이외에 필요한 컬럼을 정의하여 테이블을 생성한다.
나는 메일 발송 이력을 관리하는 테이블을 생성하여, 관련 정보들을 함께 저장하였다.

| Table      |
| ---------- |
| id (PK)    |
| origin_url |
| ...        |

### 2. Base62 모듈 생성

Base64는 많이 들어보았는데, 이번에 Base62는 처음 들어보았다.

![](https://miro.medium.com/v2/resize:fit:828/format:webp/1*KFxmnR7TtMtI-LSc2mdv_Q.png)

위의 표가 Base64 인코딩 표인데, 62번이 `+`, 63번이 `/` 기호를 사용하고 있다.
URL에서는 / 기호가 구분자로 사용되기 때문에, 인코딩을 했을 때에 / 가 포함된다면 기대값과 다르게 동작할 수 있다.

또한, + 기호는 공백문자를 표현할 때에 사용되므로 (ex) https://www.google.com/search?q=hello+world) 역시 인코딩 기호에서 제외되어야 한다.
Base62는 위 2가지 케이스를 제외하고 URL-safe 한 문자들만 모든 방식이라고 할 수 있겠다.

Base62 모듈은 오픈소스 라이브러리를 찾아보았는데 마땅한 걸 찾지 못하여, 구글링한 내용을 참고하여 직접 구현하였다.

```
@Slf4j
@Component
public class Base62 {
    private final int BASE62 = 62;
    private final String BASE62_CHAR = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

    public static String encode(long param) {
        StringBuffer sb = new StringBuffer();
        while(param > 0) {
            sb.append(BASE62_CHAR.charAt((int) (param % BASE62)));
            param /= BASE62;
        }
        return sb.toString();
    }

    public static long decode(String param) {
        long sum = 0;
        long power = 1;
        for (int i = 0; i < param.length(); i++) {
            sum += BASE62_CHAR.indexOf(param.charAt(i)) * power;
            power *= BASE62;
        }
        return sum;
    }
}
```

### 3. Base62 인코딩 방식으로 Shorten URL 생성

프로세스를 조금 더 자세히 적자면 아래와 같다.

1. 시퀀스와 원본 URL 저장

- 이 때, auto increment 되는 시퀀스를 1 부터 설정한다면 역시 시퀀스 순서를 유추할 수 있게 되기 때문에, 초기값을 10000 과 같이 큰 숫자부터 시작하게 하는 것이 좋다.

2. 시퀀스를 Base62 encode
3. shorten url 리턴

### 4. Shorten URL 클릭 시 Origin URL 으로 리다이렉트

프로세스를 조금 더 자세히 적자면 아래와 같다.

![](/z.images/shorten-url.png)

1. Shorten URL 클릭
2. Base62 decode -> 시퀀스
3. DB 에서 시퀀스로 원본 url 조회
4. 원본 url 리다이렉트

### 5. 예외 처리하기

DB 에서 시퀀스로 원본 url 조회 시, 아래의 경우에 대하여 예외를 던지도록 하였다.

1. Shorten URL 이 잘못되었을 경우

- DB 에 해당하는 정보가 없음

2. URL 유효기간이 만료되었을 경우

- 생성된 링크의 유효기간은 1일으로 설정하여, 유효기간이 만료된 링크는 더이상 사용할 수 없음

3. 이미 사용된 URL 일 경우

- URL 사용여부를 DB 에 함께 저장하여, 이미 사용된 URL은 더이상 사용할 수 없음

```
{
  "code": "-99",
  "desc": "조회된 결과가 없습니다. ",
}
```

세 가지 에러 처리를 모두 같은 에러코드를 리턴할지, 세분화 할 지 고민이었는데,

너무 세분화하여 에러코드를 정의하면 외려 에러 관리 정책 정보를 노출시키게 되는 위험이 있어 일부러 하나의 에러코드로 관리하도록 하였다.

하지만 여전히 고민스러운 부분이기는 하다. 이렇게 되면 URL이 만료되었는지, 이미 사용되었는지, 아니면 아예 부적절한 링크인지 유저 입장에서는 파악할 방법이 없으니..
이와 관련하여 좋은 글이 있어 함께 링크를 남겨본다. ([좋은 에러 메세지를 만드는 6가지 원칙](https://toss.tech/article/how-to-write-error-message))

## References.

- https://42place.innovationacademy.kr/archives/9063
- https://medium.com/monday-9-pm/%EC%B4%88%EB%B3%B4-%EA%B0%9C%EB%B0%9C%EC%9E%90-url-shortener-%EC%84%9C%EB%B2%84-%EB%A7%8C%EB%93%A4%EA%B8%B0-1%ED%8E%B8-base62%EC%99%80-%EC%B6%A4%EC%9D%84-9acc226fb7eb
- https://github.com/seruco/base62
- https://antkdi.github.io/posts/post_url-shortener_2/
