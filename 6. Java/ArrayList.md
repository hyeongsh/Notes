자바의 컬렉션 중 가장 기본적인 컬렉션
배열의 상위호환 버전 정도로 이해하면 된다.
연속적인 데이터 리스트이며, 내부적으로 Object를 상속한 요소만을 넣을 수 있다.
크기가 고정되어 있지 않으므로 가변적으로 공간을 사용할 수 있다.
리스트 중간 삽입/삭제는 빈 공간을 없애는 과정 때문에 동작이 느리다.
### 생성
``` java
ArrayList<Integer> members = new ArrayList<>();
ArrayList<Integer> members = new ArrayList<>(10);
ArrayList<Integer> members = new ArrayList<>(Arrays.asList(1, 2, 3));
ArrayList<Integer> members = new ArrayList<>(members2);
```
### 요소 추가
`arrayList.add(obj);`
`arrayList.addAll(collection);`
### 요소 삽입
`arrayList.add(index, obj);`
`arrayList.addAll(index, collection);`
### 요소 삭제
`arrayList.remove(index);`
`arrayList.remove(obj);`
`arrayList.removeAll(collection);`
`arrayList.clear();`
`arrayList.retainAll(collection);`
### 요소 검색
`arrayList.isEmpty();`
`arrayList.contains(obj);`
`arrayList.indexOf(obj);`
`arrayList.lastIndexOf(obj);`
### 요소 얻기
`arrayList.get(index);`
`arrayList.subList(fromIndex, toIndex);`
### 요소 변경
`arrayList.set(index, obj);`
### 배열 변환
String[]으로 변경할 경우 2번 방식을 사용해야 한다.
`arrayList.toArray();`
`arrayList.toArray(objArr);`
### 정렬
정렬된 값을 반환하지 않고 원본 리스트를 변경시킨다.
내림차순 정렬을 하고 싶을 경우 Comparator.reverseOrder()를 넣어주도록 하자.
`arrayList.sort();`
### Iterator
iterator는 거의 모든 컬렉션에서 사용 가능하며 앞으로만 한 칸씩 읽고 삭제하는 것이 가능하다.
ListIterator는 List에서만 사용이 가능하며 양방향 이동과 중간 수정/삽입이 가능하도록 만들어져 있다.
`arrayList.iterator();`
`arrayList.listIterator();`
`arrayList.listIterator(index);`