```python
create table A(
  c1 data_type,
  c2 data_type,
  c3 data_type,
  c4 data_type,
  c5 data_type,
  c6 data_type
  );
```  
위 결과로 만들어지는 테이블이 있다 가정...

SQL은 기본적으로 order by는 asc default.(반대는 desc)

```python
select c1, c2 from A order by c1, c2;
```
위 결과는 c1, c2를 가져오는데 c1, c2 를 기준으로 오름차순 정렬하여 출력.
단, 정렬시 c1이 c2 보다 우선순위 높음.

```python
select c1, c2 from A order by 1, 2;
```
위 결과는 위와 같음. 열 위치로 정렬할 수 있음.
위와 같이 열 위치로 정렬하는 방법은 select 목록에 포함되어 있지 않은 열도
정렬 기준으로 선택이 가능해

```python
select c1, c2 from A order by 3;
```
위 결과는 c1, c2를 가져오는데 c3 열을 기준으로 오름차순 정렬해서 출력이 되겠지?





