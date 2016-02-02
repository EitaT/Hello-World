# Amazon EC2 Container Registry（Docker レジストリ）
※ EC2 Container RegistryをECRと表記しています。

## Prerequisite

### AWS CLIインストール

``` console
$ sudo curl -L https://bootstrap.pypa.io/get-pip.py | sudo python
$ sudo pip install awscli
```

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
}
```

* 管理者権限

``` s_ECS_ECR_FullAccessUser.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:*",
        "cloudformation:CreateStack",
        "cloudformation:DeleteStack",
        "cloudformation:DescribeStack*",
        "cloudformation:UpdateStack",
        "cloudwatch:GetMetricStatistics",
        "ec2:Describe*",
        "elasticloadbalancing:*",
        "ecs:*",
        "iam:ListInstanceProfiles",
        "iam:ListRoles",
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}
```

## Login
``` console
$ $(aws ecr get-login)
WARNING: login credentials saved in /home/vagrant/.dockercfg.
Login Succeeded
```

## リポジトリの一覧を取得する
``` json
$ aws ecr describe-repositories
{
    "repositories": [
        {
            "registryId": "041095653195",
            "repositoryName": "mik/oracle-xe-11g-with-schema",
            "repositoryArn": "arn:aws:ecr:us-east-1:041095653195:repository/mik/oracle-xe-11g-with-schema"
        }
    ]
}
```

## リポジトリ内のイメージ一覧を取得する
``` json
$ aws ecr list-images --repository-name mik/oracle-xe-11g-with-schema
{
    "imageIds": [
        {
            "imageTag": "latest",
            "imageDigest": "sha256:162c055172ac188ab77b7271d443052ca853ea46b2f7ca22bac9323890d58e16"
        }
    ]
}
```

## DockerHubや、その他のDockerRegistryからImageを移行する手順
* mik/oracle-xe-11g-with-mstdataというDockerImageがローカルにあるという前提
* ECRにRepositoryを作成する。

``` json
$ aws ecr create-repository --repository-name mik/oracle-xe-11g-with-mstdata
{
    "repository": {
        "registryId": "041095653195",
        "repositoryName": "mik/oracle-xe-11g-with-mstdata",
        "repositoryArn": "arn:aws:ecr:us-east-1:041095653195:repository/mik/oracle-xe-11g-with-mstdata"
    }
}
```

* ローカルのDockerImageに、ECRのURLをTag付けする。

``` console
$ docker tag mik/oracle-xe-11g-with-mstdata:latest 041095653195.dkr.ecr.us-east-1.amazonaws.com/mik/oracle-xe-11g-with-mstdata:latest 
```

* ECRへPushする

``` console
$ docker push 041095653195.dkr.ecr.us-east-1.amazonaws.com/mik/oracle-xe-11g:latest
```

## Pullする
``` console
$ docker pull 041095653195.dkr.ecr.us-east-1.amazonaws.com/mik/oracle-xe-11g:latest
```

## その他のコマンド
``` console
$ aws ecr help
NAME
       ecr -
DESCRIPTION
       The  Amazon EC2 Container Registry makes it easier to manage public and
       private Docker images for AWS or on-premises environments. Amazon ECR
       supports resource-level permissions to control who can create, update,
       use or delete images. Users and groups can be created individually in
       AWS Identity and Access Management (IAM) or federated with enterprise
       directories such as Microsoft Active Directory. Images are stored on
       highly durable AWS infrastructure and include built-in caching so that
       starting hundreds of containers is as fast as a single container.
AVAILABLE COMMANDS
       o batch-check-layer-availability
       o batch-delete-image
       o batch-get-image
       o complete-layer-upload
       o create-repository
       o delete-repository
       o delete-repository-policy
       o describe-repositories
       o get-authorization-token
       o get-download-url-for-layer
       o get-login
       o get-repository-policy
       o help
       o initiate-layer-upload
       o list-images
       o put-image
       o set-repository-policy
       o upload-layer-part
``` 
