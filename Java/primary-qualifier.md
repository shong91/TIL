## @Primary 와 @Qualifier

스프링 애플리케이션에서, 같은 타입의 빈을 두 개 이상 사용할 때 @Primary 혹은 @Qualifier 을 사용한다.

두 어노테이션의 기능과 공통점/차이점을 정리하고, 어떻게 사용하는 것이 효율적인 사용법인지 정리해보자 한다.

### Primary

`@Primary` 는 같은 타입의 빈을 2개 이상 생성할 때, 하나의 빈에게 더 높은 선호도(higer preference) 를 부여하기 위해 사용한다.

_Simply put, we use @Primary to give higher preference to a bean when there are multiple beans of the same type._

**_왜 `@Primary` 가 필요할까?_**

스프링 컨테이너가 올라갈 때, 스프링은 컴포넌트를 스캔하며 빈을 생성한다. 스프링은 싱글톤 전략을 채택하기 때문에, 한 가지 타입의 빈은 한 번만 생성되며, 애플리케이션이 구동되는 동안 하나의 빈을 계속하여 사용한다.
하지만, 아래 코드와 같이 `@Bean` 으로 생성하는 객체들 중 같은 클래스(타입)인 빈이 있다면, 스프링은 어느 것을 빈으로 생성하여야 하는지 알 수 없기 때문에 컨테이너를 띄우지 못하고 `NoUniqueBeanDefinitionException` 이라는 예외를 던진다.

```
@Configuration
public class Config {

    @Bean
    public Employee JohnEmployee() {
        return new Employee("John");
    }

    @Bean
    public Employee TonyEmployee() {
        return new Employee("Tony");
    }
}
```

그렇기 때문에, 같은 타입의 multiple bean 을 생성하기 위하여, 추가적인 작업이 필요하다. (`@Primary`, `@Qualifier`)

먼저, `@Primary` 를 이용하는 방법이다.
TonyEmployee() bean 에 `@Primary`를 붙여, 스프링이 TonyEmployee() bean 의 의존성을 JohnEmployee() 보다 먼저 주입하도록 설정한다. 즉, TonyEmployee 을 default bean 으로 생성한다.

```
@Configuration
public class Config {

    @Bean
    public Employee JohnEmployee() {
        return new Employee("John");
    }

    @Bean
    @Primary
    public Employee TonyEmployee() {
        return new Employee("Tony");
    }
}
```

또한, `@Component` 로 생성되는 빈에 어노테이션을 붙여서도 가능하다.

아래와 같이 Manager 의 구현체로 GeneralManager, DepartmentManager 두 개의 클래스가 존재하며, GeneralManager를 `@Primary` 로 설정하였다.

이렇게 설정한다면, 스프링 컨테이너가 구동되며 컴포넌트를 스캔하는 과정에서 `@Primary` 를 함께 읽게 된다.

```
@Component
@Primary
public class GeneralManager implements Manager {
    @Override
    public String getManagerName() {
        return "General manager";
    }
}

...

@Component
public class DepartmentManager implements Manager {
    @Override
    public String getManagerName() {
        return "Department manager";
    }
}
```

서비스 레이어에서 의존성 주입된 Manager 클래스를 사용하기 위해 아래와 같이 선언하였다.

DepartmentManager와 GeneralManager 모두 autowiring 될 자격이 있다. 하지만 우리가 GeneralManager를 `@Primary` 로 생성하였기 때문에, 의존성 주입을 위해 GeneralManager 가 선택된다.

```
@Service
public class ManagerService {

    @Autowired
    private Manager manager;

    public Manager getManager() {
        return manager;
    }
}
```

### @Qualifier

두 번째 방법은 `@Qualifier` 를 사용하는 방법이다.

**_왜 `@Qualifier` 가 필요할까?_**

`@Autowired` 어노테이션은 스프링에서 빈에 의존성을 주입하기 위해 사용되는 방법이다. 이 방법은 아주 유용하여 매우 자주 사용된다.
스프링은 타입으로 해당 빈을 찾는다. `@Autowired` 를 통한 의존성 주입 시, 같은 타입의 빈이 하나 이상이라면, autowiring 할 대상이 unique 하지 않기 때문에 마찬가지로 `NoUniqueBeanDefinitionException` 을 던지게 된다.

