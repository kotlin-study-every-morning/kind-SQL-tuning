### Sort Area를 적게 사용하도록 S11iQL 작성

**소트 데이터 줄이기**

```sql
select lpad(상품번호, 30) || lpad(상품명, 30) || lpad(고객ID, 10)
from 주문상품
where 주문일시 betwwen :start and :end
order by 상품번호
```

```sql
select lpad(상품번호, 30) || lpad(상품명, 30) || lpad(고객ID, 10)
from (
	select 상품번호, 상품명
	from 주문상품
	where 주문읽시 between :start and :end
	order by 상품번호
)
```

- 두 번 째 SQL이 Sort Area를 더 적게 사용한다
- 첫 번 째는 레코드당 바이트로 가공한 결과집합을 Sort Area에 담지만 두 번 째 SQL은 가공하지 않은 상태로 정렬을 완료하고 나서 최종 출력할 때 가공한다

```sql
select *
from 예수금원장
order by 총예수금 desc;
```

```sql
select 계좌번호, 총예수금
from 예수금원장
order by 총예수금 desc;
```

- 2번이 더 적게 사용한다
- 1번은 모든 컬럼을 sort Area에 저장하는 반면, 2번은 계좌번호와 총예수금만 저장하기 때문

**분석함수에서의 Top N 소트**

- 윈도우 함수 중 rank나 row_number 함수는 Max 함수보다 소트 부하가 적다
- Top N 소트 알고리즘이 작동하기 때문
