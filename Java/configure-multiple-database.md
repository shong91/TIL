# @Primary 와 @Qualifier (2) - 활용편: 2개 이상의 데이터베이스 연결하기

앞선 [@Primary 와 @Qualifier (1) - 기본편](/Java/primary-qualifier.md) 에서, 어노테이션의 기능과 사용법에 대하여 알아보았다. 이 내용을 바탕으로, 최근 실무에 적용하였던 사례를 정리해보고자 한다.

최근 multiple database 를 구성하는 태스크가 있어, 두 개의 데이터베이스 중 하나를 메인 DB 로, 다른 하나를 서브 DB 로 정의하여 환경을 구성하였다.

데이터베이스 스키마에 따라 myBatis 와 JPA 를 분리하여 사용하는 것으로 하였다. 이에 따라 메인 DB 는 myBatis mapper 를 스캔할 수 있도록 하는 설정을 추가하고, 서브 DB 에는 entityManager 에 대한 설정을 추가하였다.

1. 메인 DB 의 DataSource 설정

- 메인 데이터베이스 커넥션에 @Primary 를 사용하여 default 로 설정한다
- 메인 데이터베이스에 대한 property 를 읽을 수 있도록 `@ConfigurationProperties` 를 설정한다
- 패키지 디렉토리 (`repository`) 에 있는 모든 파일을 Mapper 로 읽어들일 수 있도록 `@MapperScan` 어노테이션에 베이스 패키지를 추가한다

```
@Configuration
@MapperScan(
    basePackages = {
      "com.sample.myapp.aaa.*.repository"
    })
public class DataSourceConfiguration {
  @ConfigurationProperties(prefix = "spring.datasource.main-db")
  @Bean
  @Primary
  public DataSource mainDbDataSource() {
    return DataSourceBuilder.create().build();
  }

}

```

2. 서브 DB 의 DataSource 설정

- 서브 데이터베이스 커넥션에 @Qualifier 로 특정하여 명시적으로 의존성을 주입받아 사용할 수 있도록 한다
- 서브 데이터베이스에 대한 property 를 읽을 수 있도록 `@ConfigurationProperties` 를 설정한다
- JPA 의 트랜잭션 매니저를 설정하기 위해 EntityManager, JpaTransactionManager 빈을 생성한다.
- `@EnableJpaRepositories`
  - JPA Repository들을 활성화하기 위한 어노테이션
  - 기본적으로 해당 Config 클래스의 하위 패키지를 스캔하며, 특정 패키지의 파일만 스캔하도록 하기 위해 베이스 패키지를 설정하였다
  - 설정한 패키지게 있는 파일을 JPA Repository 로 생성한다
- EntityManager 생성
  - @Qualifier 로 특정하여, 서브DB 의 datasource 를 이용하여 LocalContainerEntityManagerFactoryBean 을 생성한다
  - 이 때, packages() 에 Entity 로 생성할 클래스 파일을 관리하는 패키지를 베이스 패키지로 설정한다
    - `@EnableJpaRepositories` 의 베이스 패키지는 JPA Repository 생성 대상을 설정하며
    - `EntityManager` 의 베이스 패키지는 Entity 생성 대상을 설정한다는 점에 주의하자
  - set JpaPropertyMap
    - property 파일에 JPA 관련 프로퍼티를 설정하였으나, multiple datasource 설정을 하며 해당 프로퍼티를 읽지 못하는 이슈가 발생했다
    - EntityManager 를 직접 생성해주어야 함에 따라, JPA properties 역시 직접 설정해주어야 하는 것으로 확인하였다
    - 이에 property 값을 읽어와 소스 레벨에서 값을 set 하여 주었다
- JpaTransactionManager 생성
  - EntityManager 와 마찬가지로, JpaTransactionManager 역시 직접 빈을 생성해주어야 한다

```
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = {"com.sample.myapp.bbb.*.repository"},
    entityManagerFactoryRef = "subDbEntityManagerFactory",
    transactionManagerRef = "subDbTransactionManager")
@RequiredArgsConstructor
public class DataSourceConfiguration {
  private final Environment env;

  @ConfigurationProperties(prefix = "spring.datasource.sub-db")
  @Bean
  public DataSource subDbDataSource() {
    return DataSourceBuilder.create().build();
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean subDbEntityManagerFactory(
      @Qualifier("subDbDataSource") DataSource dataSource, EntityManagerFactoryBuilder builder) {
    LocalContainerEntityManagerFactoryBean em =
        builder
            .dataSource(dataSource)
            .packages(
                "com.sample.myapp.bbb.*.repository.domain")
            .build();

    HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
    em.setJpaVendorAdapter(vendorAdapter);
    em.setJpaPropertyMap(
        Map.of(
            "hibernate.dialect",
            env.getProperty("spring.jpa.database-platform"),
            "hibernate.hbm2ddl.auto",
            env.getProperty("spring.jpa.hibernate.ddl-auto"),
            "hibernate.physical_naming_strategy",
            env.getProperty("spring.jpa.hibernate.physical_naming_strategy"),
            "hibernate.show_sql",
            env.getProperty("spring.jpa.properties.hibernate.show_sql"),
            "hibernate.format_sql",
            env.getProperty("spring.jpa.properties.hibernate.format_sql"),
            "hibernate.default_batch_fetch_size",
            env.getProperty("spring.jpa.properties.hibernate.default_batch_fetch_size")));

    return em;
  }

  @Bean
  public PlatformTransactionManager subDbTransactionManager(
      @Qualifier("subDbEntityManagerFactory")
          LocalContainerEntityManagerFactoryBean entityManagerFactory) {
    return new JpaTransactionManager(Objects.requireNonNull(entityManagerFactory.getObject()));
  }
}

```

3. application.yml 설정

```
spring:
  datasource:
    main-db:
      jdbc-url: #{DB_URL}
      username: #{USERNAME}
      password: #{PASSWORD}
      driver-class-name: com.mysql.cj.jdbc.Driver
    sub-db:
      jdbc-url: #{DB_URL}
      username: #{USERNAME}
      password: #{PASSWORD}
      driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    database-platform: org.hibernate.dialect.MySQL57Dialect
    hibernate:
      ddl-auto: none
      physical_naming_strategy: com.sample.myapp.config.UppercaseSnakePhysicalNamingStrategy
    properties:
      hibernate:
        show_sql: true
        format_sql: true
        default_batch_fetch_size: 100

mybatis:
  mapper-locations: mapper/**/*.xml
  configuration:
    map-underscore-to-camel-case: true

```

## References.

- https://www.baeldung.com/spring-boot-configure-multiple-datasources
