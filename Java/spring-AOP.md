# Spring AOP 이해하기

예전에 면접에서 "@Transactional의 동작 원리에 대하여 설명해주세요." 라는 질문을 받은 적이 있다.

당시에 제대로 답변하지 못했고, 이를 계기로 Spring AOP 에 대하여 깊게 이해하려 하지 않았다는 걸 깨달았다. 정리 해야지 생각만 하고 미뤄뒀었는데, 이번 기회에 개념을 다잡아보고자 한다.

## AOP (Aspect Oriented Programming) ?

**AOP(Aspect Oriented Programming; 관점 지향 프로그래밍) 는**, 횡단 관심사(cross-cutting concerns) 를 분리함으로써 모듈성을 높이는 것을 목표로 하는 프로그래밍 패러다임이다. 이는 코드 자체를 수정하지 않고, 기존에 존재하는 코드에 동작을 추가하여 수행하는 방식이다.

_AOP is a programming paradigm that aims to increase modularity by allowing the separation of cross-cutting concerns. It does this by adding additional behavior to existing code without modifying the code itself._

있는 그대로 번역하니 뭔가 장황하고 잘 와닿지가 않는다. 횡단 관심사는 또 무엇이란 말인가?

> **_횡단 관심사(cross-cutting concerns) ?_**

객체 지향 프로그래밍에서는 주요 관심사에 따라 클래스를 분할한다. 이 클래스들은 보통 SRP(Single Responsibility Principle) 원칙에 따라 하나의 책임만을 갖게 설계된다.

하지만 클래스를 설계하다보면 로깅, 보안, 트랜잭션 등 여러 클래스에서 공통적으로 사용하는 부가 기능들이 생긴다. 이들은 주요 비즈니스 로직은 아니지만, 반복적으로 여러 곳에서 쓰이게 된다. 이들은 여러 모듈에 공통적으로 사용되며, 하나의 모듈에서 코드가 변경될 경우 다른 다른 모듈에서도 동일하게 수정되어야 한다.

<u>이렇듯 "하나의 변경으로 인해 다른 관심사에 영향을 미치는 프로그램의 애스팩트"를 횡단 관심사(cross-cutting concerns)라고 한다.</u>

**횡단 관심사를 각 모듈에서 각각 관리한다면 어떤 문제가 발생할까?**

1. 여러 곳에서 반복적인 코드를 작성해야 한다.
2. 코드가 변경될 경우 여러 곳에 가서 수정이 필요하다.
3. 주요 비즈니스 로직과 부가 기능이 한 곳에 섞여 가독성이 떨어진다.

이러한 문제를 해결하기 위해 AOP 가 등장하였다.
AOP 는 횡단 관심사를 별도의 클래스로 모듈화하여, 이것이 변경되더라도 다른 모듈들에게 영향을 미치지 않도록 한다. 이를 통해 객체 지향 프로그래밍 원칙을 더 잘 지킬 수 있도록 하는 것이 AOP 의 핵심이다.

## AOP의 주요 개념

![](https://www.baeldung.com/wp-content/uploads/2017/11/Program_Execution.jpg)

- Business Object: 일반적인 비즈니스 로직을 가지고 있는 클래스
- Aspect: 여러 개의 클래스에 횡단 관심사로써 존재하는 모듈 (ex 통합 로깅 클래스)
- JoinPoint: 메서드의 실행이나 예외처리 등, 프로그램의 실행 동안의 포인트(지점). Spring AOP에서 JoinPoint 는 항상 메서드의 실행을 나타낸다.
- Advice: 특정 JoinPoint 에서 Aspect 가 적용될 액션. Advice 타입에는 around, before, after 등이 있다. Spring AOP에서 Advice 는 interceptor 로 모델링된다.
- Pointcut: JoinPoint 에서 Aspect 가 적용될 Advice 를 매치시키는 데 도움이 되는 술어
- Target: 하나 이상의 Aspect 에 의해 Advice 되는 객체. Spring AOP 는 런타임 프록시를 사용하여 구현되므로, Target 은 항상 프록시 된 객체이다.

- _Aspect: A modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented by using regular classes (the schema-based approach) or regular classes annotated with the @Aspect annotation (the @AspectJ style)._
- _Join point: A point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution._
- _Advice: Action taken by an aspect at a particular join point. Different types of advice include "around", "before", and "after" advice. (Advice types are discussed later.) Many AOP frameworks, including Spring, model an advice as an interceptor and maintain a chain of interceptors around the join point._
- _Pointcut: A predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default._
- _Introduction: Declaring additional methods or fields on behalf of a type. Spring AOP lets you introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an IsModified interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)_
- _Target object: An object being advised by one or more aspects. Also referred to as the "advised object". Since Spring AOP is implemented by using runtime proxies, this object is always a proxied object._
- _AOP proxy: An object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy is a JDK dynamic proxy or a CGLIB proxy._
- _Weaving: linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime._

