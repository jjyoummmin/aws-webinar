DAY2
====
serverless, Devops 관련 내용 학습.

aws basic
---------
#### IAM  

IAM role -> 관리해야 하는 secret access key가 없기 때문에 application을 사용해서 접근하는 경우 이 방식을 사용하는 것을 권장.

aws cli를 위한 크레덴셜   

[구성 및 자격증명 파일 설정](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-files.html)  
AWS CLI에서 유지되는 파일에 자주 사용되는 구성 설정과 자격 증명을 저장할 수 있습니다 (`~/.aws/config` `~/.aws/credentials`)

파일은 profiles로 나뉘는데, 기본적으로는 default에 있는 값을 사용하고, `--profile` 옵션이나 `AWS_PROFILE` 환경변수 값을 가지고 다른 설정을 참조해서 cli command를 날릴 수 있다.

설정하기
```
aws configure --profile <profile_name>

예)
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

리스트 보기
```
aws configure list
aws configure list-profiles
```

#### VPC
#### Security Group
#### CloudTrail
#### s3 
비용
1. get/pust/list 요청 비용은 모두 다르다  
put, copy, post, list >> (10배) >> get, select

2. 작은 크기의 파일을 아주 많이 저장하는 것은 좋지 못하다.  
s3 -> s3 glacier로 수명주기 전환 요청 비용이 파일 개수 비용만큼 추가됨. 그리고 glacier에 저장할 때는 원본 + 메타데이터도 저장되므로, 이 메타에 대한 storage 오버헤드도 객체 개수만큼 생성됨.  
일정 주기로 큰 파일로 압축해서 저장할 것을 권장..

3. 데이터 트랜스퍼 비용  
vpc 인바운드 트래픽은 과금 안하고, vpc 아웃바운드만 과금하는 네트워크 정책이 있는데 s3, 다이나모 db는 vpc 바깥에 있는 서비스이다.  
s3와 연결할 때 인터넷 게이트 웨이를 통하지 말고, 게이트웨이 엔드포인트를 추가하는 것이 비용 절감하는 방법.  
라우팅 테이블에 s3로 바로갈 수 있는 라우팅 정보 추가하기.  

#### slack으로 장애 알람 받기

serverless
----------
프로비전하거나 관리할 인프라가 없다.  
사용량에 따라 자동으로 스케일링된다. (서버 유닛 단위가 아니라 작업 유닛 단위로)

### lambda
- 람다 함수 하나당 하나의 실행한경을 가지고, 실행환경 재사용 가능하다. (warm start)
- 메모리 : 120mb ~ 10gb 사이의 메모리를 설정할 수 있다. 어느 정도 큰 데이터 프로세싱도 람다로 가능..
- cpu와 네트워크 : 메모리 사이즈에 따라 같이 증가
- 타임아웃 : 최대 15분
- 보안이 잘 갖춰진 vpc 안에서 호출된다.
- 로그 확인은 cloudwatch 로그 그룹으로

#### 3가지 호출 모델
![람다 호출 모델](./image/lambda_execution_model.png)
1) 동기 방식
2) 비동기 방식 -> 이벤트가 람다함수 trigger. 
3) stream -> streaming 데이터를 polling 해서 람다함수 내에서 처리

#### 콜드 스타트

![람다 라이프사이클](./image/coldstart.png)
- 패키지 사이즈, 런타임, 코드 최적화로 콜드스타트 타임 줄이기 (필요한것만 import하기, 코어 모듈만)
- 더 높은 메모리 (+cpu 사이클) 할당하는 것으로 콜드스타트 타임 줄이기.
- 초기화 로직을 핸들러 함수 바깥에 빼서, handler 함수 호출될때마다 중복으로 실행되지 않도록 하기.
- 함수는 아주 작은만 기능하도록 작게 쪼개기.
- lambda 언어 선택 : 아무래도 java같은 jvm 기반 언어들은 로딩 시간이 조금 더 걸린다. 스크립트 기반 언어가 더 빠르게 실행 가능.

#### concurrency
- 초당 max 실행환경 개수.  
- 한 실행환경마다 한번 cold start 되고나면, 그다음에 처리하는 request는 warm 상태로 커넥션, 라이브러리 초기화 과정없이 실행할 수 있음.
- 리전별, account 별 쿼터제.   
`reserved concurrency` -> 특정 함수에 대해서만 실행환경 pool을 사전에 정의해둔다. production 함수 같이 중요한 경우에 사용.  
`provisioned concurrency` -> 미리 warm pool 만들어두기. 갑자기 많은 트래픽이 몰려와도 콜드 스타트 없이 빠르게 실행될 수 있도록. (큰 쇼핑몰 이벤트 할 때)  

#### lambda 제공 여러 feature
- `layer` => 모듈 라이브러리 사이즈가 크고 거의 변경되지 않는 경우 사용하면 좋다. 콜드 스타트 타임 줄일 수 있음.
- `dead-letter queue` => 람다 비동기 호출 retry 횟수ㅊ 초과로 실패했을 때 DLQ로 보내서 메세지 아예 유실되지 않고, 나중에 확인하고 재처리할 수 있도록 하기. 메세지 알람 걸어놓으면 좋다.
- `rds proxy` => 람다와 rds 연결할 때. shared connection pool. 기본적으로 람다는 실행환경당 1개의 connection을 가지므로, concurrency 개수가 늘어나면 rds와의 connection을 효율적으로 관리할 수 없는데, 중간에 rds-proxy를 두어 해결한다.
- 버저닝, aliasing


### API gateway
#### 제공하는 프로토콜
1. RESTFUL api
- rest api (더 많은 기능 제공 - 메세지 validation, api key 관리, tracing 등)
- http api (간소화된 기능, 좀 더 빠른 실행시간, 가격 저렴)  
2. websocket

#### 3가지 endpoint 타입
1. regional -> 가장 기본적인 형태
2. edge-optimized -> 글로벌 서비스 구성할 때. cloudfront cdn 활용해서 구현되어 있음
3. private -> internal api. vpc 안에서만 접근할 때.

### SNS
- pub/sub 메세징 처리
- fanout 패턴, reactor 패턴
- subscriber 별로 메세지를 필터링할 수 있다.

타입
- standard -> 메세지 처리양이 많지 않거나, 순서가 엄청 엄격하게 지켜져야하는 경우가 아니거나, 약간의 중복이 허용되는 경우라면
- fifo -> payment같이 순서가 매우 중요하고 중복되면 절대 안되는 경우 fifo 선택.

### SQS
- 메세지 큐 서비스
- batch 형태로 여러개의 메세지 한꺼번에 polling.
- consumer가 메세지를 가져가면 visibility timeout 기간만큼 invisible 상태가 되서 다른 consumer들이 가져가지 못하게 함. 그 사이에 consumer가 처리 완료해서 큐로 삭제 요청 보내든지, 다시 visible 해지든지 둘중하나.

타입
- standard 
- fifo




🌵 sqs vs sns

![sqs vs sns](./image/sqs_sns.png)
|   |sqs|sns|
|---|------|---|
|메세지 consume    |pull 방식                               |push 방식                         |
|                |사용자가 batch로 뽑아다가 사용               |subscribe하고 있던 사용자들에게 publish|
|생성하는 리소스     |큐                                     | 토픽                              |
|메세지 persistency|retention period 동안 버퍼(큐)에 메세지 보관 |fire and forget.                  |
|사용 목적          |메세지 브로커를 둔 비동기 처리               |pub/sub을 위함. reactor pattern    |

sqs는 sidekiq, sns는 redux가 생각난다...

서로 배타적인 서비스가 아니다. 다음과 같이 함께 구성하면 더 안정적인 scale-out 구조를 만들 수 있다.

![scale-out sqs, sns](./image/sqs_sns2.png)

### EventBridge
이벤트 버스. 메세지 라우팅으로 견고한 event driven system 구축하는데 사용. 규칙을 정해서 타겟으로 라우팅 한다.

rule 동작
- `이벤트 패턴`에 반응해서 동작할 수도 있고
- 특정한 `time interval`로 동작할 수도 있음 (schedule)  
람다에 주기적으로 trigger를 걸어서 cron job을 실행하게 하려면 이 schedule rule을 이용하면 좋다.


[serverless 실습](serverless_workshop.md)

### SAM
로컬에서 IDE로 개발하고 sam cli를 통해서 배포하면 내부적으로 cloudFormation CDK로 변환해서 스택 형태로 배포되는 모델.  

오픈소스 프레임워크이다.

sam 템플릿으로 여러가지 리소스 기술.

![sam](./image/sam.png)

[sam 실습](https://catalog.us-east-1.prod.workshops.aws/workshops/d21ec850-bab5-4276-af98-a91664f8b161/ko-KR)


DevOps - CI/CD
--------------
#### codecommit
#### codebuild
#### codedeploy
![codedeploy](./image/codedeploy.png)
Deployment Group 예시.. Dev, QA, Prod..
#### codepipeline

[CICD 실습](https://catalog.us-east-1.prod.workshops.aws/workshops/cbcd960c-a07b-40c2-a01d-1d2e7a52b945/ko-KR)
