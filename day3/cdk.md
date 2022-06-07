CDK 실습
=======
![cdk app](./image/cdk_app.png)

로컬에서 본인에게 익숙한 프로그래밍 언어 (ts, js, c#, java, python, go)로 aws 리소스를 정의 -> cloudFormation stack 으로 변환하고 배포 -> aws에 리소스 프로비저닝됨


#### cdk toolkit 설치하기  
cdk는 노드 기반으로 되어있어서, 툴킷을 npm으로 설치한다.
```
aws install -g aws-cdk

cdk --version
```

#### cdk 프로젝트 생성하기
```
cdk init sample-app --language typescript
```
디렉토리 구조  
`bin` -> cdk app의 엔트리포인트. lib 디렉토리 아래에 있는 정의된 stack들을 로드한다.  
`lib` -> cdk stack 정의하는 곳. 대부분의 코드는 여기에 작성하게 될 것이다.

#### 주요 command
`cdk synth` -> 작성한 코드를 CloudFormation 템플릿으로 합성  
`cdk diff` -> 현재 작성한 app과 배포된 stack 상태 비교  
`cdk deploy` -> 설정된 aws 계정/리전으로 cloudformation stack 배포ㅕㅇ령

* aws cdk 앱을 환경(계정/리전)으로 처음 배포할 때는 `cdk bootstrap` 명령어로 bootstrap 스택을 설치해야 한다.



