# AOP와 @Transactional의 동작 원리

## what is AOP ?

> "관점 지향 프로그래밍" - Aspect(관점)이란 흩어진 관심사들을 하나로 모듈화 한 것.

객체 지항 프로그래밍(OOP)에서는 주요 관심사에 따라 클래스를 분할한다. 이 클래스들은 보통 SRP(Single Responsibility Principle)에 따라 하나의 책임만을 갖게 설계된다.

하지만 클래스를 설계하다보면 로깅, 보안, 트랜잭션 등 여러 클래스에서 공통적으로 사용하는 부가 기능들이 생긴다. 이들은 주요 비즈니스 로직은 아니지만, 반복적으로 여러 곳에서 쓰이는 데 이를 흩어진 관심사(Cross Cutting Concerns)라고 한다.

AOP 없이 흩어진 관심사를 처리하면 다음과 같은 문제가 발생한다.

- 여러 곳에서 반복적인 코드를 작성해야 한다.
- 코드가 변경될 경우 여러 곳에 가서 수정이 필요하다.
- 주요 비즈니스 로직과 부가 기능이 한 곳에 섞여 가독성이 떨어진다.

흩어진 관심사를 별도의 클래스로 모듈화하여, OOP 를 더 잘 지킬 수 있도록 하는 것이 AOP 의 핵심이다.

## AOP의 주요 개념

- Aspect: Advice + PointCut로 AOP의 기본 모듈
- Advice: Target에 제공할 부가 기능을 담고 있는 모듈
- Target: Advice이 부가 기능을 제공할 대상 (Advice가 적용될 비즈니스 로직)
- JointPoint: Advice가 적용될 위치
  - 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용 가능
- PointCut: Target을 지정하는 정규 표현식

## Spring AOP

- 기본적으로 프록시 방식으로 동작한다.
- Spring AOP는 프록시 객체를 자동으로 생성해주어, Aspect/Advice에 직접적으로 의존하지 않게 해준다.

> 프록시 패턴이란?
> 어떤 객체를 사용하고자 할 때, 직접적으로 참조하는 것이 아니라, 해당 객체를 대리(proxy) 하는 객체를 통해 대상 객체에 접근하는 방식이다.
> and more... [https://coding-factory.tistory.com/711]

Spring은 왜 Target 객체를 직접 참조하지 않고 프록시 객체를 사용할까?

프록시 객체 없이 Target 객체를 사용하고 있다고 생각해보자. Aspect 클래스에 정의된 부가 기능을 사용하기 위해서, 우리는 원하는 위치에서 직접 Aspect 클래스를 호출해야 한다.

이 경우 Target 클래스 안에 부가 기능을 호출하는 로직이 포함되기 때문에, AOP를 적용하지 않았을 때와 동일한 문제가 발생한다. 여러 곳에서 반복적으로 Aspect를 호출해야 하고, 그로 인해 유지보수성이 크게 떨어진다.

그래서 Spring에서는 Target 클래스 혹은 그의 상위 인터페이스를 상속하는 프록시 클래스를 생성하고, 프록시 클래스에서 부가 기능에 관련된 처리를 한다. 이렇게 하면 Target에서 Aspect을 알 필요 없이 순수한 비즈니스 로직에 집중할 수 있다.

> JDK Proxy와 CGLib Proxy

- JDK Proxy

  - Target의 상위 인터페이스를 상속 받아 프록시를 만든다.
  - 따라서 인터페이스를 구현한 클래스가 아니면 의존할 수 없다. Target에서 다른 구체 클래스에 의존하고 있다면, JDK 방식에서는 그 클래스(빈)를 찾을 수 없어 런타임 에러가 발생한다.

- CGLib Proxy
  - Target 클래스를 상속 받아 프록시를 만든다.
  - JDK 방식과는 달리 인터페이스를 구현하지 않아도 되고, 구체 클래스에 의존하기 때문에 런타임 에러가 발생할 확률도 상대적으로 적다.
  - JDK Proxy는 내부적으로 Reflection을 사용해서 추가적인 비용이 들지만, CGLib는 그렇지 않다.
  - Spring Boot에서는 CGLib를 사용한 방식을 기본으로 채택하고 있다.

## @Transactional

![](https://velog.velcdn.com/images/ann0905/post/56a48b12-b2d0-4071-b09e-959e585551bb/image.png)

동작 순서

1. target에 대한 호출이 들어오면 AOP proxy가 이를 가로채서(intercept) 가져온다.
2. AOP proxy에서 Transaction Advisor가 commit 또는 rollback 등의 트랜잭션 처리를 한다.
3. 트랜잭션 처리 외에 다른 부가 기능이 있을 경우 해당 Custom Advisor에서 그 처리를 한다.
4. 각 Advisor에서 부가 기능 처리를 마치면 Target Method를 수행한다.
5. interceptor chain을 따라 caller에게 결과를 다시 전달한다.

유의사항

1. 원본 클래스/인터페이스를 상속 받아 프록시를 생성하기 때문에 접근 제어가 private 인 경우 트랜잭션 처리를 할 수 없다.
2. 트랜잭션은 객체 외부에서 처음 진입하는 메서드를 기준으로 동작한다. 객체 외부에서 처음으로 진입하는 메서드에 트랜잭션 처리가 되어 있어야, 해당 요청을 프록시 객체가 대신 처리할 수 있다.

## customized AOP

https://cobbybb.tistory.com/25

## @Async

## References.

- https://velog.io/@ann0905/AOP%EC%99%80-Transactional%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC
- https://coding-factory.tistory.com/711
