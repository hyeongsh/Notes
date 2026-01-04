HashSet 을 주로 사용하며 Set 인터페이스의 구현체이다.
중복을 허용하지 않는 자료구조이다.
입력한 순서가 보장되지 않으며, null 또한 데이터로 입력이 가능하다는 특징을 가지고 있다.
TreeSet을 사용하여 오름차순 정렬을 보장받을 수 있다.
LinkedHashSet을 사용할 경우에는 데이터가 입력한 순서대로 저장된다.
원리는 데이터를 입력할 경우, 기존 저장된 객체 중 같은 hashCode()를 찾고, 같으면 equals() 메서드를 통해 동일 객체인지 판단한 후 동일 객체가 아닐 때에만 데이터가 저장이 되는 구조.
### Set 선언 방식
`Set<Integer> set = new HashSet<>();`
### 데이터 추가
`set.add(value);`
### 데이터 포함 여부 확인
`set.contains(value);`
### 데이터 삭제
`set.remove(value);`
`set.clear();`
### 사이즈 확인
`set.size();`
### 데이터 출력
`set.toString();`

``` java
Iterator iter = set.iterator();
while (iter.hasNext()) {
	System.out.println(iter.next());
}
```
### List 로 변환
`ArrayList<Integer> list = new ArrayList<>(set);`
