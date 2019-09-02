# MySQL SYSDATE, NOW
- MySQL에서 현재 날짜 및 시간정보를 리턴해주는 Built-in 함수이다.
- SYSDATE()와 NOW() 의 작동 방식은 쿼리의 실행 계획에 상당햔 영향을 미친다.

#### MySQL 메뉴얼
```SQL
SYSDATE() returns the time at which it executes.
This differs from the behavior for NOW(), which returns a constant time that indicates the time at which the statement began to execute.
(Within a stored function or trigger, NOW() returns the time at which the function or triggering statement began to execute.)

mysql> SELECT NOW(), SLEEP(2), NOW();
+---------------------+----------+---------------------+
| NOW()               | SLEEP(2) | NOW()               |
+---------------------+----------+---------------------+
| 2010-12-13 14:52:34 |        0 | 2010-12-13 14:52:34 |
+---------------------+----------+---------------------+

mysql> SELECT SYSDATE(), SLEEP(2), SYSDATE();
+---------------------+----------+---------------------+
| SYSDATE()           | SLEEP(2) | SYSDATE()           |
+---------------------+----------+---------------------+
| 2010-12-13 14:52:43 |        0 | 2010-12-13 14:52:45 |
+---------------------+----------+---------------------+
```

- SYSDATE() 함수는 트랜잭션이나 쿼리 단위에 관계없이 그 함수 실행 시점이 기준이다.
- NOW() 는 하나의 쿼리 단위가 기준이며 동일한 값을 리턴하게 된다.

- 엔티티의 등록일을 SYSDATE() 함수로 넣었다면 한 트랜잭션 단위의 ROW를 처리하는데 1시간이 걸렸다고 가정하자
- 첫번째 ROW와 마지막 ROW의 등록일은 1시간이 차이가 나게된다.
- * 이러한 차이는 Stored-Function의 DETERMINISTIC 과 NOT DETERMINISTIC의 차이와 거의 흡사

- 이로 인해 전체적인 쿼리의 실행 계획을 바꿔버리게 된다.
```SQL
-- USER TABLE 
CREATE TABLE user (
  id mediumint(9) NOT NULL AUTO_INCREMENT,
  name varchar(20) DEFAULT NULL,
  age tinyint(3) unsigned DEFAULT NULL,
  sex enum('MALE','FEMALE') DEFAULT NULL,
  regdt datetime NOT NULL,
  PRIMARY KEY (id),
  KEY ix_regdt (regdt)
) ENGINE=InnoDB;

mysql> explain select * from user where regdt>now();
+----+-------------+-------+-------+----------+---------+------+------+
| id | select_type | table | type  | key      | key_len | ref  | rows |
+----+-------------+-------+-------+----------+---------+------+------+
|  1 | SIMPLE      | user  | range | ix_regdt | 8       | NULL |    1 |
+----+-------------+-------+-------+----------+---------+------+------+
1 row in set (0.00 sec)

mysql> explain select * from user where regdt>sysdate();   
+----+-------------+-------+------+------+---------+------+------+
| id | select_type | table | type | key  | key_len | ref  | rows |
+----+-------------+-------+------+------+---------+------+------+
|  1 | SIMPLE      | user  | ALL  | NULL | NULL    | NULL | 4822 |
+----+-------------+-------+------+------+---------+------+------+
1 row in set (0.00 sec)
```

- SYSDATE() 로 비교되는 컬럼의 인덱스 유무와 상관없이 FULL_TABLE_SCAN 이 일어난다.

#### 해결 방법
- SYSDATE() 함수 사용 지양
- --sysdate-is-now 옵션을 활성화
    - 기본값은 false
    - 활성화되면 SYSDATE() 와 NOW() 함수는 동일하게 동작한다.

- 참조
  - http://intomysql.blogspot.com/2010/12/sysdate-now.html
