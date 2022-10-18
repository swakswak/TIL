# [MySQL] 쿼리이력 확인 방법
1. general_log 확인
   ```
    mysql> show variables like 'general%';
    +------------------+---------------------------------+
    | Variable_name    | Value                           |
    +------------------+---------------------------------+
    | general_log      | OFF                             |
    | general_log_file | /var/lib/mysql/5f0a8d857166.log |
    +------------------+---------------------------------+
    ```
2. general_log가 OFF로 되어있다면 ON으로
   ```
   mysql> set global general_log=on;
   Query OK, 0 rows affected (0.06 sec)
   ```
3. general_log 적용 확인
   ```
   mysql> show variables like 'general%';
   +------------------+---------------------------------+
   | Variable_name    | Value                           |
   +------------------+---------------------------------+
   | general_log      | ON                              |
   | general_log_file | /var/lib/mysql/5f0a8d857166.log |
   +------------------+---------------------------------+
   2 rows in set (0.05 sec)
   ```
4. general_log_file에 명시된 경로로 가서 로그 확인
   ```
   root@5f0a8d857166:/# tail -5f /var/lib/mysql/5f0a8d857166.log
   2022-10-18T14:23:58.799746Z	    9 Query	select * from user
   2022-10-18T14:24:20.200861Z	    8 Quit
   2022-10-18T14:25:07.640683Z	    9 Query	select * from user
   2022-10-18T14:25:26.193571Z	    9 Query	insert into user (name) values ('swakswak2')
   2022-10-18T14:25:27.937117Z	    9 Query	select * from user
   ```