## Spring AOP

**Spring AOP 는 기본적으로 프록시 방식으로 동작한다.** 프록시는 대리인이라는 뜻으로, 무언가를 대신 처리한다는 의미이다. 프록시 패턴이란 어떤 객체를 사용하고자 할 때, 객체를 직접 참조하는 것이 아닌, 해당 객체를 대리하는 객체(proxy) 를 통해 대상 객체에 접근하는 방식을 말한다.

앞서 Target은 하나 이상의 Aspect 에 의해 Advice 되는 객체이며, 항상 프록시 된 객체로 생성된다고 하였다. **_Spring 은 왜 Aspect/Advice에 직접적으로 의존하지 않고, 프록시 객체를 사용할까?_**

프록시 객체 없이 Target 객체를 직접 참조하여 사용하고 있다고 생각해보자. Aspect 클래스에 정의된 부가 기능을 사용하기 위해서, 우리는 원하는 위치에서 직접 Aspect 클래스를 호출해야 한다.

<u>이 경우 Target 클래스 안에 부가 기능을 호출하는 로직이 포함되기 때문에, AOP를 적용하지 않았을 때와 동일한 문제가 발생한다. 여러 곳에서 반복적으로 Aspect를 호출해야 하고, 그로 인해 유지보수성이 크게 떨어진다.</u>

그래서 Spring에서는 Target 클래스 혹은 그의 상위 인터페이스를 상속하는 프록시 클래스를 생성하고, 프록시 클래스에서 부가 기능에 관련된 처리를 한다. 이를 통해 Target에서 Aspect을 알 필요 없이, 순수한 비즈니스 로직에 집중할 수 있다.

> **_AOP Proxy 의 종류_**

1. JDK dynamic proxy

- Target 이 하나 이상의 인터페이스를 구현하고 있는 클래스라면 JDK dynamic proxy 방식으로 프록시를 생성한다.
- JDK dynamic proxy 란, Java의 리플렉션 패키지에 존재하는 Proxy 클래스로 생성된 프록시 객체이다. 리플랙션의 Proxy 클래스가 동적으로 프록시를 생성하며, Target 의 인터페이스를 기준으로 프록시를 생성한다.
- Spring AOP 는 JDK dynamic proxy 를 디폴트 프록시 방식으로 사용한다.
  _Spring AOP defaults to using standard JDK dynamic proxies for AOP proxies._

즉, 구현체는 인터페이스를 상속 받아야 하고, @Autowired 를 통해 프록시 빈을 사용하기 위해서는 반드시 인터페이스 타입으로 지정해 주어야 한다. 때문에, 인터페이스 타입이 아니라 클래스 타입으로 의존성을 주입하고자 한다면 런타임 에러가 발생할 수 있다.

```
@Controller
public class UserController {
  @Autowired
  private UserServiceImpl userService; // <- Runtime Error 발생
  ...
}

@Service
public class UserServiceImpl implements UserService {
  ...
}
```

2. CGLib Proxy

- Target 이 인터페이스를 구현하지 않은 클래스라면, CGLib proxy 방식으로 프록시를 생성한다.
- 이는 인터페이스가 아닌 프록시 클래스가 필요하다. 기본적으로 Business Object 가 인터페이스를 구현하지 않는 경우 CGLib 프록시가 사용된다.

어라? 근데 Service 클래스를 빈으로 생성할 때 런타임 에러가 발생한 적은 없는데?

좀 더 찾아보니, 과거에는 CGLib 의 성능적 한계로 Spring은 JDK proxy 를 기본으로 채택하고, CGLib 는 별도의 의존성을 추가하여야 하였다고 한다. 하지만 CGLib 가 개선되어 안정화 됨에 따라, Spring 3.2 부터 CGLib 을 spring core 패키지에 포함시켜 별도의 의존성을 추가하지 않아도 사용할 수 있도록 수정되었다. ([관련 Spring Boot issues](https://github.com/spring-projects/spring-boot/issues/8434))

지금 Spring 6.0 / Spring boot 3.0 이 나왔으니 꽤나 이전에 업데이트 된 사항이라, 내가 개발을 시작했을 때에는 이미 CGLib 이 스프링 코어에 들어가 있었기에 전혀 모르고 있던 부분이었다. 공부하면서 스프링의 히스토리를 +1 알게 되어 신기했다 ㅎㅎㅎ

## References.

- https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core
- https://www.baeldung.com/spring-aop
- https://velog.io/@ann0905/AOP%EC%99%80-Transactional%EC%9D%98-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC
- https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html
