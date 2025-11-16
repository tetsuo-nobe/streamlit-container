# streamlit-container


## ECS タスクとしてデプロイして、ALB との連携時の注意
* AWS マネジメントコンソールで サービス作成時に ALB を作成する場合、タスク用に指定したサブネットに ALB も作成される
    - タスク向けのサブネットを Private にすると、ALB は「インターフェース向け」設定で Private サブネットに作成される（よってアクセスできない）
* CloudFront の VPC オリジンでアクセスする場合は、ALB を 「内部向け」で作成する必要がある。
    -  そのため、AWS マネジメントコンソールでサービス作成時に ALB を作ってはいけない。
    - 事前に内部向け ALB とターゲットグループを作成しておく

* 上記から総じて、ECSサービスと連携する ALB は 事前に作成しておくのがよい

## コンテナイメージをビルドして Amazon ECR に push するまでの手順

```
docker build -t streamlit-bedrock-app .
```

```
docker run -d -p 8501:8501 -e AWS_DEFAULT_REGION=ap-northeast-1 streamlit-bedrock-app
```

```
http://localhost:8501
```

```
aws ecr create-repository --repository-name streamlit-bedrock-app --region ap-northeast-1
```

```
AWS_ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
echo ${AWS_ACCOUNT}

aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin ${AWS_ACCOUNT}.dkr.ecr.ap-northeast-1.amazonaws.com
```

```
docker tag streamlit-bedrock-app:latest ${AWS_ACCOUNT}.dkr.ecr.ap-northeast-1.amazonaws.com/streamlit-bedrock-app:latest
```

```
docker push ${AWS_ACCOUNT}.dkr.ecr.ap-northeast-1.amazonaws.com/streamlit-bedrock-app:latest
```