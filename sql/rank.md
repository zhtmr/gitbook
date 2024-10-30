# 순위집계

## RANK

> \[!NOTE]
>
> * 동일한 순위는 같은 등수로 표시
> * 이후 등수는 동일하게 표시한 등수의 갯수만큼 건너뛴 등수로 표시

```sql
SELECT RANK() OVER(PARTITION BY [그룹할 컬럼들] ORDER BY [순위를 매길때 사용할 컬럼들]) FROM [테이블명]
```

```sql
SELECT NAME
      , SCORE
      , RANK() OVER(PARTITION BY department ORDER BY score DESC)
FROM score 
```

ex) ... 3등(85점), 3등(85점), 3등(85점), <mark style="color:orange;">6등(80점)</mark>...

## DENSE\_RANK

> \[!NOTE]
>
> * 동일한 순위는 같은 등수로 표시
> * 이후 등수 차례로 표시

```sql
SELECT DENSE_RANK() OVER(PARTITION BY [그룹할 컬럼들] ORDER BY [순위를 매길때 사용할 컬럼들]) FROM [테이블명]
```

ex) ... 3등(85점), 3등(85점), 3등(85점), <mark style="color:orange;">4등(80점)</mark>...

## ROW\_NUMBER

> \[!NOTE]
>
> * 동일한 순위도 고유한 순위 부여

```sql
SELECT ROW_NUMBER() OVER(PARTITION BY [그룹할 컬럼들] ORDER BY [순위를 매길때 사용할 컬럼들]) FROM [테이블명]
```

ex) ... <mark style="color:green;">3등(85점), 4등(85점), 5등(85점)</mark>, <mark style="color:orange;">6등(80점)</mark>...





