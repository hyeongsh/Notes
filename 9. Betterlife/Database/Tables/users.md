## 목적
사용자 계정 및 인증 관리용 테이블
## 테이블
| 컬럼         | 타입           | 설명       |
| ---------- | ------------ | -------- |
| id         | BIGINT       | 기본키      |
| name       | VARCHAR(100) | 사용자 이름   |
| email      | VARCHAR(255) | 사용자 이메일  |
| password   | VARCHAR(255) | 사용자 비밀번호 |
| created_at | TIMESTAMP    | 생성 시각    |
| updated_at | TIMESTAMP    | 변경 시각    |
## DDL
```sql
CREATE TABLE users (
	id BIGINT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(100) NOT NULL,
	email VARCHAR(255) UNIQUE NOT NULL,
	password VARCHAR(255) NOT NULL,
	created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
	updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

