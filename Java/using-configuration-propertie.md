## @ConfigurationProperties 사용하기

@ConfigurationProperties

_.properties 나 _.yml 파일에서 프로퍼티를 설정하고, 자바 클래스에서 값을 바인딩하여 사용할 수 있게 하는 어노테이션.
key - value 형태로 저장되어 관리된다.
@Value 와 동일한 동작을 수행하나, 문자열 오타 등을 방지하여 편리하게 사용할 수 있다.

1. 디펜던시 추가

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
</dependency>
```

2. 클래스 파일 생성 후, `@ConfigurationProperties(prefix = "site-url")` 등 prefix 설정

```
@Component
@ConfigurationProperties(prefix = "site-url")
@Data
public class siteUrlProperties {
	private String naver; // yml 파일의 site-url.naver 값 바인딩
	private String google; // yml 파일의 site-url.google 값 바인딩

}
```

application-dev.yml, application-prod.yml 과 같이 환경 별로 설정정보를 나누어 관리할 경우, git 에 운영환경 정보가 노출되어 보안 위험이 있다.
또한 환경 설정을 변경하기 위해 git 반영 - 빌드 - 배포의 CI/CD 프로세스가 진행되어야 하는 번거로움이 있다.
이러한 번거로움을 줄이기 위해, 운영과 같은 배포 환경은 OS 환경변수를 이용해 Spring boot 기동 시점에 설정 정보를 주입한다.

(OS environment variables가 Config data (ex application.properties) 보다 우선순위가 높기 때문에 이와 같이 가능하다)

> Spring Boot uses a very particular PropertySource order that is designed to allow sensible overriding of values. Properties are considered in the following order (with values from lower items overriding earlier ones):
>
> 1.  Default properties (specified by setting SpringApplication.setDefaultProperties).
> 2.  @PropertySource annotations on your @Configuration classes. Please note that such property sources are not added to the Environment until the application context is being refreshed. This is too late to configure certain properties such as logging._ and spring.main._ which are read before refresh begins.
> 3.  Config data (such as application.properties files)
> 4.  A RandomValuePropertySource that has properties only in random.\*.
> 5.  OS environment variables.
> 6.  Java System properties (System.getProperties()).
> 7.  JNDI attributes from java:comp/env.
> 8.  ServletContext init parameters.
> 9.  ServletConfig init parameters.
> 10. Properties from SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable or system property).
> 11. Command line arguments.
> 12. properties attribute on your tests. Available on @SpringBootTest and the test annotations for testing a particular slice of your application.
> 13. @TestPropertySource annotations on your tests.
> 14. Devtools global settings properties in the $HOME/.config/spring-boot directory when devtools is active.
>     출처 : https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config

## Kubernetes Configmap 을 사용하기

쿠버네티스 환경에서는 configmap 을 사용하면 편리하게 환경설정 파일을 관리할 수 있다.
configmap은 key-value 쌍의 데이터를 저장하는 API 오브젝트로, POD는 configmap을 환경변수로 사용할 수 있다.

configmap 에 설정한 값을 OS 환경변수로 적용하면, Spring boot 기동 시 환경변수를 읽어 설정 값을 적용하게 된다.

이와 같이 configmap 을 사용하면 파드의 종류, 개수가 많아도, configmap의 설정만으로 각 어플리케이션을 관리할 수 있게 된다.

## References

- https://programmer93.tistory.com/47
- http://dveamer.github.io/backend/BindingFromEnvironmentsVariables.html
