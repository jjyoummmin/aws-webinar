DAY1
====
aws 컴포넌트들 중 가장 기본이 되는 것들에 대해 학습하는 내용이었다.

IAM
---
aws 전체의 권한 통제 시스템

일단 태초에 AWS account(계정)과 root user가 있다. 루트는 꼭 필요할 때만 사용하고 항상 IAM User 생성해서 사용할 것.

![iam](./image/iam.png)

### 인증 🎫 : 누구인가? 
- user (group에 속할 수 있음) : 실사용자 기준. 상시적 
- group
- role : 외부 사용자, aws 서비스, 프로그램에 부여. 임시(기본 1시간)

`user` -> (name, pw, api나 cli 로그인을 위한 2 access key)  
`role` -> does not have any credentials (password or access keys) associated with it. 

### 인가 👌 : 인증된 사용자/리소스가 요청한 서비스에 대한 권한 갖고 있는가?
- policy 기준으로

![iam_policy](./image/iam_policy.png)

**identity** 기반 vs **resource** 기반 policy로 구분할 수 있음
(예를 들자면 계정 A의 유저 a / 계정 A의 s3 리소스)

요청 성공 조건 -> IAM 보안 주체의 적법한 서명값이 포함되어 있고 (인증) AND 정책 (policy)에 의해 해당 요청이 정확하게 인가되어야 한다.

#### 최소 권한 원칙을 위한 툴 추천
- Access Advisor : 어떤 권한을 언제 마지막으로 사용했는지 확인
- Access Analyzer : aws account내의 어느 리소스에 누가 접근 가능한지 분석
- Credential Report 
- IAM Policy Simulator


Log
---
#### AWS CloudTrail
모든 aws API 요청들에 대해 신뢰성 있는 기록을 수행. (누가 언제 어디서 무엇을 결과 형태로)

90일간의 이력 조회는 무료
s3나 cloudwatch에 전달하는 서비스는 무료

#### AWS CloudWatch
aws 리소스, 앱에 대한 모니터링


VPC
---

Computing
---------

Storage
-------

비용 절감
-------
- 리소스에 태그를 추가해서 관리하기
- 데이터 트랜스퍼 비용 줄이기
1) 기본 : in 무료, out 유료
2) 같은 리전 내 다른 az : in/out 유료. 가급적이면 같은 az 내 인스턴스 끼리 데이터 주고받도록 구성하기
- 컴퓨팅 인스턴스. 딱 맞는 spec 사용하기  
추천하는 툴
    - cost explorer
    - compute optimizer
    - trusted advisor
- 빌링 리포트 생성하기


[실습](hands_on_lab.md)