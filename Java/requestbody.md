## @RequestBody 의 동작방식

@RequestBody는 클라이언트가 전송하는 Json(application/json) 형태의 HTTP Body 내용을 Java Object로 변환시켜주는 역할을 한다.

그렇기 때문에 Body가 존재하지 않는 HTTP Get 메소드에 @RequestBody를 활용하려고 한다면 에러가 발생하게 된다.

다음은 RequestBody 어노테이션에 대한 설명이다.

```
Annotation indicating a method parameter should be bound to the body of the web request.
The body of the request is passed through an {@link HttpMessageConverter} to resolve the
method argument depending on the content type of the request. Optionally, automatic
validation can be applied by annotating the argument with {@code @Valid}.
```

Body 는 HttpMessageConverter 를 통해, request 의 Content-type 에 따라 method arguments 를 지정하고, 지정된 타입에 맞추어 객체를 변환한다.

`Content-type: application/json` 으로 받는 HTTP Body는, Spring에서 관리하는 MessageConverter들 중 하나인 MappingJackson2HttpMessageConverter를 통해 Java 객체로 변환되는 것이다.

값을 주입하지 않고 변환을 시키므로(엄밀히는 Reflection을 사용하여 할당), 변수들의 생성자나 Setter함수가 없어도 정상적으로 값이 할당됨

## References.

https://mangkyu.tistory.com/72
https://kim-jong-hyun.tistory.com/60
