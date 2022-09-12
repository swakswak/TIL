# [MySQL] JSON 타입  컬럼 특정 속성 제거

이 글은 MySQL에서 JSON 타입의 컬럼을 사용할 때 JSON 타입의 컬럼 내에서 특정 속성을 제거하고 조회하는 방법에 대해 설명합니다.

다음과 같은 사용자 정보를 저장한다.

```json
{
	"id": 1,
	"json_data": {
		"cart": ["테스트주도개발", "실용주의프로그래머"],
		"name": "솩솩1",
		"joinedDate": "2020-07-02"
	}
}
```

```json
{
	"id": 2,
	"json_data": {
		"cart": ["클린코드", "클린아키텍처", "도메인주도개발"],
		"name": "솩솩2",
		"joinedDate": "2022-09-12"
	}
}
```

`json_data` 컬럼은 JSON 타입의 컬럼이며, 조회 시 `json_data.cart`는 제외하고 조회하고자 한다.

## 테이블 생성

```sql
create table `user`
(
    `id`   bigint primary key auto_increment,
    `json_data` json
);
```

## 데이터 삽입

```sql
insert into `user` (json_data)
    value (
    '{
      "name": "솩솩1",
      "joinedDate": "2020-07-02",
      "cart": [
        "테스트주도개발",
        "실용주의프로그래머"
      ]
    }'
    );

insert into `user` (json_data)
    value (
    '{
      "name": "솩솩2",
      "joinedDate": "2022-09-12",
      "cart": [
        "클린코드",
        "클린아키텍처",
        "도메인주도개발"
      ]
    }'
    );
```
## 데이터 조회
일반적인 조회이며 결과는 다음과 같다.

```sql
select * from `user`;
```

|  id | json_data                                                                                       |
| --- |-------------------------------------------------------------------------------------------------|
| 1 | {<br/>"cart": ["테스트주도개발", "실용주의프로그래머"],<br/>"name": "솩솩1",<br/>"joinedDate": "2020-07-02"<br/>} |
| 2 | {<br/>"cart": ["클린코드", "클린아키텍처", "도메인주도개발"],<br/>"name": "솩솩2",<br/>"joinedDate": "2022-09-12"<br/>}                |

`cart` 속성을 제외하고 조회하기 위해서는 `json_remove` 함수를 사용하면 된다.

```sql
select 
	id, 
	json_remove(json_data, '$.cart') as json_data 
from 
	`user`;
```

| id | json_data |
| --- | --- |
| 1 | {<br/>"name": "솩솩1",<br/>"joinedDate": "2020-07-02"<br/>} |
| 2 | {<br/>"name": "솩솩2",<br/>"joinedDate": "2022-09-12"<br/>} |