## 목적
사용자 todo 관리용 테이블
## 테이블
| 컬럼             | 타입           | 설명                 |
| -------------- | ------------ | ------------------ |
| id             | BIGINT       | 기본키                |
| user_id        | BIGINT       | 작성자 `users.id` 참조  |
| title          | VARCHAR(255) | 할 일 제목             |
| type           | VARCHAR(20)  | [[TodoType]] 참조    |
| status         | VARCHAR(20)  | [[TodoStatus]] 참조  |
| is_recurring   | BOOLEAN      | 반복 여부              |
| repeat_days    | INT          | 반복 주기 (비트마스킹)      |
| parent_todo_id | BIGINT       | 상위 Todo (자식인 경우에만) |
| active_from    | DATE         | 시작일                |
| active_until   | DATE         | 종료일                |
| created_at     | TIMESTAMP    | 생성 시각              |
| updated_at     | TIMESTAMP    | 변경 시각              |
## DDL
```sql
CREATE TABLE todos (
	id BIGINT NOT NULL AUTO_INCREMENT,
	user_id BIGINT NOT NULL,
	title VARCHAR(255) NOT NULL,
	type VARCHAR(20) DEFAULT 'GENERAL',
	status VARCHAR(20) DEFAULT 'PLANNED',
	is_recurring BOOLEAN DEFAULT FALSE,
	repeat_days INT DEFAULT 0,
	parent_todo_id BIGINT,
	active_from DATE,
	active_until DATE,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
	
	PRIMARY KEY (id),
	CONSTRAINT fk_todos_user FOREIGN KEY (user_id)
		REFERENCES users(id) ON DELETE CASCADE
	CONSTRAINT fk_todos_parent FOREIGN KEY (parent_todo_id)
		REFERENCES todos(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

