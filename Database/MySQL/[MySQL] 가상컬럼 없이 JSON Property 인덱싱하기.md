# [MySQL] 가상컬럼 없이 JSON Property 인덱싱하기

### 테이블, 인덱스 생성
```mysql
CREATE TABLE json_test (
    id bigint auto_increment primary key,
    jsondata json,

    index char_index(((CAST(jsondata->>"$.char" as char(255)) collate utf8mb4_bin))) using BTREE,
    index unsinged_index((CAST(jsondata->>"$.unsigned" as unsigned ))) using BTREE,
    index singed_index((CAST(jsondata->>"$.signed" as signed ))) using BTREE,
    index decimal_index((CAST(jsondata->>"$.decimal" as decimal ))) using BTREE,
    unique index unique_char_index(((CAST(jsondata->>"$.unique" as char(255)) collate utf8mb4_bin))) using BTREE
);

INSERT INTO json_test (jsondata)
VALUES ('{
  "char": "shim",
  "unsigned": 1,
  "signed": -1,
  "decimal": 0.1,
  "unique": "one",
  "without_index": "a"
}');

INSERT INTO json_test (jsondata)
VALUES ('{
  "char": "choi",
  "unsigned": 2,
  "signed": -2,
  "decimal": 0.22,
  "unique": "two",
  "without_index": "b"
}');

INSERT INTO json_test (jsondata)
VALUES ('{
  "char": "kim",
  "unsigned": 3,
  "signed": -3,
  "decimal": 0.333,
  "unique": "three",
  "without_index": "c"
}');
```
  

### 쿼리 실행 결과 확인
```mysql
// char_index
explain select * from json_test where jsondata->>"$.char" = "choi";
+--+-----------+---------+----------+----+-------------+----------+-------+-----+----+--------+-----+
|id|select_type|table    |partitions|type|possible_keys|key       |key_len|ref  |rows|filtered|Extra|
+--+-----------+---------+----------+----+-------------+----------+-------+-----+----+--------+-----+
|1 |SIMPLE     |json_test|null      |ref |char_index   |char_index|1023   |const|1   |100     |null |
+--+-----------+---------+----------+----+-------------+----------+-------+-----+----+--------+-----+
    
// unsinged_index
explain select * from json_test where jsondata->>"$.unsigned" = CAST(1 as unsigned );
+--+-----------+---------+----------+----+--------------+--------------+-------+-----+----+--------+-----------+
|id|select_type|table    |partitions|type|possible_keys |key           |key_len|ref  |rows|filtered|Extra      |
+--+-----------+---------+----------+----+--------------+--------------+-------+-----+----+--------+-----------+
|1 |SIMPLE     |json_test|null      |ref |unsinged_index|unsinged_index|9      |const|1   |100     |Using where|
+--+-----------+---------+----------+----+--------------+--------------+-------+-----+----+--------+-----------+

// singed_index
explain select * from json_test where jsondata->>"$.signed" = CAST(-1 as signed );
+--+-----------+---------+----------+----+-------------+------------+-------+-----+----+--------+-----------+
|id|select_type|table    |partitions|type|possible_keys|key         |key_len|ref  |rows|filtered|Extra      |
+--+-----------+---------+----------+----+-------------+------------+-------+-----+----+--------+-----------+
|1 |SIMPLE     |json_test|null      |ref |singed_index |singed_index|9      |const|1   |100     |Using where|
+--+-----------+---------+----------+----+-------------+------------+-------+-----+----+--------+-----------+

// decimal_index
explain select * from json_test where jsondata->>"$.decimal" = CAST(1 as decimal );
+--+-----------+---------+----------+----+-------------+-------------+-------+-----+----+--------+-----+
|id|select_type|table    |partitions|type|possible_keys|key          |key_len|ref  |rows|filtered|Extra|
+--+-----------+---------+----------+----+-------------+-------------+-------+-----+----+--------+-----+
|1 |SIMPLE     |json_test|null      |ref |decimal_index|decimal_index|6      |const|1   |100     |null |
+--+-----------+---------+----------+----+-------------+-------------+-------+-----+----+--------+-----+

// unique_char_index
explain select * from json_test where jsondata->>"$.unique" = "one";
+--+-----------+---------+----------+-----+-----------------+-----------------+-------+-----+----+--------+-----+
|id|select_type|table    |partitions|type |possible_keys    |key              |key_len|ref  |rows|filtered|Extra|
+--+-----------+---------+----------+-----+-----------------+-----------------+-------+-----+----+--------+-----+
|1 |SIMPLE     |json_test|null      |const|unique_char_index|unique_char_index|1023   |const|1   |100     |null |
+--+-----------+---------+----------+-----+-----------------+-----------------+-------+-----+----+--------+-----+

// 인덱스 없는 컬럼
explain select * from json_test where jsondata->>"$.without_index" = "a";
+--+-----------+---------+----------+----+-------------+----+-------+----+----+--------+-----------+
|id|select_type|table    |partitions|type|possible_keys|key |key_len|ref |rows|filtered|Extra      |
+--+-----------+---------+----------+----+-------------+----+-------+----+----+--------+-----------+
|1 |SIMPLE     |json_test|null      |ALL |null         |null|null   |null|3   |100     |Using where|
+--+-----------+---------+----------+----+-------------+----+-------+----+----+--------+-----------+
```