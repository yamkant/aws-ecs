## AWS ECS 구현 방법

### AWS ECS CLI 설치 및 계정 생성
- aws ecs를 위한 CLI를 설치합니다.
    ```sh
    # aws ecs를 위한 CLI를 설치합니다.
    $ curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
    $ sudo installer -pkg AWSCLIV2.pkg -target /
    ```

- AWS CLI에 접근하기 위해 아래 값들을 설정합니다.
    ```sh
    $ aws configure
    AWS Access Key ID [None]: ACCESS_KEY_ID
    AWS Secret Access Key [None]: SECRET_ACCESS_KEY
    Default region name [None]: ap-northeast-2
    Default output format [None]: json
    ```
- configure 설정이 끝나면 `~/.aws/credentials` 폴더가 생성됩니다.

- ECS CLI를 사용하기 위해 IAM 권한을 가진 계정을 생성합니다.  
  (위에서 configure에 등록한 계정은 `iam:CreateUser` 권한 필요 - `IAMUserFullAccess` 정책을 부여했습니다.)
    ```sh
    $ aws iam create-user --user-name ecs-user
    > "Arn": "arn:aws:iam::301869408653:user/ecs-user"

    $ aws iam create-access-key --user-name ecs-user
    > "AccessKeyId": "AKIAUMSGRUGGXHHX6257",
    > "SecretAccessKey": "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"

    # ECS 접근을 위한 정책들을 설정합니다.
    $ aws iam attach-user-policy \
        --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --user-name ecs-user
    $ aws iam attach-user-policy \
        --policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess --user-name ecs-user
    $ aws iam attach-user-policy \
        --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess --user-name ecs-user

    # 아래 json 파일을 생성한 후, 해당 경로에서 명령어를 실행하여 정책을 추가합니다.
    $ aws iam create-policy --policy-name ecsUserPolicy --policy-document file://ecs-user-policy.json
    > "PolicyId": "ANPAUMSGRUGG5Z2CWN5G3",
    > "Arn": "arn:aws:iam::301869408653:policy/ecsUserPolicy",

    $ aws iam attach-user-policy \
        --policy-arn arn:aws:iam::301869408653:policy/ecsUserPolicy \
        --user-name ecs-user
    
    # 위에서 생성한 ecs-user로 access key와 secret key를 등록하여 다시 인증합니다.
    $ aws configure
    ```
    ecs-user 추가적인 권한 설정을 위한 정책
    ```
    # ecs-user-policy.json
    {
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:iam::301869408653:role/ApplicationAutoscalingECSRole"
        }, {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeAddresses",
                "ec2:AllocateAddress",
                "ec2:DescribeInstances",
                "ec2:AssociateAddress"
                "ecr:CreateRepository"
            ],
            "Resource": "*"
        }]
    }
    ```

### Docker 구성 및 ECR 업로드
**간단한 설명**
- ecs에서의 최소 단위는 "태스크"이며, 하나의 태스크 내에는 다수의 이미지 파일을 사용할 수 있습니다.
- 하나의 "태스크"를 구성할 때 빌드시킬 도커 이미지는 ECR이라고 하는 저장소에 push 해야합니다.

**ECR 등록 및 업로드**
- ECR Repository 생성
    ```sh
    $ aws ecr create-repository --repository-name my-web
    > "repositoryUri": "301869408653.dkr.ecr.ap-northeast-2.amazonaws.com/my-web"
    $ aws ecr create-repository --repository-name my-nginx
    > "repositoryArn": "arn:aws:ecr:ap-northeast-2:301869408653:repository/my-nginx"
    ```
- 

### ECS 클러스터 생성
[ECS cluster 생성 예시](images/../srcs/images/ecs-cluster-setting.png)
- 빠른 구현을 위해 `VPC`는 기본으로, EC2를 이용한 서버 구성으로 진행합니다. (프리티어 사용 가능)
- 일정 시간이 지나고, 클러스터가 생성된 직 후, EC2 대시보드를 보면 인스턴스도 함께 생성됩니다.
- 새로 생성된 인스턴스 특징: `IAM: ecsInstanceRole`, `AMI: ami-ecs`, `ASG(Auto Scailing Group) 설정됨`, `기본 보안 그룹 설정`
- 따라서, 만약 ECS에 등록된 인스턴스를 추가하고자 하면, 위의 설정을 그대로 EC2를 생성하면 됩니다.




  
  

- ECR 로그인 정보를 사용하여 aws cli 도커에 로그인합니다.
```
- aws ecr get-login-password --region ap-northeast-2 | docker login \
    --username AWS --password-stdin 218479086962.dkr.ecr.ap-northeast-2.amazonaws.com
```