_By default, Spring resolves autowired entries by type. If more than one bean of the same type is available in the container, the framework will throw NoUniqueBeanDefinitionException, indicating that more than one bean is available for autowiring._

아래와 같이 스프링이 주입할 수 있는 bean 후보가 2개가 있다.
Formatter 의 구현체로 FooFormatter 와 BarFormatter 가 빈으로 생성되고, 서비스 레이어에서 의존성을 주입 받도록 하고자 한다. 하지만 스프링은 두 개의 빈 중 어느 것을 주입하여야 하는지 알지 못하기 때문에, NoUniqueBeanDefinitionException 을 던지게 된다. 이를 해결하기 위하여 `@Qualifier` 를 사용할 수 있다.

```
@Component("fooFormatter")
public class FooFormatter implements Formatter {

    public String format() {
        return "foo";
    }
}

@Component("barFormatter")
public class BarFormatter implements Formatter {

    public String format() {
        return "bar";
    }
}

@Component
public class FooService {

    @Autowired
    private Formatter formatter;
}
```

`@Qualifier` 를 사용하므로써, 우리는 어떤 빈이 주입되어야 하는지에 대한 이슈를 제거할 수 있다. 방금 전 코드에 @Qualifier 를 붙여 다시 살펴보자.

```
public class FooService {

    @Autowired
    @Qualifier("fooFormatter")
    private Formatter formatter;
}
```

`@Qualifier` 어노테이션을 사용할 때에, 우리가 사용하고자 하는 '특정한' 구현체의 이름을 함께 적어준다. 이렇게 함으로써 스프링이 같은 타입의 multiple bean을 모호함 없이 제대로 찾을 수 있도록 한다.

_By including the @Qualifier annotation, together with the name of the specific implementation we want to use, in this example Foo, we can avoid ambiguity when Spring finds multiple beans of the same type._

`@Primary` 와 마찬가지로, Component 와 함께 사용하여 스프링이 컴포넌트를 스캔하는 과정에서 `@Qualifier` 를 함께 읽게 할 수 있다.

```
@Component
@Qualifier("fooFormatter")
public class FooFormatter implements Formatter {
    //...
}

@Component
@Qualifier("barFormatter")
public class BarFormatter implements Formatter {
    //...
}

```

### @Qualifier vs @Primary

- `@Primary` 는, 같은 타입의 multiple beans 이 존재할 때에, 이들 간의 선호도 를 정의하여, **하나의 구현체를 default 값으로 사용**하게 하는 것이다. 기본값으로 주입되어야 하는 빈을 특정하고 싶다면, `@Primary` 를 사용하는 것이 유용한다.
- 다른 빈을 같은 injection point 에서 사용하여야 하는 순간도 있을 것이다. 이 경우 `@Qualifier` 를 사용하여, **default 로 주입된 빈이 아닌, 다른 특정 빈을 주입하도록** 할 수 있다.

💡 @Qualifier, @Primary 어노테이션이 둘 다 존재할 때에는, `@Qualifier` 가 우선한다. 기본적으로 Primary 는 default 값을 정의하는 반면, Qualifier 는 specific 한 값을 정의하기 때문이다.

_It's worth noting that if both the @Qualifier and @Primary annotations are present, then the @Qualifier annotation will have precedence. Basically, @Primary defines a default, while @Qualifier is very specific._

## 실무 적용하기

최근 multiple database 를 구성하며, datasource 를 두 개 설정하여야 하는 일이 있었다.

이 때 두 개의 데이터베이스 중 하나를 메인 DB, 다른 하나를 서브 DB 라고 정의하여,

- 메인 데이터베이스 커넥션에 @Primary 를 사용하여 default 로 설정하고,
- 서브 데이터베이스 커넥션에 @Qualifier 로 특정하여 명시적으로 의존성을 주입받아 사용할 수 있도록 하였다.

## References.

- https://www.baeldung.com/spring-primary
- https://www.baeldung.com/spring-qualifier-annotation
