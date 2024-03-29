# 1. 오라클 기동 
## 용어 정리
- 컨트롤 파일 : 데이터베이스의 구성 정보가 적혀 있는 파일
- 인스턴스(instance) : 백그라운드 프로세스 + 공유 메모리 
- 스토리지(storge) : 일반적으로 데이터베이스라고 불리는 부분(데이터 파일, 컨트롤 파일 등이 보관)
## 기동 상태
- NOMOUNT : 인스턴스가 기동한 상태
- MOUNT : 컨트롤 파일을 읽은 상태 
- OPEN : 데이터 파일 점검까지 완료한 상태 
```
sql> startup
----
ORACLE instance started.

Database mounted.
Database opened.
```

### ① 기동 정지 상태에서 NOMOUNT로
```
sql> start nomount
----
ORACLE instance started.
```
- ORACLE_HOME과 ORACLE_SID 환경 변수를 토대로 초기화 파라미터 파일을 찾아서 읽어옴
	- PFILE : 텍스트 형식으로 저장된 파라미터 파일(텍스트 에디터를 통해 변경, 수정 시 재기동 후 적용됨)
    - SPFILE : 바이너리 형식의 파라미터 파일 (SQL문을 통해 변경, 적용 범위 지정 가능)
- 파라미터를 토대로 공유 메모리를 확보하고 백그라운드 프로세스를 생성 
### ② NOMOUNT에서 MOUNT로
```
sql> alter database mount;
----
Database altered.
```
- 컨트롤 파일을 읽어옴
	- 초기화 파라미터에 기술된 컨트롤 파일의 경로를 사용 
- REDO 로그 파일이나 데이터 파일 등 데이터베이스를 구성하는 각종 파일의 위치를 파악 
- 위치 파악만 하기 때문에 해당 시점에 실제 파일이 없더라도 에러가 발생하지 않음 
### ③ MOUNT에서 OPEN으로
```
sql> alter database open;
----
Database altered.
```
- 데이터 파일을 읽어서 점검
- 백그라운드 프로세스를 기동
	- 서버 프로세스는 클라이언트 요청이 올 때 응답
- 데이터 파일을 읽고 기록할 수 있는 상태가 되어 SQL을 실행할 수 있게 됨 
# 2. 오라클 정지
- 기동 작업의 역순으로 데이터베이스를 닫은 후 인스턴스 종료
- 인스턴스 종료
	- 공유 메모리 반환
    - 백그라운드 프로세스 정지
    - 버퍼 캐시에 분산된 데이터 정리 : 기록되지 않은 변경 데이터 기록 
```
sql> shutdown
----
Database closed.
Database dismounted.
ORACLE instance shut down.
```
## shutdown 옵션
- (없음) : 커넥션의 종료를 기다리며, 변경된 데이터를 파일에 기록
- transactional : 트랜잭션이 종료되면 커넥션을 끊고, 변경된 데이터를 파일에 기록
- immediate : 커넥션의 종료를 기다리지 않지만, 변경된 데이터는 파일에 기록
- abort : 커넥션의 종료를 기다리지 않고, 변경된 데이터도 파일에 기록하지 않음 
	- instance recovery 
    	- 다음 기동 시에 데이터를 자동으로 복구
        - REDO 로그 파일의 데이터를 사용해서 복구