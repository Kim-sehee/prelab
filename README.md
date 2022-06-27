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
![image](https://github.com/Kim-sehee/prelab/blob/389926d47ef750eed5660ec3a695ce616663c3bc/ecr.JPG)

### PreLab 2일차 산출물 ###

- **코드빌드 및 프로비저닝**
  - 빌드를 시작하기에 앞서, 4개의 마이크로서비스 git 주소를 fork하는 작업을 수행한다.
    - [Gateway] : https://github.com/acmexii/gateway
    - [Order] : https://github.com/acmexii/order
    - [Delivery] : https://github.com/acmexii/delivery
    - [Product] : https://github.com/acmexii/product

  - fork한 4개의 마이크로서비스 git 저장소에 있는 buildspec.yml을 각 마이크로서비스 이름에 맞게 수정한다.
  - buildspec.yml의 10라인에 있는 버전을 corretto11로 수정한다.
![image](https://github.com/Kim-sehee/prelab/blob/2d62e71dbf94a9bf70e204165340a489be5203ec/buildspec_11.JPG)

  - AWS console에서 codebuild메뉴의 프로젝트 빌드에서 빌드 프로젝트 생성을 클릭하여 4개의 마이크로서비스 프로젝트를 빌드한다.
![image](https://github.com/Kim-sehee/prelab/blob/2d62e71dbf94a9bf70e204165340a489be5203ec/buildspec_11.JPG)  
