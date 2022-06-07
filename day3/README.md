DAY3
====

container
---------

observability
-------------

amplify
-------

CDK
---
IaC (Infrastructure as Code)  
인프라를 코드로 관리하기.

[cdk 실습](cdk.md)

### aws 리소스 생성하는 방법
1. 웹 콘솔
- 가장 쉽고, 편리하다.
- 하지만 비슷한 리소스를 반복해서 배포하고, 수정해야 할 경우 비생산적이고, 휴먼에러가 발생할 가능성이 높다.

2. script
- 웹 콘솔보다 자동화할 수 있는 방법.
- aws에서 제공하는 api로 리소스를 생성, 삭제, 변경하기 (sdk)
- 하지만 이것도 리소스 변경이 필요할 때마다 자주 코드를 수정해줘야 한다.
- `declarative` x `imperative` o 중간에 에러 발생했을 때 처리가 어렵다.

3. provisioning engine
- `Ansible playbook`, `Terraform hashicorp`, `AWS cloudformation` 같은 것들이 있다.
- `declarative` 하게 원하는 프로비저닝 된 모습을 정의하면 그대로 만들어 준다.
- 그러나 config할게 많고 사용하기에 learning curve가 높다.

4. CDK
- (3)번 방식의 CloudFormation을 쉽게 사용할 수 있도록 추상화된 프레임워크 도구
- 친숙한 프로그래밍 언어를 사용해서 app 개발하듯이 인프라를 생성, 관리
- 리소스 정의를 컴포넌트화, 재사용 가능.

### CDK 3가지 구성 요소

![cdk 구조](./image/cdk_app.png)
1. core framework -> app, stack, resources

app은 다수의 stack으로 구성되어 있고, stack안에 construct로 각종 aws 서비스 정의한다.

`construct` => 가장 기본적인 building block  
level1 : 가장 저수준. cfn이라는 prefix가 붙은 경우. 이것을 사용하게 되는 경우는 자주 있진 않을 것이다.  
level2 : 좀더 고수준. 각종 메소드가 정의되어 있어서 필수적인 값들만 오버라이드하여 사용한다.  
level3 : aws 서비스들로 구성된 패턴. 예) `ecs_pattern.ApplicationLoadBalancedFargateService`

[cdk reference 문서 참고하기](https://docs.aws.amazon.com/cdk/api/v2/)  
[aws 솔루션 구문 패턴](ucts-master-cards.sort-by=item.additionalFields.headline&constructs-master-cards.sort-order=asc&awsf.constructs-master-filter-tech-categories=*all&awsf.constructs-master-filter-products=*all)  

2. construct library -> aws 서비스를 다루기 위한 컴포넌트 클래스
3. cdk cli -> cli 명령어로 빌드, cloudformation stack 합성, 배포


### 작동방식
1. cdk app 작성
2. 빌드하기 (예를들면 ts -> js)
3. cdk synth : cloudformation template stack으로 변환
4. cdk deploy : 배포. aws에 리소스 프로비저닝 됨.
