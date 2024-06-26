# 1. ACID
- 트랜잭션의 특성
### ① Atomicity
- 원자성
- 트랜잭션에 포함된 데이터의 변경은 모두 반영되거나 모두 반영되지 않아야 함
- all or nothing
- 수행 중인 트랜잭션에서 데이터를 일부만 변경하고 나머지는 수행하지 않은채 커밋할 수 없음
- 따라서, 트랜잭션은 <u>데이터 변경을 위한 최소 단위</u>가 됨
### ② Consistency
- 일관성
- 트랜잭션에 의해 데이터 간의 일관성이 어긋나서는 안됨
- 혹은 사용자가 지정한 제약 조건들이 잘 유지되어야 함
- ex. 고객 개인의 데이터는 변경되었는데, 고객 전체 통계 데이터는 변경되지 않으면 일관성이 깨진 것
### ③ Isolation
- 고립성
- 트랜잭션은 고립되고 독립되어 있음
- 즉, 트랜잭션을 단독으로 실행하든 다른 트랜잭션과 동시에 실행하는 결과는 같아야 함
- DBMS들은 isolation level를 설정
	- READ UNCOMMITED
    - READ COMMITED
    - REPEATABLE READ
    - SERIALIZABLE 
### ④ Durability
- 지속성
- 트랜잭션은 장애가 발생하더라도 데이터는 반드시 복구되어야 함
# 2. REDO log
- 지속성을 구현하면서도 성능을 지키기 위한 구현 방법
	- 커밋한 데이터를 즉시 디스크에 기록하면 지속성은 지킬 수 있으나 성능이 감소함
    - REDO log를 통해 데이터를 한꺼번에 기록해서 I/O 횟수를 줄이고, 시퀀셜 액세스를 통한 I/O 시간도 줄여 성능을 향상시킴
- 로그 처리 과정들은 Latch에 의해 보호됨 
	- redo copy 혹은 redo allocation 등으로 표시되는 Latch
## 복구
- 특정 시점까지의 정보와 변경 정보가 있다면 최신 상태까지 복구 가능
- 오라클은 `SCN` 으로 시점을 관리
- 따라서 특정 SCN까지의 데이터베이스 정보와 REDO log가 있다면 복구가 가능 
	- roll-forward : 데이터베이스와 REDO로 최신 상태로 복구 하는 것
    - rollback : 데이터베이스와 UNDO로 변경을 되돌리는 것
## 구조
- 캐시에서 데이터 변경이 일어나면, REDO로그가 생성됨
- REDO 로그는 가장 먼저 공유 메모리에 있는 <u>REDO 로그 버퍼</u>에 기록됨
- **LGWR**에 의해 REDO로그가 REDO 로그 파일로 기록됨
	- 주로 log file sync 대기 이벤트가 LGWR이 로그를 기록하는 것을 기다리는 것 
    - 단축하고 싶다면, 커밋 횟수를 줄이거나 write 캐시를 가진 스토리지를 사용하는 방법 등이 있음 
- REDO 로그 파일은 개수가 한정되어 있어, <u>아카이블 REDO 로그 파일</u>을 통해 REDO 로그를 보관 
	- 단, 백업 SCN 이전에 만들어진 아카이브 REDO 로그 파일은 필요하지 않음 
### REDO 로그 파일
- 로그 파일은 매우 중요한 파일이므로 **다중화** 필요
- 다중화 시, 그룹이 늘어나는 것이 아니라 멤버가 늘어나는 것이므로 sync에 유의해야 함 
- 로그 파일은 개수가 한정되어 있으므로 1->2->3->1->2->3.. 식으로 순환하며 저장됨 
# 3. UNDO log
- 이전의 데이터로 되돌리기 위한 데이터 
- 데이터가 변경되면 UNDO 정보가 생성됨
- UNDO가 변경되는 정보 또한 REDO에 기록 
## 보관 장소
- 세그먼트에 보관
	- 기본적으로 트랜잭션과 UNDO 세그먼트는 일대일 대응
    - 필요에 따라 세그먼트의 수를 늘림 
