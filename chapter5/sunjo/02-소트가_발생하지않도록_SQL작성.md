<aside>
💡 union, minus, distinct는 소트 연산을 발생시키므로 꼭 필요할 때만 사용

</aside>

### Union vs Union All

---

union all은 중복을 확인하지 않고 두 집합을 단순히 결합하므로 소트 작업을 수행하지 않는다.

- 결과 집합이 예상과 달라질 수 있음(union all은 합집합 - 상호 배타적일 떄 사용)

### exists 활용

---

dinstinct는 중복을 제거하기 때문에 소트 연산 발생

- 인덱스가 [상품 번호 + 계약일자]일 떄 상품 수는 적고 상품별 계약 건수가 많으면 비효율적
- exists는 존재 여부만 확인하면 되기 때문에 조건절을 만족하는 데이터를 모두 읽지 않는다.

### 조인 방식 변경

---

```sql
select * 
from where 지점ID
order by 계약일시
```

인덱스가 [지점ID + 계약일시]일 떄 소트 연산 생략 가능하지만 해시조인이면 sort order by가 나타난다.