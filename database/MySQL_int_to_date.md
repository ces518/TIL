# MySQL 숫자를 날짜타입으로 변경
```sql
SELECT DATE_FORMAT(FROM_UNIXTIME(regdate), '%Y-%m-%d')
```
