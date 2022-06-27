# prelab

### PreLab 1일차 산출물 ###

- **환경설정**
  - 할당 받은 AWS Cloud 계정을 가지고 환경설정을 한다.
  - AWS console의 IAM 화면의 사용자 메뉴에서 Access Key ID와 Secret Access Key를 확인한다.
![image](https://github.com/Kim-sehee/prelab/blob/cf47fa3ddf98742cc043557da54978ee993e7200/accesskey_setting.JPG)
  - IAM 보안자격증명 설정은 아래 명령어를 수행한다.
```
$ aws configure
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]: 
Default output format [None]:
```
  - 참고로, Default output format은 json 구문을 넣어주어야한다.

  - IAM 설정 확인은 아래 명령어를 수행한다.
```
$ aws iam list-account-aliases
```

  - IAM 클러스터를 생성한다.
```
$ eksctl create cluster --name user12-eks --version 1.21 --nodegroup-name standard-workers --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 3
```

  - IAM 클러스터 토큰을 생성한다
```
$ aws eks --region ap-northeast-1 update-kubeconfig --name user12-eks
```

  - 클러스터 설정 확인은 아래로 수행한다.
```
$ kubectl config current-context
```

  - 컨테이너 레지스트리(ECR)을 마이크로 서비스 개수만큼 생성한다.
 ```
 $ aws ecr create-repository \
    --repository-name user12-gateway  \
    --image-scanning-configuration scanOnPush=true \
    --region ap-northeast-1
 $ aws ecr create-repository \
    --repository-name user12-order  \
    --image-scanning-configuration scanOnPush=true \
    --region ap-northeast-1
 $ aws ecr create-repository \
    --repository-name user12-delivery  \
    --image-scanning-configuration scanOnPush=true \
    --region ap-northeast-1
 $ aws ecr create-repository \
    --repository-name user12-product  \
    --image-scanning-configuration scanOnPush=true \
    --region ap-northeast-1
 ```
 
  - 생성 완료 후 아래 명령어로 확인하면 다음과 같이 생성한 것을 확인할 수 있다
 ```
 aws ecr list-images --repository-name user12
 ```
