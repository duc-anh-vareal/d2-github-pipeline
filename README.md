# Deploy ECS with CloudFormation on CodePipeline

### Dockerize a simple app

```
# Run on local
docker build -t books-api ./app/
docker run -it -p 4567:4567 --rm books-api:latest
open http://localhost:4567/
open http://localhost:4567/stat
open http://localhost:4567/api/books
```

### Push Docker Image to ECR
```
aws ecr create-repository --repository-name books-api
aws ecr get-login --no-include-email | sh
IMAGE_REPO=$(aws ecr describe-repositories --repository-names books-api --query 'repositories[0].repositoryUri' --output text)
docker tag books-api:latest $IMAGE_REPO:init
docker push $IMAGE_REPO:init
```

### Create CloudFormation Stacks

```
aws cloudformation create-stack --template-body file://$PWD/infra/vpc.yml --stack-name vpc
aws cloudformation create-stack --template-body file://$PWD/infra/iam.yml --stack-name iam --capabilities CAPABILITY_IAM
aws cloudformation create-stack --template-body file://$PWD/infra/app-cluster.yml --stack-name app-cluster
aws cloudformation deploy --template infra/api.yml --stack-name api --parameter-overrides DockerImage=$IMAGE_REPO:v1 
aws cloudformation deploy --template infra/pipeline.yml --stack-name pipeline --parameter-overrides GitHubRepositoryName=d2-github-pipeline GitHubAccountName=duc-anh-vareal GitHubOAuthToken=XXXX --capabilities CAPABILITY_IAM
```

```
aws cloudformation deploy --template infra/secret-manager.yml --stack-name secret-manager --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM
aws cloudformation deploy --template infra/dynamo-db.yml --stack-name dynamo-db --capabilities CAPABILITY_IAM
```
