# Map, Iterator

create Map: Map 의 순서를 고정하고자 한다면 LinkedHashMap 사용한다.

```
Map<String, String> map = new LinkedHashMap<String, String>();
map.put("key1", "value1");
map.put("key2", "value2");
map.put("key3", "value3");
map.put("key4", "value4");
map.put("key5", "value5");

```

## 1. Entry 에 For-Each Loop 사용

- 가장 일반적이고 거의 모든 경우에 사용된다.
- 반복문 안에 key값과 value값이 전부 필요할때 사용한다.

```
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
    System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}
```

## 2. Key Value 에 For-Each Loop사용

- entry 대신 key값이나 value 값만 필요할 때 사용한다.
- entrySet 사용 시 보다 약 10% 속도 향상.

```
//iterating over keys only
for (Integer key : map.keySet()) {
    System.out.println("Key = " + key);
}

//iterating over values only
for (Integer value : map.values()) {
    System.out.println("Value = " + value);
}
```

## 3. Iterator 사용

- keySet이나 values 에도 똑같이 적용 가능
- 반복문을 사용하는 도중에 iterator.remove()를 통해 항목들을 삭제할 수 있는 유일한 방법
  (삭제 작업을 For-Each 구문에서 사용한다면 "unpredictable results"를 얻게된다(javadoc, Map은 순서를 보장하지 않기 때문에))
- 성능 측면에서 For-Each와 비슷하다

```
// map to iterator
Iterator<Map.Entry<String, String>> iterator = contractMap.entrySet().iterator();
while (iterator.hasNext()) {
  Map.Entry<String, String> entry = iterator.next();
  System.out.println(entry.getKey() + " : " + entry.getValue());
}
```

## 4. Key 값으로 Value 를 찾는 반복문

- key값을 이용해서 value를 찾는 과정에서 시간이 많이 소모된다. (1번 방법보다 약 20%~200% 성능저하가 있음)

```
for (Integer key : map.keySet()) {
    Integer value = map.get(key);
    System.out.println("Key = " + key + ", Value = " + value);
}
```
