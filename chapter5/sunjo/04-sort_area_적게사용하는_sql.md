<aside>
💡 메모리 내에서 처리를 완료할 수 있도록 노력
sort area의 크기를 늘리는 방법도 있지만 sort area를 적게 사용할 방법을 찾자
</aside>

### 소트 데이터 줄이기

---

1번

```sql
select lpad(상품번호, 30) || lpad(상품명, 30) || lpad(고객ID, 10)
 || lpad(고객명, 20) || to_char(주문일시, 'yyyymmdd')
from 주문상품
where 주문일시 betwwen :start and :end
order by 상품번호

```

30+30 +10+20+17 바이트로 가공한 결과 집합을 sort area에 담는다.

2번

```sql
select lpad(상품번호, 30) || lpad(상품명, 30) || lpad(고객ID, 10)
|| lpad(상품명, 20) || to_char(주문일시, 'yyyymmdd')
from (
	select *
	from 주문상품
	where 주문읽시 between :start and :end
	order by 상품번호
)

```

- 가공하지 않은 상태로 정렬을 완료하고 나서 최종 출력할 때 가공한다.

즉 2번이 sort area를 훨씬 적게 사용한다.

```sql
select * 
from 예수금원장
order by 총예수금 desc
```

```sql
select 계좌번호 
from 예수금원장
order by 총예수금 desc
```

아래 쿼리가 계좌번호만 저장하기 때문에 sort area를 적게 사용한다.

### TopN 쿼리의 소트부하 경감원리

---

소트 연산을 생략할 수 없을 때 top n 쿼리

1000명중 가장 큰 학생 선발

1. 전교생을 운동장에 집합
2. 10명을 불러서 키순으로 세움
3. 나머지 990명을 들여보내면서 기존 10명의 학생과 비교
4. 10명 학생 키순을 재배치

⇒ top n 소트

위 방식처럼

```sql
select *
from (
	select rownum no, a.*
	from (
	  /* sql */
	) a
	where. rownum <= (:page * 10)
where no >= (:page-1)* 10 + 1
```

- table full scan으로 처리하게 된다면 sort order by 오퍼레이션이 나타난다.
- 그리고 sort order by stopkey 있다면 top n 소트 알고리즘이 작동하게 됩니다.

⇒ 이 알고리즘이 작동하면 소트 연산 횟수와 sort area 샤용량을 최소화해준다. (느낌상 소트 할일이 없어서 소트 연산횟수는 적고 저장량은 비교할 10개이기 때문에 sort area 사용량이 최소인것 같다.)

### TopN 쿼리가 아닐 때

---

```sql
select *
from (
	select rownum no, a.*
	from (
	  /* sql */
	) a
where no >= (:page-1)* 10 + 1
```

- rownum이 없어서 stopkey가 없어 top n 소트가 작동하지 않는다면 많은 양의 데이터를 읽고 정렬을 하게된다.
- 그렇게 되면 디스크를 사용해야한다.

### 분석함수에서 top N 소트

---

- 윈도우 함수 중 rank나 row_number 함수는 Max 함수보다 소트 부하가 적다.
- ⇒ top N 소트 알고리즘이 작동하기 때문이다.