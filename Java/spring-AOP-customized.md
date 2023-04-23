## Customized AOP 적용하기

Spring AOP 의 개념에 대해 이해하고, 대표적인 AOP 어노테이션인 @Transactional 의 동작 원리에 대하여 살펴 보았다.

개발을 하다 보면 cross-cutting concerns 이 빈번하게 생겨난다. 이런 것들을 하나의 모듈로 만들어 적용 할 수 있도록, AOP 를 통해 Custom Annotation 을 만들어 보자.

### 1. 어노테이션 정의

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
    ...
}
```

메서드의 실행 시간을 로깅하기 위해 위와 같이 `@LogExecutionTime` 어노테이션을 정의하였다. 런타임 시에 어노테이션이 저장되며, 메서드 단위에 적용된다.

| annotation 명 | 내용                                                                | 속성                                                                                 |
| ------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| @Retention    | 어노테이션의 정보 유지 범위를 지정한다.                             | RetentionPolicy.CLASS(default), RetentionPolicy.SOURCE, RetentionPolicy.RUNTIME      |
| @Target       | 어노테이션의 사용 가능 대상을 지정한다.                             | ElementType.TYPE (클래스, 인터페이스, enum),ElementType.FIELD, ElementType.METHOD 등 |
| @Inherited    | 부모 클래스에 어노테이션이 선언되었다면 자식 클래스에게 상속시킨다. |

### 2. 메서드에 어노테이션 선언

어노테이션을 서비스 레이어의 메서드에 선언한다.

```
@LogExecutionTime
public void serve() throws InterruptedException {
    Thread.sleep(2000);
}
```

### 3. Advisor 정의

해당 메서드가 호출되는 시점에 custom annotation을 처리할 수 있도록, Advisor 를 생성한다.

- Target: Advice를 주입할 대상
- Advice: 횡단 관심사를 분리한 모듈
- JoinPoint: Advice가 적용될 수 있는 메서드
- Pointcut: Target을 선정하기 위한 방법

```
@Aspect
@Order(1) //JoinPoint에 여러 Advice가 걸려있을시 Advice 수행 순서를 정함.
@Slf4j
@Component
public class SampleAspect {
    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {

        // custom annotation 에 필드가 존재한다면 타겟 메서드의 Signature를 통해 셋팅된 값을 꺼내올 수 있다.
        // MethodSignature methodSignature = (MethodSignature) proceedingJoinPoint.getSignature();
        // LogExecutionTime custom = methodSignature.getMethod().getAnnotation(LogExecutionTime.class);
        // log.info("execute custom annotation processing with annotation param = {}", custom.field());

        long start = System.currentTimeMillis();

        log.info("Before invoke");

        // 타겟 메서드 수행: 타겟 메서드의 반환값이 proceed() 의 반환값으로 그대로 전달된다.
        Object proceed = joinPoint.proceed();

        log.info("After invoke");

        long executionTime = System.currentTimeMillis() - start;

        log.info("{} executed in {} ms", joinPoint.getSignature(), executionTime);

        return proceed;
    }
}
```

**어드바이스의 종류**

- `@Before`: 타겟이 호출되기 전 어드바이스 내용이 주입, 수행된다.
- `@After`: 타겟이 수행이 끝나면 어드바이스 내용이 주입, 수행된다. (성공/실패 무관)
- `@AfterReturning`: 타겟이 성공적으로 수행되면 결과값을 리턴한 후 어드바이스 내용이 주입, 수행된다.
- `@AfterThrowing`: 타겟이 예외를 던지게 되면 어드바이스 내용이 주입, 수행된다.
- `@Around`: 타겟에 대한 내용 수행 전후를 감싸 어드바이스 내용이 주입, 수행된다.

## References

- https://www.baeldung.com/spring-aop-annotation
