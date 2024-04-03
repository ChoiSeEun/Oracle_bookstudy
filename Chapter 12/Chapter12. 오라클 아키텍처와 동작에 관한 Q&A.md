# 1. 성능 확인
## OLTP
- 오라클의 AWR
- 스태츠팩
- OS의 I/O 정보에 표시되는 I/O 1회의 시간
- `v$session_wait` 에서 확이할 수 있는 I/O 관련 대기 이벤트의 수
- (유닉스) vmstat에서의 b 컬럼
	- 디스크 I/O를 대기하는 프로세스 수
- 서버 프로세스 대기 이벤트 수 
	- DBWR/ LGWR의 I/O 지연을 파악 
## Batch
- 오라클의 AWR
- 스태츠팩
- OS의 I/O 정보에 표시되는 1회의 I/O에 걸린 시간 
## 네트워크 지연 
- `V$session` 을 이용해서 작업 요청이 도착하지 않았다는 사실 직접 확인
- 패킷 캡처 도구를 이용해서 패킷의 송수신을 직접 확인
## OS 문제
- (유닉스) vmstat, sar
- (윈도우) 성능 모니터 
## 과부하
- OS 정보
- I/O 정보
- `v$session_wait` 등에서 대기열이 생기지 않았는지
- 가능한 부하를 줄인 후, 각 처리에 걸리는 시간 변화 조사
## 데이터베이스 상태
- 작업의 처리가 진행 중인지 **처리량**을 확인
- 대기가 없는지 **응답 시간**을 확인
- `v$session_wait` 를 이용해서 대기가 많은지 확인
- 배치 시스템인 경우
	- 처리량이나 대기의 양은 크게 도움되지 않음
    - 각각의 배치가 처리 종료 예정 시간이 지났음에도 끝나지 않으면, 각 해당 시간을 측정
### 결과 해석
- 처리량이 적고 대기가 많다면,
	- 해당 대기가 지연의 원인일 가능성이 높음
- 처리량이 적고 대기가 거의 없다면,
	- 오라클까지 요청이 도착하지 않았을 가능성이 높음
    - 애플리케이션이나 네트워크 지연이 원인일 수 있음
- 처리량에 비례해서 대기가 많다면,
	- 단순히 처리량이 많은 상황일 수 있음 
## 모니터링
- 지연이 발생했을 때 나타나는 대기 이벤트의 양 확인
- `v$session_wait` 나 `v$session` 등에서 대기 중인 세션의 양을 정기적으로 확인
	- 임계치를 넘었을 때 통보할 수 있는 체계를 미리 만들어 두는 것이 좋음 
- 스태츠팩이나 AWR을 이용한 분석
	- `v$session` , `v$session_wait` , `v$sysstat`
    - `v$session` 과 `v$session_wait` 는 그 당시의 상태를 알 수 있고,
    - `v$sysstat` 은 처리량 등의 상세한 내용을 알 수 있음
# 2. 대기 이벤트
- `db file sequential read` : 싱글 블록 I/O
- `db file scattered read` : 멀티 블록 I/O
- `read by other sesion` : 다른 세션이 읽어오는 중이므로 대기
- `write complete waits` : 기록하고 있으므로 대기
- `buffer busy waits` : 블록 사용 중
- `free buffer waits` : 빈 버퍼가 부족
- `log file sync` : REDO  로그 기록을 대기
- `direct path read/write` : I/O가 버퍼 캐시를 경유하지 않고 PGA 영역에 바로 발생
- `direct path read/write temp` : 버퍼 캐시를 경유하지 않고 PGA 영역에 액세스하는 I/O 중 임시 테이블스페이스에 액세스할 때 발생
- `log file sync` : LGWR이 공유 메모리 상의 REDO 로그를 디스크 상의 REDO 로그 파일로 기록할 때 발생
- `end:TX` : 테이블의 로우 1개의 로우 lock을 표시
