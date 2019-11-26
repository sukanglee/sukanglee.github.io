---
title: "postgresql tablespace 사용하기"
date: 2019-11-26 11:26:28 +0900
categories: postgresql
---


- 리눅스 커맨드 상에서 실제 디렉토리 만들기 & 권한 설정
```ubuntu
mkdir <tablespace_dir>
chown postgres <tablespace_dir>
chmod 744 <tablespace_dir>
```
- postgresql 커맨드 상에서 테이블스페이스 생성
```postgresql
CREATE TABLESPACE <tablespace_name> LOCATION '<tablespace_dir_absolutepath>'
```
- postgresql 커맨드 상에서 테이블 생성 시 테이블스페이스 사용 설정
```postgresql
CREATE TABLE <table_name>(
  <column_name1> <data_type1>, 
  <column_name2> <data_type2>,
              ... ,
  <column_nameN> <data_typeN>
)TABLESPACE <tablespace_name>;
```




