# prelab
**Hybrid Cloud 1 그룹 김세희 10341**

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

### PreLab 2-3일차 산출물 ###

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
![image](https://github.com/Kim-sehee/prelab/blob/ea4e564a085150813c4c2a99f6ab1cd10682da23/build1.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/ea4e564a085150813c4c2a99f6ab1cd10682da23/build2.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/ea4e564a085150813c4c2a99f6ab1cd10682da23/build3.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/ea4e564a085150813c4c2a99f6ab1cd10682da23/build4.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/ea4e564a085150813c4c2a99f6ab1cd10682da23/build5.JPG)

  - KUBE URL 설정은 다음과 같다.
  - 클러스터 메뉴에서 API 서버 엔드포인트 URL을 copy한다.
![image](https://github.com/Kim-sehee/prelab/blob/aabba358e87fb2cb2ffef185000a1297817a8ce8/APIserver.JPG)

  - KUBE TOKEN은 다음에서 얻을 수 있다.
```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
EOF

$ cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
EOF

$ kubectl -n kube-system describe secret eks-admin
```
 - 설정을 완료하면 최종적으로 아래와 같이 된다.
![image](https://github.com/Kim-sehee/prelab/blob/main/kube_token.JPG)

  - 인라인 정책 추가를 위해 아래의 환경에서 서비스 역할 링크로 이동한다.
![image](https://github.com/Kim-sehee/prelab/blob/main/servicerole.JPG)

  - 인라인 정책 권한을 추가하여 JSON에 해당 구문을 삽입한다.
![image](https://github.com/Kim-sehee/prelab/blob/6a747cb1bc6ce1a5fd79e796f26fa186c75bd66e/policysetting.JPG)
```
{
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:GetAuthorizationToken",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart"
      ],
      "Resource": "*",
      "Effect": "Allow"
 }
```
  - 4개의 마이크로 서비스에 대한 프로젝트 빌드를 시작한다.
![image](https://github.com/Kim-sehee/prelab/blob/02455ed1a615c375d99ae8a0de0b74d9f9dfc149/orderbuild.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/02455ed1a615c375d99ae8a0de0b74d9f9dfc149/productbuild.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/02455ed1a615c375d99ae8a0de0b74d9f9dfc149/gatewaybuild.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/02455ed1a615c375d99ae8a0de0b74d9f9dfc149/deliverybuild.JPG)

- **마이크로서비스 테스팅**
  - 테스트를 수행하기 전에 배포된 Gateway가 라우팅할 마이크로서비스의 Endpoint를 확인한다.
```
$ kubectl get service
```
  - 배포된 마이크로서비스를 테스트하기 전, 통합 메시징 인프라(Kafka)가 설치되어야 한다.
  - 설치 방법은 아래와 같이 수행한다
  - Kafka 설치
```
$ helm repo update
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ kubectl create ns kafka
$ helm install my-kafka bitnami/kafka --namespace kafka
```
  - Kafka Monitor 설치
```
helm repo add kafka-ui https://provectus.github.io/kafka-ui
helm repo update
helm install kafka-ui kafka-ui/kafka-ui  --namespace=kafka \
--set envs.config.KAFKA_CLUSTERS_0_NAME=shop-Kafka \
--set envs.config.KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=my-kafka:9092 \
--set envs.config.KAFKA_CLUSTERS_0_ZOOKEEPER=my-kafka-zookeeper:2181
```

![image](https://github.com/Kim-sehee/prelab/blob/93ecce9f26fcce97cbcbe86b08d9bd118f127f3d/kafkasetting1.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/93ecce9f26fcce97cbcbe86b08d9bd118f127f3d/kafkasetting2.JPG)
![image](https://github.com/Kim-sehee/prelab/blob/93ecce9f26fcce97cbcbe86b08d9bd118f127f3d/kafkasetting3.JPG)