- 즉, 테이블스페이스들 중 어딘가에 보관 -> <u>UNDO 테이블스페이스</u>
## 구조
- 링 버퍼 : 일정 시간이 지나면 데이터가 덮어쓰이는 버퍼
- 하지만 커밋하지 않은 데이터가 덮어써지지는 않음 
	- 커밋한 이후에도 일정 시간 유지하고 싶다면 `undo_retention` 파라미터 사용 
- 덮어쓰지 못하고 UNDO 세그먼트가 가득차면, 익스텐트를 추가해서 세그먼트 크기를 늘림
# 4. 동작 과정
## ① 롤백
- 롤백을 수행하면 UNDO에 보관된 정보를 사용해서 데이터를 원래의 값으로 되돌림
- 서버 프로세스가 비정상적으로 종료된다면,
	- PMON : 종료된 서버 프로세스를 정리
    - SMON : 데이터를 트랜잭션 시작 전의 상태로 되돌림 
## ② 읽기 일관성
- 트랜잭션 시작 시점의 데이터를 보여주기 위해 UNDO 사용
- 즉, UNDO를 사용해서 과거의 데이터를 메모리 위에 재현 
## ③ 커밋되지 않은 데이터를 읽을 때
- 오라클은 기본적으로 **READ COMMITTED**
- 따라서 커밋되지 않은 다른 세션의 데이터를 읽는다면, 변경되기 이전의 데이터를 보여줌 
- 필요할 때는 UNDO를 사용해 과거의 데이터를 메모리 위에 재현하는 방식으로 구현 
## ④ ORA-1555
- 대부분 시간이 오래 걸리는 검색을 수행할 때, UNDO의 정보가 덮어써져서 발생하는 에러
- 앞서 말한 `undo_retention`을 사용해서 UNDO 보존 기간의 하한값을 지정 가능
	- 단, UNDO 테이블스페이스의 자동 확장 기능이 ON일 때만 유효
 	- 필요하다면 MAXSIZE 값도 설정 
## ⑤ 체크포인트
> 메모리의 데이터를 디스크와 동기화 하는 작업
즉, DBWR이 데이터를 메모리에서 디스크로 기록한 시점 

- 너무 오래된 데이터를 기점으로 롤 포워드하려면 많은 시간이 소요
- 체크포인트를 통해 데이터를 디스크에 정기적으로 기록해서 롤 포워드 시간을 감소
- 단, 너무 빈번하게 기록한다면 병목 현상으로 인한 성능 감소 발생
## ⑥ 인스턴스 복구
= 충돌 복구 crash recovery 
> 오라클이 기동할 때 자동으로 수행하는 복구  

- 데이터 파일에 REDO로그를 적용하고 데이터를 최신 상태로 갱신
- 커밋하지 않은 트랜잭션은 UNDO 정보를 사용해서 롤백 처리 수행
	- 커밋 정보가 REDO에 기록되지 않았을 것이므로, 커밋하지 않은 트랜잭션 구분 가능 
- 좀 더 빠르게 복구하고 싶다면 체크포인트가 적절한 빈도로 수행되도록 설정 
- ABORT에 의한 정지 후 다시 기동할 때도 인스턴스 복구가 수행됨 
# 5. 멀티테넌트 아키텍처
- Multi-Tenant Architecture, MTA
- 여러 개의 데이터베이스를 관리하면서도 시스템의 독립성을 확보하고, 운영 비용을 절감하기 위한 시스템 아키텍쳐 
- 업무용 데이터베이스와 관리 데이터베이스가 존재
	- PDB(Pluggable Database) : 업무용 데이터베이스, 데이터베이스 전체에 공유하는 오브젝트와 메타데이터를 관리
	- CDB(Container Database) : 관리 데이터베이스 , 애플리케이션에서 접속하는 데이터베이스 
- 백업/복구, 업그레이드, 패치 등의 기본적 관리를 CDB를 통해 내부적으로 할 수 있으므로 운영 비용이 절감됨 
- 프로세스나 메모리와 같은 자원은 공유하지만, 사용자 데이터는 개별로 저장
	-> 자원을 효율적으로 활용하고, 데이터의 독립성을 유지할 수 있음 