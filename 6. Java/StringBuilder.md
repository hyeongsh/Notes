java.lang 패키지의 일부로 변경 가능한 문자열을 다룰 때 사용한다.
StringBuilder를 사용하면 기존 객체 안에서 문자열을 수정할 수 있어 성능이 향상된다.
### 문자열 추가
`sb.append("string");`
문자열을 이어붙일 때 사용한다.
### 문자열 삽입
`sb.insert(index, "string");`
특정 위치에 문자열을 삽입할 수 있다.
### 문자열 삭제
`sb.delete(index1, index2);`
특정 범위의 문자열을 삭제한다.
`sb.deleteCharAt(index);`
특정 문자 하나만 삭제한다.
### 문자열 변경
`sb.replace(index1, index2, "string");`
특정 범위의 문자열을 다른 문자열로 교체할 수 있다.
### 문자열 뒤집기
`sb.reverse();`
문자열을 뒤집는다.
### 문자 추출
`sb.charAt(index);`
특정 인덱스의 문자를 반환한다.
`sb.substring(index1, index2);`
특정 범위의 문자열을 반환한다.
### 문자열 길이 확인, 용량 관리
`sb.length();`
`sb.capacity();`
`sb.ensureCapacity(num);`
길이를 체크하거나, 현재 용량을 체크할 수 있으며, 최소 용량을 지정할 수 있다.

