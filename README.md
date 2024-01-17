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

