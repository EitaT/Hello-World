# Amazon EC2 Container Registry（Docker レジストリ）
※ EC2 Container RegistryをECRと表記しています。

## Prerequisite

### AWS CLIインストール
sudo curl -L https://bootstrap.pypa.io/get-pip.py | sudo python
sudo pip install awscli

### User情報を環境変数に設定する
sECRuser(一般ユーザ)

``` console
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_DEFAULT_REGION=us-east-1
```


s_ECS_ECR_FullAccessUser(管理者向けユーザ)

``` console
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_DEFAULT_REGION=us-east-1
``` 

管理者と一般ユーザの違いリポジトリの作成・削除が一般ユーザの場合実施できない、管理者であれば実施できる等。

各ユーザの権限詳細は下記です。
※RcloudではIAMユーザにアタッチされたポリシーを参照できないため下記は想像です。

* 一般ユーザ権限

``` sECRuser.json 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:GetRepositoryPolicy",
        "ecr:DescribeRepositories",
        "ecr:ListImages",
        "ecr:BatchGetImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecr:PutImage"
      ],
      "Resource": "*"
    }
  ]
