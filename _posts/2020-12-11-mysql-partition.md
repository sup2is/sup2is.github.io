---
layout: post
title: "MySQL에서 파티션 사용하기"
tags: [Database, MySQL, MySQL Partition]
date: 2020-12-11
comments: true
---


<br>

# OverView

이번시간에는 MySQL 파티션에 대해서 알아보고 간단한 예제를 작성해보도록 하겠다.

# MySQL 파티션

파티션에 대해서 간단하게 알아보면 파티션은 다음과 같다.

- MySQL 서버 입장에서 데이터를 별도의 테이블로 분리해서 저장하지만 사용자 입장에서는 여전히 하나의 테이블로 읽가와 쓰기를 할 수 있게 해주는 솔루션
- 일반적으로 DBMS의 파티션은 하나의 서버에서 테이블을 분산하는 것
- MySQL은 5.1 이상에서 사용 가능함
- 해시, 리스트, 키, 레인지 총 4가지 파티션 방법을 제공하고 서브 파티셔닝 기능까지 제공할 수 있음

파티션을 사용하는 이유는 **하나의 테이블이 너무 커진 상태에서 인덱스의 크기가 물리적인 메모리보다 훨씬 크거나 데이터 특성상 주기적인 삭제 작업이 필요한 경우**에 파티션이 필요한 대표적인 예가 된다.

인덱스의 크기가 너무 크다면 **파티션으로 데이터와 인덱스를 조각화해서 물리적 메로리를 효율적으로 사용**할 수 있게 만들 수 있다.

이런 대표적인 예는 데이터의 성향에 따라 결정된 해시, 리스트, 키, 레인지 총 4가지 파티션에 따라 조금씩 다르겠지만 다음과 같은 예가 있다.

- 로그 테이블처럼 날짜와 같은 데이터로 테이블을 구분할 수 있을 때 -> 레인지 파티션 
- 데이터들을 균등하게 분배해야할 때 -> 해시 파티션

# MySQL 파티션 종류

## 레인지 파티션

레인지 파티션은 파티션 키의 연속된 범위로 파티션을 정의하는 방법이고 일반적으로 가장 많이 사용하는 장식이다.

레인지 파티션은 다음과 같은 성격의 테이블에서 사용하기 좋다.

- 날짜를 기반으로 데이터가 누적되는 데이터
- 연도, 월, 일 단위로 분석하고 삭제해야하는 데이터
- 범위를 기반으로 데이터를 여러 파티션에 균등하게 나눌 수 있는 데이터
- 파티션 키 위주로 검색이 자주 실행될 때



### 레인지 파티션 생성

레인지 파티션을 사용하는 logs 테이블을 생성해보자.

```sql
CREATE TABLE logs (
    log_idx INT NOT NULL,
    /*기타 컬럼*/
	reg_date DATETIME NOT NULL,
	PRIMARY KEY(log_idx, reg_date)
)
PARTITION BY RANGE (YEAR (reg_date)) (
	PARTITION log_2018 VALUES LESS THAN (2019),
	PARTITION log_2019 VALUES LESS THAN (2020),
	PARTITION log_2020 VALUES LESS THAN (2021),
	PARTITION log_etc VALUES LESS THAN MAXVALUE
);
```

다음과같이 PARTITION BY RANGE 키워드로 파티션을 생성할 수 있고 실제 `YEAR()` 함수 또는 `TO_DAYS()` 함수로 파티션 키를 설정할 수 있다.

생성된 파티션을 직접확인하고싶다면 다음과 같은 쿼리를 실행해서 확인할 수 있다.



