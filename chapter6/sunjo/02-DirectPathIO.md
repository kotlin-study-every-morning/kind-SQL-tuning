<aside>
💡 direct path io에 관해 알아보자
</aside>

### Direct Path IO

---

- 자주 읽는 블록에 대한 반복적인 IO는 버퍼캐시를 경유한다.
- 대량의 데이터, 배치 프로그램은 대량 데이터를 처리하기 때문에 버퍼캐시를 경유하지 않고 Direct Path IO기능을 사용할 수 있다.

**대량 데이터는 왜 Direct Path IO를 사용해야하냐?**

- 대량데이터를 버퍼캐시에서 찾을 가능성이 없기 때문이다.
- 대량 블록을 디스크로부터 버퍼캐시에 적재하고서 읽어야 하는 부담도 크다.

**Direct Path IO가 작동하는 경우**

1. 병렬 쿼리로 Full scan 수행할 때 ⇒ parallel, parallel_index 힌트 사용
2. 병렬 DML 수행할 때
3. Direct Path Insert를 수행할 때
4. temp 세그먼트 블록들을 읽고 쓸 때
5. direct 옵션을 지정하고 export를 수행할 때
6. nocache 옵션을 지정한 LOB 컬럼을 읽을 때

이중에 1, 2, 3이 활용도가 높다.

### Direct Path Insert

---

**일반 insert가 느린 이유**

1. 데이터를 입력할 수 있는 블록을 freelist에서 찾는다. (freeList란 블록중 데이터 입력이 가능한 블록을 목록으로 관리하는 것)
2. freelist에서 할당받은 블록을 버퍼캐시에서 찾는다.
3. 버퍼캐시에 없으면 데이터파일에서 읽어 버퍼캐시에 적재한다.
4. insert 내용을 undo 세그먼트에 기록한다.
5. insert 내용을 redo 로그에 기록한다.

Direct path insert를 사용하면 대량데이터를 빠르게 입력 가능하다.

- insert … select 문에 append 힌트 사용
- parallel 힌트를 이용해 병렬 모드로 insert
- direct 옵션을 지정하고 sql loader로 데이터 적재
- CTAS(create table .. as select) 수행

direct path insert가 빠른 이유

1. freelist를 참조하지 않고 데이터를 순차적으로 입력한다.
2. 블록을 버퍼 캐시에서 탐색하지 않는다.
3. 버퍼캐시에 적재하지 않고 데이터 파일에 직접 기록한다.
4. undo로깅을 하지 않는다.
5. redo 로깅을 안하게 할 수 있다.(alter table t NOLOGGING) - 일반 insert 문은 로깅하지 않게 하는 방법이 없음
- Array processing도 direct path insert 방식을 처리할 수 있음 ⇒ append_values 힌트

Direct Path insert 주의점

1. exclusive 모드 TM Lock이 걸린다. → 커밋하기 전까지 다른 트랜잭션은 해당 테이블에 DML을 수행하지 못한다.
2. freelist조회 하지 않고 HWM 바깥 영역에 입력하므로 테이블에 여유공간이 있어도 재활용하지 않는다.
    1. 즉 여유공간이 생겨도 insert하는 데이블은 사이즈가 줄지 않고 계속 늘어간다.

### 병렬 DML

---

update, delete는 기본적으로 direct path write가 불가능하다. ⇒ 유일한 방법은 병렬 DML

```java
alter session enable parallel dml;
```

1. ㅎ활성화 시켜줌
2. 각 DML에 힌트를 사용 - /*+ full(c) parallel(c 4) */

**만약 병렬 DML 활성화를 시켜주지 않는다면?**

- 대상 레코드를 찾는 작업은 병렬로 진행하지만 추가 / 변경 / 삭제는 QC가 혼자 담당하므로 병목이 생긴다.

```java
qc란?
Query Coordinator의 줄임말
SQL을 병렬로 실행하면 병렬도로 지정한 만큼 또는 두배로 병렬 프로세스를 띄워 동시에 작업을 진행하는데 이때 
최초 DB 접속해서 SQL 수행한 프로세스를 QC라고 부른다.
```

**병렬 DML이 잘 작동하는지 확인**
DML 작업을 각 병렬 프로세스가 처리하는지, 아니면 QC가 처리하는지 실행계획에서 보면 된다.