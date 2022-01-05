# Java Stream 활용하기

## Java stream ?

- 배열/컬렉션을 함수형으로 처리 가능
- 간단한 병렬처리 가능(하나의 작업을 둘 이상으로 쪼개어 동시 진행)

자바 스트림을 사용하여 배열/인스턴스에 여러 함수를 조합하여 원하는 결과를 필터링하고, 가공된 결과를 얻을 수 있으며,

람다식을 사용하여 코드의 가독성을 높이고 간결하게 표현할 수 있다.

- 생성하기
  - 배열 / 컬렉션 / 빈 스트림
  - Stream.builder() / Stream.generate() / Stream.iterate()
  - 기본 타입형 / String / 파일 스트림
  - 병렬 스트림 / 스트림 연결하기
- 가공하기
  - Filtering
  - Mapping
  - Sorting
  - Iterating
- 결과 만들기
  - Calculating
  - Reduction
  - Collecting
  - Matching
  - Iterating

많은 내용이 있지만 일단 자주 쓰이는 것 먼저 정리하려고 한다.

## 가공하기

### map

스트림 내 요소들을 하나씩 특정 값으로 매핑(변환) 해준다.

.map() 내부에 선언한 특정 로직을 거쳐 변환되는 값을 새로운 스트림에 담아 리턴한다.

```
List<String> names = Arrays.asList("Eric", "Elena", "Java");

Stream<String> stream =
  names.stream()
  .map(String::toUpperCase);
// [ERIC, ELENA, JAVA]
```

객체의 수량을 가져올 수도 있으며, 원하는 숫자 자료형으로 변환할 수도 있다. (mapToInt(), mapToDouble(), ..)

```
List<Product> productList =
  Arrays.asList(new Product(23, "potatoes"),
                new Product(14, "orange"),
                new Product(13, "lemon"),
                new Product(23, "bread"),
                new Product(13, "sugar"));

Stream<Integer> stream =
  productList.stream()
  .map(Product::getAmount);
// [23, 14, 13, 23, 13]
```

### filter

스트림 내 요소들이 조건에 맞는지 하나씩 걸러낸다(filtering).

stream 객체로 리턴하게 되므로, list 로 변경하여 사용할 수 있다.

```
Stream<String> stream =
  names.stream()
  .filter(name -> name.contains("a"));
// [Elena, Java]

List<String> list =
 names.stream()
  .filter(name -> name.contains("a"))
  .collect(Collectors.toList());
```

## 결과 만들기

### reduce

스트림 내 엘리먼트를 비교하고 컬렉션에서 하나의 값으로 연산한다.

`reduce` 메서드는 아래와 같이 세 가지의 파라미터를 받을 수 있다.

- accumulator : 각 요소를 처리하는 계산 로직. 각 요소가 올 때마다 중간 결과를 생성하는 로직.
- identity : 계산을 위한 초기값으로 스트림이 비어서 계산할 내용이 없더라도 이 값은 리턴.
- combiner : **병렬(parallel) 스트림에서** 나눠 계산한 결과를 하나로 합치는 동작하는 로직.

```
// 1개 (accumulator)
OptionalInt reduced =
  IntStream.range(1, 4) // [1, 2, 3]
  .reduce((a, b) -> {
    return Integer.sum(a, b);
  });

// 2개 (identity)
int reducedTwoParams =
  IntStream.range(1, 4) // [1, 2, 3]
  .reduce(10, Integer::sum); // method reference

// 3개 (combiner)
Integer reducedParallel = Arrays.asList(1, 2, 3)
  .parallelStream()
  .reduce(10,
          Integer::sum,
          (a, b) -> {
            System.out.println("combiner was called");
            return a + b;
          });
// combiner was called
// combiner was called
// 36
```

### collect

스트림의 값을 하나로 모으는 종료 작업이라고 볼 수 있다.

Collector 타입의 인자를 받아 처리하며, 여러 convenience method를 제공하고 있어 간단하게 사용이 가능하다.

`Collectors.toList()`: 스트림 결과를 리스트로 반환

```
Stream<String> stream =
  names.stream()
  .filter(name -> name.contains("a"))
  .collect(Collectors.toList());
```

`Collectors.joining()`: 스트림 결과를 하나의 스트링으로 이어 붙여 반환

```
List<String> list =
 names.stream()
  .filter(name -> name.contains("a"))
  .collect(Collectors.joining(", "));
// 리스트의 엘리먼트를 콤마로 구분하여 출력
```

`Collectors.groupingBy()`: 특정 조건으로 요소들을 그룹핑

```
Map<Integer, List<Product>> collectorMapOfLists =
 productList.stream()
  .collect(Collectors.groupingBy(Product::getAmount));
```

### Reference.

https://futurecreator.github.io/2018/08/26/java-8-streams/