![1564389258F05B893DBB2CE941B4CB46_img_760](https://user-images.githubusercontent.com/30790184/102008245-8ca32400-3d72-11eb-9c6c-064cc6d9b5bb.jpg)



### 레인지 파티션의 추가 및 분리와 병합

MAXVALUE 파티션이 정의되지 않았을 경우 아래와 같이 단순하게 파티션을 추가할 수 있다.

```SQL
ALTER TABLE logs ADD PARTITION (PARTITION log_2021 VALUES LESS THAN (2022))
```

만약 MAXVALUE를 사용한 파티션이 있을 경우 아래와 같이 `REORGANIZE` 파티션을 분리할 수 있다.

```SQL
ALTER TABLE logs
	REORGANIZE PARTITION log_etc INTO (
	PARTITION log_2022 VALUES LESS THAN (2022),
	PARTITION log_etc VALUES LESS THAN MAXVALUE
);
```

기존 파티션들은 다음과 같이 병합할 수 있다.

```SQL
ALTER TABLE logs
	REORGANIZE PARTITION log_2022, log_etc INTO (
	PARTITION log_etc VALUES LESS THAN MAXVALUE
);
```

파티션을 삭제역시 ALTER문을 사용한다.

```sql
ALTER TABLE employees DROP PARTITION log_2021;
```


### 레인지 파티션의 주의사항

- 파티션의 키가되는 값이 NULL이라면 NULL은 어떤 값보다 작은 값으로 간주되기때문에 주의할 것
- 다음과 같은 파티션 키는 피하는것이 좋음
  - UNIX_TIMESTAMP()를 이용한 변환 식을 파티션 키로 사용
  - 날짜를 문자열로 포맷팅한 형태의 파티션키
  - YEAR()이나 TO_DATES()함수 이외의 함수가 사용된 파티션 키



## 리스트 파티션

리스트 파티션은 레인지 파티션과 매우 흡사한데 가장 큰 차이점은 **레인지 파티션은 파티션 키 값이 연속된 값의 범위로 구성되어 있지만 리스트 파티션은 파티션 키 값 하나하나를 리스트로 나열**해야한다.

리스트 파티션은 파티션 키 값이 코드값이나 카테고리와 같이 고정적이고 정렬 순서와 관계 없이 파티션을 생성해야할 때 주로 사용한다.

### 리스트 파티션 생성

리스트 파티션의 생성은 다음과 같이 할 수 있다.

```sql
CREATE TABLE books (
    book_id INT NOT NULL,
	category_id INT NOT NULL,
	PRIMARY KEY(book_id, category_id)
)
PARTITION BY LIST (category_id) (
	PARTITION sf VALUES IN (1, 2, 3),
	PARTITION thriller VALUES IN (4, 5, 6),
	PARTITION romance VALUES IN (7, 8, 9),
	PARTITION etc VALUES IN (10, NULL)
);
```



### 리스트 파티션 추가 및 분리와 병합

파티션 정의에서 VALUES IN을 사용하는것 이외에 리스트 파티션의 추가 및 분리, 병합은 레인지 파티션과 동일하다.



### 리스트 파티션  주의사항

- 레인지 파티션에서 사용했던 MAXVALUE를 사용할 수 없음
- 레인지 파티션과는 달리 NULL을 저장하는 파티션을 별도로 생성 가능
- MySQL 5.1은 정수타입 5.5는 문자열 타입도 가능



## 해시 파티션

해시 파티션은 MySQL에서 정의한 해시 함수에 의해 레코드가 저장될 파티션을 결정하는 방법이다. **MySQL에서 정의한 해시 함수는 파티션 표현식의 결과 값을 파티션 개수로 나눈 나머지로 저장될 파티션을 결정**하는 방식이다.

해시 파티션은 특정한 키 없이 데이터를 균등하게 나눠야할 때 사용한다.

### 해시 파티션 생성

해시 파티션은 다음과 같이 생성할 수 있다.

```sql
CREATE TABLE member (
    member_id INT NOT NULL,
    name VARCHAR(10) NOT NULL,
	PRIMARY KEY(member_id)
)
PARTITION BY HASH (member_id) 
PARTITIONS 4 (
	PARTITION member_1,
    PARTITION member_2,
    PARTITION member_3,
    PARTITION member_4
);
```



### 해시 파티션  추가 및 분리와 병합

- 해시파티션에는 특정 파티션을 삭제, 병합하는것은 불가능하고 테이블 전체적으로 파티션의 개수를 줄이는 것만 가능함
- 해시파티션에는 특정 파티션만 분리하는것은 불가능하고 테이블 전체적으로 파티션의 개수를 늘리는 것만 가능함



### 해시 파티션 주의사항

이 방식을 사용할때 가장 주의해야할 것은 **파티션수의 변화가 이뤄지면 MySQL에서 정의한 해시 함수에 의해 파티션에 들어가있는 레코드의 재배치**가 이루어지기 때문에 해시 파티션이 용도에 적합한 해결책인지 확인이 필요하다.



## 키 파티션

키 파티션은 해시 파티션과 사용법과 특성이 거의 같다. 키 파티션의 특징은 해시 파티션과는 다르게 파티션의 키타입이 MySQL의 MD5() 함수에 의해 파티셔닝이 되기 때문에 파티션 키가 반드시 정수 타입이 아니어도 된다는 특징이 있다.

### 키 파티션의 생성

```sql
CREATE TABLE member2 (
    member_id INT NOT NULL,
    name VARCHAR(10) NOT NULL,
	PRIMARY KEY(member_id)
)

PARTITION BY KEY () 
PARTITIONS 4 (
	PARTITION member_1,
    PARTITION member_2,
    PARTITION member_3,
    PARTITION member_4
);
```

기본적으로 PK가 있을 경우 자동으로 PK키가 파티션 키로 사용된다. 만약 PK가 없고 유니크키가 있다면 유니크 키를 파티션 키로 사용한다. 

`KEY()` 부분에 아무것도 명시하지 않는다면 PK가 다중 칼럼으로 이루어진 경우 모든 칼럼이 파티션 키의 대상이된다. `KEY()` 부분에 키를 명시적으로 설정할 수 있다.

### 키 파티션의 추가 및 분리와 병합

해시 파티션과 동일하다.

# MySQL 파티션의 제한사항

파티션은 다음과 같은 여러개의 제약사항을 가진다.

- MySQL 5.1 버전 이하에서는 숫자 값 (INTEGER 타입 칼럼 또는 INTEGER 타입을 반환하는 함수 및 표현식)에 의해서만 파티션이 가능함 5.5 이상부터는 숫자타입 뿐 아니라 문자열이나 날짜 타입 모두 사용할 수 있도록 개선됨
- 키 파티션은 해시 함수를 MYSQL이 직접 선택하기 때문에 칼럼 타입 제한이 없음
- 최대 1024개의 파티션을 가질 수 있음 서브파티션 포함
- 스토어드 루틴이나 UDF그리고 사용자 변수 등을 파티션 함수나 식에 사용할 수 없음
- 파티션 생성 이후 SQL 서버의 sql_mode 파라미터 변경은 추천하지 않음
- 파티션 테이블에서는 외래키 사용 불가
- 파티션 테이블은 전문 검색 인덱스 생성 불가
- 공간 확장 기능에서 제공되는 칼럼 타입은 파티션 테이블에서 사용 불가
- 임시 테이블은 파티션 기능 사용 불가
- MyISAM 파티션 테이블의 경우 키 캐시를 사용할 수 없음 5.5이상부터는 보안됨
- 파티션 키의 표현식은 일반적으로 칼럼 그 자체 또는 mysql 내장 함수를 사용할 수 있는데 일부만 사용 가능함
- 모든 파티션은 같은 구조의 인덱스만 가질 수  있음 별도로 파티션마다 추가 불가



# 파티션 프루닝

파티션이 여러개인 테이블에서 불필요한 파티션을 빼고 쿼리를 수행하기 위해 접근해야 할 것으로 판단되는 테이블만 골라내는 과정을 파티션 프루닝이라고한다. 



<br>

***

포스팅은 여기까지 하겠습니다. 퍼가실때는 출처를 반드시 남겨주세요!

<br>

**References**

- Real-MySQL - 이성욱 (위키북스)
