# Map, Iterator

```
// create Map: Map 의 순서를 고정하고자 한다면 LinkedHashMap 사용
Map<String, String> map = new LinkedHashMap<String, String>();
map.put("contract", exceldto.getContractcode());
map.put("year", exceldto.getChargeyear());
map.put("month", exceldto.getChargemonth());

// map to iterator
Iterator<Map.Entry<String, String>> iterator = contractMap.entrySet().iterator();
while (iterator.hasNext()) {
  Map.Entry<String, String> entry = iterator.next();
  System.out.println(entry.getKey() + " : " + entry.getValue());
}

```
