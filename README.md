# MySQL_Q-A

## 1. MySQL HeatWave 
### 1) replicaton 구성
- MySQL MDS 8.0에서 MySQL 5.7로 replication 구성시 (upgrade시 rollback용)
  - replication 오류 사항
    <img width="1143" alt="image" src="https://github.com/khkwon01/MySQL_Q-A/assets/8789421/bbf44d20-c475-4cc8-bf82-16af38dc229c">
  - 대응 방안
    기본적으로 상위 버전(8.0)에서 하위 버전(5.7)로 replication은 권장하지 않습니다. 왜냐하면 8.0에 새로운 기능과 character set등이 포함 되어 있어
    관련 내용을 binlog 기록후 5.7에서 사용시 replication에 오류가 발생

### 2) Maintenance 
- MySQL upgrade시 알림 설정   
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
      (메일 알림 기준으로 maintenance 시간은 대략 15분 내외 소요됨)  
      ![image](https://github.com/khkwon01/MySQL_Q-A/assets/8789421/51a270b6-ad96-47eb-8e39-cefb77167473)
