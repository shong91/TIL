# Java Object 를 Map 으로 변경하기 (ObjectMapper)

Jackson databind 라이브러리를 사용하여 객체를 Map 타입으로 변환할 수 있다.

1. pom.xml 에 디펜던시 추가

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.0</version>
</dependency>
```

2. 클래스 생성

```
@Data
public class Person {
    private int age;
    private String name;

    public Person() {}
    public Person(int age, String name) {
        this.age = age;
        this.name = name;
    }
    @Override
    public String toString() {
        return "Person {" + "age=" + age + ", name='" + name + '}';
    }
}

```

3. ObjectMapper 를 사용하여 변환

```
// create Object
Person person = new Person(20, "Kim");

// ObjectMapper
ObjectMapper objectMapper = new ObjectMapper();

// convert object to map
Map<String, Object> map = objectMapper.convertValue(person, Map.class);

for(String key : map.keySet()) {
  System.out.println(key + " : " + map.get(key));
}

// convert map to object
Person convertedPerson = objectMapper.convertValue(map, Person.class);
System.out.println(convertedPerson);
```
