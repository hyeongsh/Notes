## SELECT 문
```SQL
SELECT column_name
	FROM table_name
	WHERE condition
	GROUP BY column_name
	HAVING condition
	ORDER BY column_name
	LIMIT number;
```
### WHERE 종류
```SQL
-- between
WHERE column_name between A and B;
-- in()
WHERE column_name in(A, B, C);
-- like
WHERE column_name like 'A___';
WHERE column_name like 'A%';
WHERE column_name like '%A%';
```
### ORDER BY
WHERE 절 다음에 나와야 한다.
ASC는 오름차 순으로 기본 정렬 방식이고, DESC는 내림차순으로 직접 지정해줘야 사용 가능하다.
## CREATE 문
만약 db나 table이 존재하는 걸 무시하고 처음부터 깔고 싶을때가 있다.
`drop database if exists sqlDB;`
`drop table if exists sqlTable;`
### create table
```sql
CREATE TABLE table_name (
	column1 BIGINT PRIMARY KEY AUTO_INCREMENT,
	column2 VARCHAR(10) NOT NULL,
	column3 INT,
	
	PRIMARY KEY(column1),
	FOREIGN KEY(column2) REFERENCES table2(id)
);
```