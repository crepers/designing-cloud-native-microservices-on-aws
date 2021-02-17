[<04 모델링 및 개발로 돌아 가기>](../04-modeling-and-development/README.md)

# AWS CDK와 함께 AWS Code* Suite를 통해 Coffeeshop 애플리케이션 배포

**이 저장소에서 계속 학습할 수 있는 쿠도(Kudos)입니다. 이제 애플리케이션을 실제 AWS 환경에 배포해야 합니다.**


![](../img/EventStormingWorkshop-CDK.jpg)

AWS에 애플리케이션을 배포하려면 다음과 같은 필수 도구가 설치되어 있어야 합니다.

* [AWS CLI](https://docs.aws.amazon.com/zh_tw/cli/latest/userguide/cli-chap-install.html)
* [AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html)



## 배포 지침

### 이 저장소(repo)를 자신의 github 계정에 포크

이 워크숍은 Github 웹훅 메커니즘을 활용하여 애플리케이션을 AWS에 자동으로 빌드/배포하므로 첫 번째 단계는 포크입니다.

또한 소스 코드에서 repo 소유자 정보를 업데이트하십시오.

**EventStormingWorkShop/deployment/coffeeshop-cdk/lib/coffee-shop-code-pipeline.ts**



> 소유자를 github 계정으로 바꿉니다.

```typescript
const defaultSource = codebuild.Source.gitHub({
            owner: 'humank',
            repo: 'EventStormingWorkShop',
            webhook: true, // optional, default: true if `webhookFilteres` were provided, false otherwise
            webhookFilters: [
                codebuild.FilterGroup.inEventOf(codebuild.EventAction.PUSH).andBranchIs('master'),
            ], // optional, by default all pushes and Pull Requests will trigger a build
        });
```





-----

### Github Webhook 통합 가져 오기

AWS Codebuild 콘솔을 열고 **Create build project** 를 클릭합니다. 이 단계를 활용하여 webhook을 설정하지만 실제로 빌드 프로젝트를 저장할 필요는 없습니다. 스크린 샷을 따르십시오.

> 빌드 프로젝트 생성

![](../img/setup-webhook.png)



> 소스를 지정하고 github를 선택한 다음 **Github에 연결을 클릭합니다**

![](../img/specify-source.png)



> AWS CodeBuild를 승인하고 **Authorize aws-codesuite** 를 클릭하고 **암호** 를 확인합니다.

![](../img/authorize.png)



> Github에 연결합니다.

![](../img/get-connected.png)



**이제 github 계정이 aws-codesuite에 연결되었으므로 이 코드 프로젝트를 저장할 필요없이 취소만하면 됩니다. 이 단계는 웹훅 인증만을 위한 것입니다.**

-----

### CDK로 Code* family를 이용하여 CI/CD 파이프 라인을 사용한 인프라 및 애플리케이션 배포

**이 CDK 애플리케이션을 실행하면 3 개의 가용성 영역 환경이있는 표준 VPC와 프라이빗 서브넷을 제공하는 하나의 NATGateway가 제공됩니다.**

**또한, 컨테이너 오케스트레이션 서비스를 쉽게 사용할 수 있도록 Fargate 모드의 ECS 클러스터도 생성됩니다.**

### Code* family를 이용한 애플리케이션 배포

```shell script
cd deployment/coffeeshop-cdk

npm install

npm run build 

cdk synth

cdk bootstrap aws://${your-aws-id}/${your-region-todeploy}

cdk deploy CoffeeShopCodePipeline 
```

**이 워크숍 샘플 코드는 Maven에서 관리하는 Libs 종속성인 Quarkus Framework를 사용하여 Java8로 개발되었습니다. 이 CDK CoffeeShopCodePipeline 스택을 실행하면 다음을 확인 수 있습니다.**

* ECR - Orders-Web 애플리케이션을 제공하기 위해 Docker 이미지 저장소를 생성합니다.
* CodeCommit 리포지토리 - 자동 배포용
* CodeBuild - Github WebHooked 프로젝트 가져오기, 소스코드 빌드, Docker 이미지 빌드, ECR에 이미지 푸시, **Orders-web** Fargate 서비스 배포, **coffee-sls Lambda 함수** 배포, **Dynamodb 테이블 생성 - { Order, Coffee}**, 기본 **Amazon EventBridge** 에서 이벤트 규칙을 만듭니다. 기타 등등..



**배포 결과**

```shell
Outputs:
CoffeeShopCodePipeline.CodeBuildProjectName = CodeBuildProject
CoffeeShopCodePipeline.AlbSvcServiceURL46A1D997 = http://Coffe-AlbSv-5MLHALGIGWUB-82783022.us-west-2.elb.amazonaws.com
CoffeeShopCodePipeline.AlbSvcLoadBalancerDNS20AA0F0B = Coffe-AlbSv-5MLHALGIGWUB-82783022.us-west-2.elb.amazonaws.com
CoffeeShopCodePipeline.Hint =
Create a "imagedefinitions.json" file and git add/push into CodeCommit repository "EventStormingWorkshop" with the following value:

[
  {
    "name": "defaultContainer",
    "imageUri": "123456789012.dkr.ecr.us-west-2.amazonaws.com/solid-humank-coffeeshop/orders-web:latest"
  }
]

CoffeeShopCodePipeline.Bucket = coffeeshop-nypea
CoffeeShopCodePipeline.CodeCommitRepoName = EventStormingWorkshop
CoffeeShopCodePipeline.ServiceURL = http://Coffe-AlbSv-5MLHALGIGWUB-82783022.us-west-2.elb.amazonaws.com
CoffeeShopCodePipeline.StackName = CoffeeShopCodePipeline
CoffeeShopCodePipeline.StackId = arn:aws:cloudformation:us-west-2:584518143473:stack/CoffeeShopCodePipeline/f10c0520-0618-11ea-8122-023709c486f0

Stack ARN:
arn:aws:cloudformation:us-west-2:584518143473:stack/CoffeeShopCodePipeline/f10c0520-0618-11ea-8122-023709c486f0
```

["imagedefinitions.json"](https://docs.aws.amazon.com/codepipeline/latest/userguide/file-reference.html#pipelines-create-image-definitions) 파일을 만들고 CodeCommit 리포지토리에 git add/push 합니다. "EventStormingWorkshop"(위 배포의 일부로 생성됨)은 다음 값을 사용합니다.
```
[
  {
    "name": "defaultContainer",
    "imageUri": "your ecr repository arn for this coffeeshop/solid-humank-coffeeshop/orders-web:latest"
  }
]
```


### 애플리케이션 배포 방법

두 가지 방법을 통해 이러한 애플리케이션을 배포 할 수 있습니다.

1. 처음에는 CodeBuild 서비스에서 애플리케이션을 직접 직접 배포하여 CodeBuild 프로젝트를 선택하고 **빌드 시작** 버튼을 클릭하면 배포 프로세스가 시작됩니다.
2. 언제든 github의 EventStormingWorkshop 리포지토리에서 변경을 수행하는 동안 커밋하고 마스터 브랜치로 푸시하면 CodeBuild 서비스가 자동으로 이를 빌드하고 코드 파이프 라인을 트리거하여 이러한 모든 애플리케이션을 배포합니다.

### EventBridge로 Lambda 함수 트리거 설정

```shell
targetArn=$(aws lambda get-function --function-name coffee-sls-OrderCreatedHandler | jq '.Configuration.FunctionArn')

aws events  put-targets --rule OrderCreatedRule --targets "Id"="OrderCreated","Arn"=$targetArn

ruleArn=$(aws events list-rules --name-prefix OrderCreatedRule | jq -r '.Rules[0].Arn')

aws lambda add-permission \
	--function-name coffee-sls-OrderCreatedHandler \
  --action lambda:InvokeFunction \
	--statement-id stat-coffee-sls \
  --principal events.amazonaws.com \
	--source-arn $ruleArn
```

### 테스트 실행

**모든 설정이 완료되었으므로 이제 커피 주문을 위해 만든 URL을 입력 할 수 있습니다.**

**Orders-web** 서비스 엔드 포인트는 스택 출력에서 확인할 수 있습니다 - **CoffeeShopCodePipeline.AlbSvcServiceURLxxxx**

```shell
curl --header "Content-Type: application/json" \                                                                                            
        --request POST \
        --data '{"items":[{"productId":"5678","qty":2,"price":200}]}' \
        http://Coffe-AlbSv-6K6V97WJT6AV-1556524944.us-west-2.elb.amazonaws.com/order

Result : 
{"items":[{"productId":"5678","qty":2,"price":200,"fee":400}],"status":0,"id":"ord-20191126-5906","createdDate":1574801783.400000000,"modifiedDate":null}
```

**DynamoDB에서 주문(order) 테이블 확인**

![](../img/order-table-items.png)

**람다 함수 (주문 생성 이벤트 핸들러) 로그 확인**
Cloudwatch 서비스 콘솔에서, 로그 그룹 검색 : ***/aws/lambda/coffee-sls-OrderCreatedHandler***

```shell script
START RequestId: acfc1cf1-ba73-402e-921d-2fa2d95af5dc Version: $LATEST
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:39 - 0
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:40 - 7ebdf9f0-888d-540e-038d-bd6e25bea29f
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:41 - null
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:42 - solid.humank.coffeeshop.order
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:43 - 584518143473
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:44 - 2019-11-27T05:58:18Z
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:45 - us-west-2
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:46 - [Ljava.lang.String;@7ca48474
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:47 - 0
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:48 - bd56e57b-1575-49b0-b002-a8ef33c926a2
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:49 - 1
2019-11-27 05:58:23 [main] INFO  solid.humank.coffeeshop.cofee.sls.orders.OrderCreatedHandler:50 - EntityId(abbr=ord, seqNo=5907, occurredDate=2019-11-27T05:58:14.881Z)
2019-11-27 05:58:24 [main] DEBUG software.amazon.awssdk.request:84 - Sending Request: DefaultSdkHttpFullRequest(httpMethod=POST, protocol=https, host=dynamodb.us-west-2.amazonaws.com, encodedPath=/, headers=[amz-sdk-invocation-id, Content-Length, Content-Type, User-Agent, X-Amz-Target], queryParameters=[])
2019-11-27 05:58:27 [main] DEBUG software.amazon.awssdk.request:84 - Received successful response: 200
2019-11-27 05:58:27 [main] DEBUG software.amazon.awssdk.request:84 - Sending Request: DefaultSdkHttpFullRequest(httpMethod=POST, protocol=https, host=dynamodb.us-west-2.amazonaws.com, encodedPath=/, headers=[amz-sdk-invocation-id, Content-Length, Content-Type, User-Agent, X-Amz-Target], queryParameters=[])
2019-11-27 05:58:27 [main] DEBUG software.amazon.awssdk.request:84 - Received successful response: 200
Coffee made...
END RequestId: acfc1cf1-ba73-402e-921d-2fa2d95af5dc
REPORT RequestId: acfc1cf1-ba73-402e-921d-2fa2d95af5dc	Duration: 8150.39 ms	Billed Duration: 8200 ms	Memory Size: 512 MB	Max Memory Used: 156 MB	Init Duration: 887.71 ms
```

** DynamoDB에서 커피9(coffee) 테이블 확인 **

![](../img/coffee-table-items.png)



이제 전체 커피 주문 프로세스 여정을 모두 마쳤습니다. 더 많이 실습하고 싶다면 가능한 한 더 많은 비즈니스 시나리오를 구현하고 **AWS의 클라우드 커피숍** 을 모두 경험해보세요.
