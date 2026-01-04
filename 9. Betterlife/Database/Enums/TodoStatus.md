## 목적
할 일(Todo)의 진행 상태를 나타내는 Enum
## Enum 값 목록
| Enum Name  | 의미  |
| ---------- | --- |
| `PLANNED`  | 계획됨 |
| `DONE`     | 완료됨 |
| `FAILED`   | 실패함 |
| `CANCELED` | 취소됨 |
## 규칙
- 기본 상태는 `PLANNED` 상태
- UI에서는 색상 구분
## DB 매핑
| 항목        | 값                                            |
| --------- | -------------------------------------------- |
| 컬럼 타입     | `VARCHAR(20)`                                |
| 저장 형식     | 문자열(`PLANNED`, `DONE`, `FAILED`, `CANCELED`) |
| JPA 어노테이션 | `@Enumerated(EnumType.STRING)`               |
