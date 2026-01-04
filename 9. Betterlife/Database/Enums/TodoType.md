## 목적
할 일(Todo)의 분류를 나타내는 Enum
## Enum 값 목록
| Enum Name    | 의미      |
| ------------ | ------- |
| `GENERAL`    | 일반 할 일  |
| `WORK_STUDY` | 업무 및 학습 |
| `WORKOUT`    | 운동      |
| `SCHEDULE`   | 일정      |
## DB 매핑
| 항목        | 값                                                   |
| --------- | --------------------------------------------------- |
| 컬럼 타입     | `VARCHAR(20)`                                       |
| 저장 형식     | 문자열(`GENERAL`, `WORK_STUDY`, `WORKOUT`, `SCHEDULE`) |
| JPA 어노테이션 | `@Enumerated(EnumType.STRING)`                      |
