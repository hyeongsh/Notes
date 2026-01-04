java.util 패키지의 일부로, 배열을 다루기 위한 다양한 메서드를 제공한다.
특정 값으로 요소를 정렬하거나, 검색 및 채우는 것과 같은 일반적인 배열 작업을 제공해준다.
### 1차원 배열의 출력
`Arrays.toString(arr);`
반복문 없이 값을 출력할 수 있게 해준다.
### 2차원 배열의 출력
`Arrays.deepToString(arr);`
1차원 배열처럼 출력할 경우, 주소값이 출력되므로 deepToString을 이용해 출력한다.
### 1차원 배열의 정렬
`Arrays.sort(arr);`
주어진 배열을 오름차순으로 정렬한다.
내림차순으로 정렬하기 위해서는 Collections.reverseOrder()를 추가해야 하는데, 이 경우 기본 자료형은 포함되지 않으므로, Integer, Long, Double, Character로 변경해서 사용해야 한다.
### 배열에 특정 값 채우기
`Arrays.fill(arr, value);`
배열을 임의값으로 모두 채울 때 사용한다.
### 배열의 값 비교
`Arrays.equal(arr, arr);`
배열 안의 요소가 같은지 확인한다.
### 배열을 리스트로 변환
`Arrays.asList(arr);`
배열을 List로 변환한다.
### 배열의 복사
`Arrays.copyOf(arr, length);`
배열을 특정 길이만큼 복사한다.
입력받은 길이가 더 길다면 기본값으로 채워진다.
`Arrays.copyOfRange(arr, position1, position2)`
배열의 특정 영역을 복사한다.
position2는 미만이다. 포함되지 않는다.

