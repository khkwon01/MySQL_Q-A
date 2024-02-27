## 1. MySQL & MySQL HeatWave(Cloud) 
### 1) shape (리소스, Cloud only)
- HeatWave에서 제공하는 MySQL shape
  - https://docs.oracle.com/en-us/iaas/mysql-database/doc/supported-shapes.html

### 2) replicaton (channel) 구성
- channel 구성시 binary log expire 설정 (* Cloud only)
  - MDS는 기본적으로 binary log expire 1시간 설정되어 있어, MDS를 소스로 설정시 충분한 시간 확보 필요
    * binlog_expire_logs_seconds 

- upgrade시 rollback용으로 하위버전 replication 구성 (MySQL 5.7 --> MySQL 8.0 (MDS) --> MySQL 5.7)
  - replication 오류 사항
    <img width="1143" alt="image" src="https://github.com/khkwon01/MySQL_Q-A/assets/8789421/bbf44d20-c475-4cc8-bf82-16af38dc229c">
  - 대응 방안    
    기본적으로 상위 버전(8.0)에서 하위 버전(5.7)로 replication은 권장하지 않습니다. 왜냐하면 8.0에 새로운 기능과 character set등이 포함 되어 있어
    관련 내용을 binlog 기록후 5.7에서 사용시 replication에 오류가 발생
- MySQL MDS Read replica 연결 분산
  - read load balancer 사용시 분산방법     
    read load balancer를 사용하여 read replica 연결시에는 5-Tuple hash 방식으로 client ip, port, dest ip, port, protocal 5가지를
    가지고 read 부하를 replica에 분산     
    ![image](https://github.com/khkwon01/MySQL_Q-A/assets/8789421/a6a7bca7-d5fb-4d13-b202-988c157075c6)
  - read load balancer 설정
    mysql mds의 read load balancer에는 위에 분산 알고리즘을 제외하고는 다른 분산 알고리즘은 지원하지 않음
  - read replica 모니터링 (*Software 방식도 동일)
    ```
    show replica status\G
    select * from performance_schema.replication_connection_status
    select * from performance_schema.replication_applier_status_by_coordinator
    select * from performance_schema.replication_applier_status_by_worker
    select * from performance_schmea.error_log
    ```
- 줄지 않고 늘어나는 연동(replication lag) 시간
  - source와 target간 데이터 export와 load 시간을 줄여서 변동분 반영 시간 최소화   
    데이터 export와 load시 mysql-shell에 multi-thread, chunk 사이즈를 조정, consistent option false 등
  - source(master) binlog file 및 position를 확인하여 mysqlbinlog를 사용하여 중단 시점에 SQL 확인    
    또는 replica에서 show processlist를 통해 문제가 되는 쿼리와 테이블 확인   
  - 문제가 되는 쿼리 및 테이블가 확인될 경우 source와 target간 실행 계획이 동일한지 확인    
    또는 source 테이블에 FK등이 있는지 체크

### 3) Maintenance 
- MySQL upgrade시 알림 설정 (*Cloud only)
  현재 시스템 알림이 없는데, 아래 설정을 통해 maintenance 시작과 종료시 알람을 직접 받을 수가 있음
  - 설정 메뉴
    - Observability & Management 항목 --> Events Service --> Rules 에서 upgrade rule 설정
  - 설정 방법
    - Events Service 설정     
      - 아래에 항목을 설정하여 rules 설정
        ![image](https://github.com/khkwon01/MySQL_Q-A/assets/8789421/9d2749f4-a1fc-49d5-a7de-bc5226cfe391)      
      - 테스트로 events rule 만들어 놓은 항목
        ![image](https://github.com/khkwon01/MySQL_Q-A/assets/8789421/22e6b0fe-9985-4b1a-bffe-12f6a2c943c4)
      - 아래와 같이 upgrade 시작과 끝에 해당 하는 evnet 설정하고 notification를 e-mail 선택    
        (upgrade 시작 : MySQL - Upgrade DB System Begin, upgrade 종료 :  Upgrade DB System End)    
        ![image](https://github.com/khkwon01/MySQL_Q-A/assets/8789421/74a5e6d2-396d-4632-bcd2-2f3f4cf922ed)
    - Event 설정후 MySQL 시스템 upgrade시 알림 내용    
      위에 upgrade 알림 설정후 MySQL 시스템에 설정된 Maintenance 시점에 upgrade가 발생할 경우 아래와 같이 알림이 옴   
      (상황에 따라 다르겠지만, 메일 알림 기준으로 maintenance 시간은 대략 20분 내외 소요됨)  
      ![image](https://github.com/khkwon01/MySQL_Q-A/assets/8789421/51a270b6-ad96-47eb-8e39-cefb77167473)
- (참고) 알람 수신을 위한 notification topic 생성 방법
  - Notification 서비스를 하나 만들고 구독에 Email 구독 추가하시면 됩니다.
  - Subscription Topic  리스트박스에 위에서 만든 Notification Topic명을 선택해주고,    
    구독(Subscriptions) 화면에 고객분 이메일을 추가하시면 됩니다. 이 때 해당 email 로 Confirm 메일이 발송됩니다.    
    해당 메일로 들어가셔서 링크를 클릭하셔서 confirm 하셔야 Ative 상태가 됩니다.
    ![image](https://github.com/khkwon01/MySQL_Q-A/assets/8789421/6c51a0a3-dcac-4656-810e-dd7667c83842)

### 4) Character set 또는 Collation
- source와 target간 table에 default character set 또는 collation이 틀려서 아래와 같은 오류 발생 가능
  - Illegal mix of collation (utf8mb4_unicode_ci,IMPLICIT) and (utf8mb4_0900_ai_ci,IMPLICIT) for operation    
    source와 target간 전체 DB와 테이블, 컬럼간 character set 또는 collation를 맞추어서 반영하는게 필요함
