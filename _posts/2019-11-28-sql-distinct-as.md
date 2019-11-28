- DISTINCT
```python
select distinct c1 from table_name; 
select distinct c1, c2 from table_name;
```
첫번째 쿼리문은 첫번째 컬럼인 c1에 중복된 것을 제거한 결과가 나올거고,
두번째 쿼리문은 c1의 값과 c2의 값을 한 쌍으로 보고 (c1,c2의 조합) 중복을 제거한 결과가 나올거야

- AS
```python
select id as user_id, position, dept_nm from emp;
select id "user_id", position, dept_nm from emp;
```
위 두 쿼리문은 동일한 결과가 나올거야
