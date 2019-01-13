# Hello Note Kubernetes - (KUBERNETES IN THE GOOGLE CLOUD 1주차)



이번 세션에서는 2장을 리뷰한다.

​    

## Overview

먼저 Kubernetes가 무엇인지 알아보자. [Kubernetes 홈페이지](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)에서는 Kubernetes에 대해서 다음과 같이 소개하고 있다.

> Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

> The name **Kubernetes** originates from Greek, meaning *helmsman* or *pilot*, and is the root of *governor* and [cybernetic](http://www.etymonline.com/index.php?term=cybernetics). *K8s* is an abbreviation derived by replacing the 8 letters “ubernete” with “8”.



Kubernetes는 구글이 15여년 동안 걸친 대규모 운영 워크로드 운영 경험으로 만들어졌다고 소개하고 있으며, 관련 논문으로서 [Large-scale cluster management at Google with Borg](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43438.pdf)을 소개하고 있다.



해당 페이지에서는 Kubernetes는

- 컨테이너 플랫폼
- 마이크로서비스 플랫폼
- 이식성 있는 클라우드 플랫폼
- 그 외

로 구분해서 볼 수 있다고 하는데... 아리송하다.. 그래서 정확히 Kubenetes가 뭘까?..



개인적인 생각으로 Kubernetes는 **컨테이너 오케스트레이션 시스템(Container Orchestration System)**이라고 보는게 적합할 듯하다. Kubernetes 홈페이지에서는 Kubernetes가 `단순한 컨테이너 오케스트레이션 시스템`이 아니다. 라고 이야기를 하지만 시작하는 입장에서 너무 복잡하게 접근하려고하면 학습하기 어렵기 때문에, 일단은 **컨테이너 오케스트레이션 시스템(Container Orchestration System)**이라고 생각하겠다.



그럼 이제 **컨테이너 오케스트레이션 시스템(Container Orchestration System)**이라는 것은 무엇인가? 왜 컨테이너 오케스트레이션 시스템이 필요한가?에 대해서 알면 어느정도 Kubernetes이 무엇이고 왜 배워야하는지 이해할 수 있지 않을까?



앞서 우리는 Docker를 학습했다. 다시 한번 짚고 넘어가자면 Docker는 `Linux Container`기술을 통해서 개발 및 배포 환경을 통합하게끔 도와주는 기술이라고 할 수 있다. 이를 통해 얻게되는 장점은 아래와 같다고 이야기했었다.

- 복잡한 리눅스 어플리케이션을 컨테이너로 묶어서 실행할 수 있음
- 개발, 테스트, 서비스 환경을 하나로 통일하여 효율적으로 관리할 수 있음

![vms-and-containers](C:\Users\Martin\Documents\dev\TIL\Cloud\kubernetes\51082023-6ff2b180-1741-11e9-9f45-3f19ec71eb77.jpg)



그러면 Docker로 제품레벨의 서비스를 만들어서 AWS에 배포한다면 어떤 구조가 될지 생각을 해보자.

아마도 대략적으로 다음과 같은 그림이 될 것이다. 조금 더 상세히 이야기해보면

- (react / vue.js / angular 등)으로 구성된 프론트엔드
- (Django / Laravel / Spring 등)으로 구성된 백엔드
- (MySQL / MariaDB / RDS / 등등)으로 구성된 DB 
- 기타

로 서비스를 구성한 후에 이를 AWS에서 배포하는 형태가 될 것이고,  그 앞에 로드밸런서가 붙어서 접속자 수가 많아지면 이를 하나하나씩 통으로 Scale out하는 형태로 진행될 것이다. 따라서 이전에는 어플리케이션을 작성한 후에, Dockerfile로 잘 감싸서 AWS Instance에 물려주면 모든게 쉽게 해결됬던 세상이었다.



하지만.... request수가 많다고 해당 사용자들이 Scale out된 모든 서비스들(로그인 / 데이터 조회 / 결재 / 등등)을 골고루 사용하는 것일까? 내 생각은 그렇지 않다고 본다. 어떤 특정한 이유로 특정한 서비스들만 사용하는 빈도가 유독 많게 될 것이고 이로인해 Instance를 통으로 Scale out하는건 퍼블릭 클라우드 회사나 Scale out되는 Instance 비용을 지불하는 서비스 회사입장에서도 서로 좋지 않을 것이다. 



그 이유는

- 퍼블릿 클라우드 회사의 비효율적인 리소스 운영
- 특정 부분만 Scale out하면 되는데, 이를 통짜로 Scale out하는 것을 통해서 과도한 Instance 요금 지불





![aws-reinvent-2016-from-monolithic-to-microservices-evolving-architecture-patterns-in-the-cloud-arc305-16-1024](https://user-images.githubusercontent.com/13328380/51082913-b9e49300-1753-11e9-8ab7-70c77faf1f4d.jpg)



그렇다면, 이를 어떻게 해결할 수 있을까? 바로 Microservices 개념이 된다. 예를들어 기존의 백엔드 시스템이 `RESTful`방식으로 구현되어있다면  `URL Routing`을 통해서 아래와 같은 기능들이 하나의 백엔드 시스템에 통합되어있었을 것이다.

- Upload
- Streaming
- Transcode
- Download
- Recommendations
- Subscriptions



그러면 이제는 `URL Routing`으로 분기해 처리했던 방식을 하나의 `Upload` 단일 서비스, `Streaming` 단일 서비스 형태로 `Micro Service`로 나눈 이후에 이를 이용하여 서비스하자라는 개념으로 전환하면 위에서 언급되었던 단점들을 해소할 수 있을 것이다.



![video-platform-monolith-microservices](https://user-images.githubusercontent.com/13328380/51083010-7723ba80-1755-11e9-8b92-42f081f09116.png)



하지만 이렇게 구성하면, 한가지 문제점이 생기게 된다. 기존에 Docker를 이용하여 Monolith 아키텍쳐를 배포했는데, 이제는 다수의 Dockerfile을 이용하여 서비스를 관리하고 배포하고 개발해야하는 상황이 된 것이다. 하지만 Docker는 이를 다루는 용도의 도구는 아니였던 것이다.



그래서 이를 해소하기 위해서 나온 것이 **컨테이너 오케스트레이션 시스템(Container Orchestration System)**이다.  현재 대표적인 컨테이너 오케스트레이션 시스템은 아래와 같이 3가지가 있다.

![container_ochestration](https://user-images.githubusercontent.com/13328380/51083055-88b99200-1756-11e9-9c32-8f9b0c537645.PNG)



이제 감이 온다. 우리는 **컨테이너 오케스트레이션 시스템(Container Orchestration System)** 중 kubernetes라는 것을 공부하고 있는 것이다. 그럼 컨테이너 오케스트레이션 시스템이 가지고있는 기능들은 어떤 것이 있을까?

바로 다음과 같은 기능들을 제공한다.

- 컨테이너 자동 배치 및 복제
- 컨테이너 그룹에 대한 로드 밸런싱
- 컨테이너 장애 복구
- 클러스터 외부에 서비스 노출
- 컨테이너 추가 또는 제거로 확장 및 축소
- 컨테이너 서비스간의 인터페이스를 통한 연결 및 네트워크 포트 노출 제어



자 이제 Kubernetes가 무엇인지 대략적으로 알았다. 이제 본격적으로 실습을 진행해보자.



실습 시, 작업하는 것이 어떤형태로 구성되는지 참조하라고 그림을 제공해주는데 해당 그림은 다음과 같다.

![ba830277f2d92e04](https://user-images.githubusercontent.com/13328380/51083109-b3581a80-1757-11e9-8724-05322c7557de.png)

해당 그림은 다음과 같은 것을 의미한다.

- Local에서 Dockerfile을 이용해서 작은 서비스(micro service;함수)를 작성한다.

- 해당 Dockerfile을 이용하여 Docker Image로 build한다.

- build한 Image를 registry 서버에 push한다.(registry서버는 private 혹은 public이 될 수 있다.)

- kubernetes는 해당 서비스를 registry로부터 docker image를 pull 해서 사용자의 kubernetes 명령을 수행한다.

  *GCP는 Google Container Engine(GKE)라는 것을 통해서 Kubernetes를 관리하고, 사용자는 `gcloud`를 통해서 GKE를 관리할 수 있는 것으로 보인다. GKE위의 kubernetes는 사용자가 직접 kubectl을 이용하여 제어할 수 있는 것 같다.*

​    

## Node.js 어플리케이션 만들기



### Local Node.js Application

먼저 Kubernetes Engine에 배포할 어플리케이션을 작성하자

```bash
$ vim server.js
```

```javascript
var http = require('http');
var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end("Hello World!");
}
var www = http.createServer(handleRequest);
www.listen(8080)
```



위의 내용을 모두 작성했다면, 이를 실행하여 내가 작성한 어플리케이션이 잘 작동하는지 확인한다.

```bash
$ node server.js
```

![24aab6bb51533e91](https://user-images.githubusercontent.com/13328380/51083162-c28b9800-1758-11e9-909a-205d8b9192cc.png)

​    

### Make Container Image

  

#### Dockerfile

node js 어플리케이션을 만들었으니 이제 이를 기반으로 `Dockerfile`을 만들자.

```bash
$ vim Dockerfile
```

```dockerfile
FROM node:6.9.2
EXPOSE 8080
COPY server.js .
CMD node server.js
```

- 베이스 이미지를 `node:6.9.2`로 한다.
- 8080포트를 외부에 노출한다.
- local에 작성했던 `server.js`파일을 Docker Image로 복사한ㄷ다.
- `server.js`를 구동한다.

  

#### Build image

`Dockerfile`을 만들었으니, 이제 Docker Image를 Build한다.

```bash
$ docker build -t gcr.io/PROJECT_ID/hello-node:v1 .
```

  

#### Run Container

Docker Image를 만들었다면, 이제 Docker Container를 실행하자.

```bash
$ docker run -d -p 8080:8080 gcr.io/PROJECT_ID/hello-node:v1
>>
325301e6b2bffd1d0049c621866831316d653c0b25a496d04ce0ec6854cb7998
```

  

#### Test

실행한 Docker Container를 확인해보자

```bash
$ curl http://localhost:8080
>>
Hello World!
```

  

#### Stop Container

Docker Container 실행을 중지하기 위해서 Docker Container list를 확인한다

```bash
$ docker ps
>>
CONTAINER ID        IMAGE                              COMMAND
2c66d0efcbd4        gcr.io/PROJECT_ID/hello-node:v1    "/bin/sh -c 'node
```



`CONTAINER ID`를 확인했다면, 이제 Docker Container를 중지한다.

```bash
$ docker stop 2c66d0efcbd4
>>
2c66d0efcbd4
```

​    

### Push Docker Image to Registry Server

`Dockerfile`로 Build한 Docker Image가 잘 작동하는 것을 확인했으니, 이제 Docker image를 Registry 서버로 push하자!



#### change Docker Image tag

Registry 서버로 Docker Image를 Push하려면 먼저 Docker Image의 tag를 변경해줘야한다.

변경이 다 되었다면, 다음과 같은 명령어로 gcr에 Docker Image를 push해준다.

```bash
$ gcloud docker -- push gcr.io/PROJECT_ID/hello-node:v1
>>
The push refers to a repository [gcr.io/qwiklabs-gcp-6h281a111f098/hello-node]
ba6ca48af64e: Pushed 
381c97ba7dc3: Pushed 
604c78617f34: Pushed 
fa18e5ffd316: Pushed 
0a5e2b2ddeaa: Pushed 
53c779688d06: Pushed 
60a0858edcd5: Pushed 
b6ca02dfe5e6: Pushed 
v1: digest: sha256:8a9349a355c8e06a48a1e8906652b9259bba6d594097f115060acca8e3e941a2 size: 2002
```



#### Check gcr

push를 완료했다면, Docker Image가 Google Container Registry에 잘 push되었는지 확인해보자

![f795727b8d6d6cb3](https://user-images.githubusercontent.com/13328380/51083260-67f33b80-175a-11e9-924a-ee08519ffaaa.png)

​     

## Make k8s Cluster

이제 GKE에 Kubernetes를 올리고, kubernetes를 이용해서 node.js로 만든 서버를 작동시켜보자.

*GKE에 kubernetes를 설정하는 방법은 생략하겠다.*

![c9bee694e1721a07](https://user-images.githubusercontent.com/13328380/51083281-ca4c3c00-175a-11e9-9e22-80363d509bee.png)



### Creating a cluster with two nodes

다음과 같은 명령어로 2개의 노드를 갖는 클러스터를 생성한다.

```bash
$ gcloud container clusters create hello-world \
                --num-nodes 2 \
                --machine-type n1-standard-1 \
                --zone us-central1-f
>>
Creating cluster hello-world...done.
Created [https://container.googleapis.com/v1/projects/PROJECT_ID/zones/us-central1-f/clusters/hello-world].
kubeconfig entry generated for hello-world.
NAME         ZONE           MASTER_VERSION  MASTER_IP       MACHINE_TYPE   STATUS
hello-world  us-central1-f  1.5.7           146.148.46.124  n1-standard-1  RUNNING
```



생성을 완료하면, GCP에서 다음과 같은 화면을 확인할 수 있다.



![eaa78cd88776ff47](https://user-images.githubusercontent.com/13328380/51083304-26af5b80-175b-11e9-8d47-04bcaa359dbb.png)



### Make Pod

Kubernetes에서 Pod라는 개념은 `관리 및 네트워킹 용도로 서로 연결된 컨테이너 그룹`으로 설명된다. 즉, Pod는 하나 또는 그 이상의 컨테이너를 포함하고 있는 그룹이라고 이해하면 된다.



다음과 같은 명령어를 이용하여 Pod를 생성한다.

```bash
$ kubectl run hello-node \
    --image=gcr.io/PROJECT_ID/hello-node:v1 \
    --port=8080
>>
deployment "hello-node" created
```

​    

### Check Deployments

아래 명령을 이용하여 Deployments list를 확인한다.

```bash
$ kubectl get deployments
>>
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   1         1         1            1           2m
```

​    

### Check Pod made by Deployments

아래 명령을 실행하여 Deployments에 의해 만들어진 포드를 확인한다.

```bash
$ kubectl get pods
>>
NAME                         READY     STATUS    RESTARTS   AGE
hello-node-714049816-ztzrb   1/1       Running   0          6m
```

​    

### Command of kubernetes

#### cluster-info

```bash
$ kubectl cluster-info
```



#### config view

```bash
$ kubectl config view
```



#### events

```bash
$ kubectl events
```



#### logs

```bash
$ kubectl logs <pod-names>
```

​    

## 외부 트래픽 허용

pod는 기본적으로 cluster에 포함된 내부 IP로만 접근할 수 있다. 따라서 kubernetes 가상 네트워크 외부에서 특정 컨테이너에 접근할 수 있게 하려면 Pod를 Kubernetes 서비스로 노출해야한다.

다음과 같은 명령어를 이용하여 서비스를 노출한다.

```bash
$ kubectl expose deployment hello-node --type="LoadBalancer"
>>
service "hello-node" exposed
```

- `--type="LoadBalacer"`는 기반 인프라에서 제공하는 LoadBalancer를 사용할 것이라고 명시하는 옵션

- Pod를 직접 노출하는 것이 아니라, Deployment를 노출시킨다는 것을 인지하고 있어야함.

  (이렇게 할 경우, 서비스가 해당 deployment에서 관리하는 모든 Pod에 걸쳐 트래픽의 부하를 분산한다.)



#### Check IP address

아래와 같은 명령어로 클러스터 서비스를 모두 나열한 후, 네트워크 외부에서 서비스에 공개적으로 접근할 수 있는 IP 주소를 확인한다.

```bash
$ kubectl get services
>>
NAME         CLUSTER-IP     EXTERNAL-IP      PORT(S)    AGE
hello-node   10.3.250.149   104.154.90.147   8080/TCP   1m
kubernetes   10.3.240.1     <none>           443/TCP    5m
```

- 서비스에 사용할 수 있는 IP주소는 2개이다. 포트는 둘다 `8080`을 사용하고 있으나, 하나는 내부 IP이며 다른 IP는 외부 부하 분산 IP이다. 따라서 내부 IP는 클라우드 가상 네트워크 내부에서만 사용할 수 있다.

 

#### Access External IP

이제 External IP를 이용하여 서비스에 접근해보자

![67cfa8c674f8c708](https://user-images.githubusercontent.com/13328380/51083418-4182cf80-175d-11e9-89fe-9e129b0c3403.png)

​    

## 서비스의 규모 확장하기

kubernetes의 가장 강력한 기능은 어플리케이션을 간단하게 확장할 수 있다는 것이다.

어느날 갑자기 어플리케이션에 더 많은 capacity가 필요해지면, Replica Controller에 pod의 새로운 Replica 여러개를 관리하라고 할 수 있다.

이는 아래 명령으로 할 수 있다.

```bash
$ kubectl scale deployment hello-node --replicas=4
>>
deployment "hello-node" scaled

$ kubectl get deployment
>>
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   4         4         4            3           16m

$ kubectl get pods
>>
NAME                         READY     STATUS    RESTARTS   AGE
hello-node-714049816-g4azy   1/1       Running   0          1m
hello-node-714049816-rk0u6   1/1       Running   0          1m
hello-node-714049816-sh812   1/1       Running   0          1m
hello-node-714049816-ztzrb   1/1       Running   0          16m
```

- 위의 설명에서는 선언 접근법을 사용하고 있다. 즉, 신규 인스턴스를 시작하거나 중지하는 대신 항시 몇개의 인스턴스가 실행되고있어야하는지 선언한 것이다. kubernetes의 조정 루프는 사용자가 요청한 대로 구현되도록 조정하고 필요시 조취를 취한다.



![587f7f0a097aaa2](https://user-images.githubusercontent.com/13328380/51083444-c66de900-175d-11e9-93c9-eac3b08624ce.png)

​    

##  Service Upgrade Rollout

제품이 이미 배포되어 돌아가고있는 경우에도 어플리케이션에 버그나, 추가기능이 있다면 이를 적용해줘야한다. 하지만 이런 경우에 서비스를 일시적으로 중단하고 재배포를 진행해야하는데 이때 서비스는 순간적으로 굉장히 위험한 상태에 노출된다.

*(재배포를 진행하는 동안, 재배포가 알 수 없는 이유로 안되는 경우도 있으며 여러가지 문제 상황들이 생길 여지가 굉장히 많다.; 몰론 그동안 사용자들이 서비스를 이용할 수 없다는 점도 굉장히 치명적이다.)*



Kubernetes는 사용자들에게 영향을 주지 않으면서 재배포가 가능한 `무중단 배포`를 지원한다. 



### 어플리케이션 수정

`무중단 배포` 시나리오를 테스트하기 위해 코드를 일부 고쳤다고 가정하자

```bash
$ vim server.js
```



`server.js`에서 응답 메세지를 업데이트한다.

```javascript
response.end("Hello Kubernetes World!");
```



#### Re-build Docker Image

코드가 변경되었으니, 이제 Docker Image도 변경해주며, 변경이 완료되면 registry 서버에 push를 진행한다.

```bash
$ docker build -t gcr.io/PROJECT_ID/hello-node:v2 . 
$ gcloud docker -- push gcr.io/PROJECT_ID/hello-node:v2
```



#### 무중단 배포

kubernetes가 변경된 어플리케이션 버전으로 복제 컨트롤러를 원활하게 업데이트해준다.

실행 중인 컨테이너의 이미지 label을 변경하려면 기존 `hello-node deployment`를 수정하여 이미지를 `gcr.io/PROJECT_ID/hello-node:v1에서 gcr.io/PROJECT_ID/hello-node:v2`로 변경해야한다.

```bash
$ kubectl edit deployment hello-node
>>
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: 2016-03-24T17:55:28Z
  generation: 3
  labels:
    run: hello-node
  name: hello-node
  namespace: default
  resourceVersion: "151017"
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/hello-node
  uid: 981fe302-f1e9-11e5-9a78-42010af00005
spec:
  replicas: 4
  selector:
    matchLabels:
      run: hello-node
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: hello-node
    spec:
      containers:
      - image: gcr.io/PROJECT_ID/hello-node:v1 ## Update this line ##
        ## => gcr.io/PROJECT_ID/hello-node:v1에서 gcr.io/PROJECT_ID/hello-node:v2
        imagePullPolicy: IfNotPresent
        name: hello-node
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      securityContext: {}
      terminationGracePeriodSeconds: 30
```



수정이 완료되면 다음과 같은 메세지를 확인할 수 있다.

```bash
$ deployment "hello-node" edited
```



이제 아래 명령을 이용하여 새로운 이미지로 배포를 진행한다.

```bash
$ kubectl get deployments
>>
NAME         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-node   4         4         4            4           1h
```

​    

## Kubernetes Graphics Dashboard

kubernetes는 최신버전에서 그래픽 UI가 도입되었다.

다음과 같은 명령어를 이용하면 kubernetes 클러스터 대시보드에 접근할 수 있다.

```bash
$ gcloud container clusters get-credentials hello-world \
    --zone us-central1-f --project <PROJECT_ID>
$ kubectl proxy --port 8081
```



API 엔드포인트로 이동한 후에, URL에서 마지막 `/?authuser=0`을 삭제 후 끝에 `/ui`를 붙여서 접속하면 다음과 같은 화면을 볼 수 있다.

![69f247ac054ce2a5](https://user-images.githubusercontent.com/13328380/51083548-77c14e80-175f-11e9-8bfb-61f77f417113.png)

​    

## Reference

[[1] 쿠버네티스란 무엇인가?](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/)

[[2] [IT 트렌드] 컨테이너 오케스트레이션(CONTAINER ORCHESTRATION](http://www.mantech.co.kr/container_orchestration/)

[[3] 초보를 위한 도커 안내서 - 도커란 무엇인가?](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)

[[4] AWS re:Invent 2016: From Monolithic to Microservices: Evolving Architecture Patterns in the Cloud (ARC305)](https://www.slideshare.net/AmazonWebServices/aws-reinvent-2016-from-monolithic-to-microservices-evolving-architecture-patterns-in-the-cloud-arc305)

[[5] The Death of Microservice Madness in 2018](https://dwmkerr.com/the-death-of-microservice-madness-in-2018/)

[[6] 쿠버네티스 #1 - 소개](http://bcho.tistory.com/1255)