# simple-fastapi-using-mysql-container-to-ecr

🫵🫵🫵 簡単なFastAPIアプリケーション(DB操作あり)をECRにデプロイしてみる！  

## ローカルでの開発

DevContainerに入り、以下のコマンドを実行してください。  

```shell
cd ./fastapi-app/
python -m venv /myenv/
source /myenv/bin/activate
pip install -r ./requirements.txt
uvicorn main:app --reload --host 0.0.0.0 --port 80
```

FastAPIのDockerコンテナをローカルで起動させる場合には、以下のコマンドを実行してください。  

```shell
docker build -t fastapi-app ./fastapi-app/
docker run --rm --name fastapi-app -p 80:80 fastapi-app
```

## デプロイ

DevContainerに入り、以下のコマンドを実行してください。  
※ `~/.aws/credentials`にAWSの認証情報があることを前提とします。  

```shell
export ECR_REPOSITORY_URI=$(aws ecr describe-repositories --repository-names fastapi-db-app --query 'repositories[0].repositoryUri' --output text)
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REPOSITORY_URI

docker build -t fastapi-db-app ./fastapi-db-app/
docker tag fastapi-db-app:latest $ECR_REPOSITORY_URI:latest
docker push $ECR_REPOSITORY_URI:latest
```

GitHub Actionsでデプロイする場合には、以下のシークレットを設定してください。  

| シークレット名 | 説明 |
| --- | --- |
| AWS_ACCESS_KEY_ID | AWSのアクセスキーID |
| AWS_SECRET_ACCESS_KEY | AWSのシークレットアクセスキー |
| AWS_REGION | AWSのリージョン |

タグをプッシュすると、GitHub Actionsがデプロイを行います。  
手動でトリガーすることも可能です。  

---

デプロイされたECRをローカルでテストする場合には、以下のコマンドを実行してください。  

```shell
export ECR_REPOSITORY_URI=$(aws ecr describe-repositories --repository-names fastapi-db-app --query 'repositories[0].repositoryUri' --output text)
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REPOSITORY_URI

docker run --rm -p 80:80 --name fastapi-db-app $ECR_REPOSITORY_URI:latest
```
