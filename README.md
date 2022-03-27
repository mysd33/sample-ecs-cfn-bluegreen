# AWS CloudFormationを使用してCodeDeployを通じてECS Blue/Greenデプロイを実行するsample

## 参考
* https://docs.aws.amazon.com/ja_jp/codedeploy/latest/userguide/deployments-create-ecs-cfn.html
* https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/blue-green.html
* https://qiita.com/yusuke-ka/items/02207af95b88ce06bd51

## 制約
* 既存のリソースのImportが使えない
* 大阪リージョンでは使えない

## 環境構築
### 1. VPCおよびサブネット、InternetGateway等の作成
```sh
aws cloudformation validate-template --template-body file://cfn-vpc.yaml
aws cloudformation create-stack --stack-name Sample-VPC-Stack --template-body file://cfn-vpc.yaml
```

### 2. ECSおよびBlueGreenデプロイ等の作成
* 既存のリソースのImportが使えないので、パラメータでVPC、SubnetIDを指定する
```sh
aws cloudformation validate-template --template-body file://cfn-ecs.yaml
aws cloudformation create-stack --stack-name Sample-ECS-Stack --template-body file://cfn-ecs.yaml --parameters ParameterKey=Vpc,ParameterValue=vpc-08d5ce502d2cb5eb8 ParameterKey=Subnet1,ParameterValue=subnet-0173f5ff259a73573 ParameterKey=Subnet2,ParameterValue=subnet-0fee1107c6c5f7344 --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND
```

### 3. BlueGreenデプロイ
* cfn-ecs.yaml修正してスタック更新
  * イメージタグ「Image: 'nginxdemos/hello:latest'」を「Image: 'nginxdemos/hello:plain-text'」に変える等して、TaskDefinitionかTaskSetに変更が入るようにするとBlueGreenデプロイが起動する
```sh
aws cloudformation validate-template --template-body file://cfn-ecs.yaml
aws cloudformation update-stack --stack-name Sample-ECS-Stack --template-body file://cfn-ecs.yaml --parameters ParameterKey=Vpc,ParameterValue=vpc-08d5ce502d2cb5eb8 ParameterKey=Subnet1,ParameterValue=subnet-0173f5ff259a73573 ParameterKey=Subnet2,ParameterValue=subnet-0fee1107c6c5f7344 --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND
```