# 1. LOCK
- 데이터를 변경하는 작업 도중에는 다른 작업들은 접근할 수 없도록 해야 함
	- 변경되기 전 데이터를 조회하는 작업은 접근 가능 
- 즉, <u>다중 처리를 구현하기 위해 데이터를 보호한다</u>는 것이 본질 
- LOCK을 건 세션의 트랜젝션이 완료(=커밋 또는 롤백이 수행)될 때 해제됨 
- DML이 실행되는 경우, 자동으로 로우 lock을 걸어 문제 발생 방지 
## 종류
- `TX` : row와 관련된 lock
- `TM` : table에 거는 lock
## 모드
- `X` : 같은 대상에 대하여 다른 모드의 lock을 허용하지 않는 배타적 lock
- `RX(=RS)` : 같은 대상에 대하여 RX(=RS) lock을 허용하는 lock 
# 2. 대기
- Idle 대기: 처리할 것이 없어서 대기하는 상태
	- SQL*Net message from client
    - smon timer
    - pmon timer
    - rdbms ipc message
    - wakeup time manager
    - Queue Monitor Wait 
- Non-Idle 대기 : 이유가 있거나 이상 상태 등으로 대기하는 상태

## Non-Idle 대기
- SQL 에 걸린 시간에 포함되는 대기 
	- 즉, SQL 튜닝 시 신경써야 하는 부분 
- 가장 대표적인 None-Idle 대기에는 디스크 I/O 대기가 있음 
- Lock에 의한 대기는, Lock이 걸려 있는 대상에 다시 Lock을 걸려고 했을 때 발생 

## Deadlock
> 서로가 상대방이 보유하고 있는 lock을 기다리느라 영원히 작업 처리를 진행할 수 없는 상태

- 오라클에서 Deadlock이 발생한 경우, 한쪽의 처리가 자동으로 롤백 
	- alert 파일과 트레이스 파일에 정보가 표시됨
    - Deadlock 그래프를 통해 관련 정보 확인 가능 
# 3. Latch
- 다중 처리를 구현하기 위한 Lock
- 메모리나 데이터를 조작할 때 상호 배타적으로 처리하지 않아 데이터가 손상되는 것을 방지하기 위해 사용 
- `Lock`과의 차이점
	- 오라클 내부에서 자동으로 획득
    - SQL을 한 번 실행하기 위해서는 여러 Latch를 얻고 해제하는 것을 반복 
- CPU 스케줄링, OS 페이징 등의 현상으로 인해 Latch 경합이 발생할 수 있음 