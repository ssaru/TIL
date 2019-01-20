# Continuous Delivery with Jenkins in Kubernetes Engine - (KUBERNETES IN THE GOOGLE CLOUD 2주차)



이번 세션에서는 5강을 리뷰한다.

​    

## Overview

이번장은 Kubernetes Engine위에서 `Jenkins`를 이용해서 Continous Delivery(CD) pipeline을  학습한다. `Jenkins`는 공통의 Repository에서 빈번하게 코드를 병합하는 개발자들이 사용하는 자동화 서버다. 해당 강의에서 구축할 솔루션의 다이어그램은 아래와 같다.



![diagram](https://user-images.githubusercontent.com/13328380/51435686-7db7b200-1cc0-11e9-89d4-5293b94e65a4.png)



Kubernetes에서 젠킨스에서 돌리는 방법은 [다음](https://cloud.google.com/solutions/jenkins-on-kubernetes-engine)을 참고하자

​    

## What you will do

- Kubernetes 엔진 클러스터에 Jenkins 응용 프로그램 제공
- HELM을 사용해서 Jenkins 어플리케이션 설정
- Jenkins 어플리케이션 기능 탐색
- Jenkins 파이프라인 만들기 및 실행

​    

## What is Kubernetes Engine

Kubernetes엔진은 GCP에서 제공하는 컨테이너를 위한 클러스터 매니저이면서 컨테이너 오케스트레이션 시스템이다.

​    

## What is Jenkins

[Jenkins](https://jenkins.io/)은 빌드 / 테스트 / 배포 파이프라인을 유연하게 조율해주는 오픈소스 자동화 서버이다. Jenkins는 개발자가 지속적인 배포에 대한 오버헤드에 대한 걱정없이 프로젝트를 빠르게 반복할 수 있게 도와준다.

​    

## What is Continuous Delivery / Continous Deployments?

Kubernetes Engine에 CD pipeline을 Jenkins 이용해 구축하는 것은 VM 기반의 deployment를 진행할 때보다 중요한 이점이 있다.

컨테이너를 빌드 프로세스에서 사용하는 경우에 하나의 가상 호스트가 여러 운영체제에서 작업을 실행할 수 있다. Kubernetes Engine은 일시적인 build executor를 제공한다. 이 build executor는 빌드가 활발할 때만 사용되므로 다른 작업을 위한 클러스터 리소스가 남는다. 또 다른 일시적인 build excutor의 장점은 몇초만에 실행되는 빠른 속도가 있다.



​    

## Provisioning Jenkins

### Creating a Kubernetes cluster

다음 명령어를 이용해서 Kubernetes cluster를 provisioninig한다.

```bash
$ gcloud container clusters create jenkins-cd \
--num-nodes 2 \
--machine-type n1-standard-2 \
--scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform"
>

```



다음 명령을 이용해서 cluster가 작동하는지 확인한다

```bash
$ gcloud container clusters list
>

```



이제 아래 명령어를 이용해서 cluster의 credential을 획득한다.

```bash
$ gcloud container clusters get-credentials jenkins-cd
>

```



Kunerntes Engine은 위에서 획득한 credential을 이용해서 새롭게 provision된 cluster에 접근할 수 있다.

아래 명령어를 이용해서 연결이 되는지 확인하자

```bash
$ kubectl cluster-info
>

```

​    

 ## Install Helm

이번 강의에서는 Helm을 사용해서 Chart Repository에서 Jenkins를 설치할 것이다.  Helm은 Kubernetes 어플리케이션의 배포 및 설정을 쉽게해주는 패키지 매니저다. Jenkins를 한번 설치하면 우리는 우리만의CI/CD 파이프라인을 설정할 수 있다.



1. Helm을 다운로드하고, 설치

   ```bash
   $ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz
   $ tar zxfv helm-v2.9.1-linux-amd64.tar.gz
   $ cp linux-amd64/helm .
   ```

2. 클러스에서 Jenkins권한을 주기 위해서 스스로를 RBAC 클러스터 관리자로 추가한다.

   ```bash
   $ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
   ```

3. Helm의 서버사이드인 Gient Thiller는 클러스터안에서 cluster-admin 역활을 한다.

   ```bash
   $ kubectl create serviceaccount tiller --namespace kube-system
   $ kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
   ```



4. Helm을 초기화한다.

   ```bash
   $ ./helm init --service-account=tiller
   $ ./helm update
   ```

5. Helm이 작동하는지 확인한다.

   ```bash
   $ ./helm version
   >
   Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
   Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
   ```



​    

## Configure and Install Jenkins

1. Helm CLI를 사용해서 configuration setting과 함께 chart를 배포한다.

   ```bash
   $ ./helm install -n cd stable/jenkins -f jenkins/values.yaml --version 0.16.6 --wait
   ```

2. 위의 명령어를 제대로 입력했다면, Jenkins Pods가 작동하고 있을 것이다.

   ```bash
   $ kubectl get pods
   >
   NAME                          READY     STATUS    RESTARTS   AGE
   cd-jenkins-7c786475dd-vbhg4   1/1       Running   0          1m
   ```

3. 아래의 명령어를 이용해서 Jenkins UI로 port-forwarding을 해준다.

   ```bash
   $ export POD_NAME=$(kubectl get pods -l "component=cd-jenkins-master" -o jsonpath="{.items[0].metadata.name}")
   kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
   ```

4. Jenkins 서비스가 만들어졌는지 확인한다.

   ```bash
   $ kubectl get svc
   >
   NAME               CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
   cd-jenkins         10.35.249.67   <none>        8080/TCP    3h
   cd-jenkins-agent   10.35.248.1    <none>        50000/TCP   3h
   kubernetes         10.35.240.1    <none>        443/TCP     9h
   ```



​    

## Connect to Jenkins

1. Jenkins chart는 자동으로 admin password를 생성한다. 다음 명령어를 이용해서 Jenkins UI를 실행한다.

   ```bash
   $ printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
   ```

2. 8080 포트로 웹을 열어서 Jenkins UI를 확인한다.

3. 이제 `admin`이라는 사용자 이름과 자동으로 생성된 password를 이용해서 로그인 할 수 있다.



이것으로 Kubernetes Engin에 jenkins를 설치하고 셋팅했다. 이제 자동 CI/CD 파이프라인을 만들어 보자

​     

## Understanding Application

Go로 작성된 `gcem`이라는 어플리케이션을 CI/CD 파이프라인을 통해서 배포할 것이다.  만약 해당 어플리케이션을 실행시켜면 우리는 다음과 같은 화면을 확인할 수 있다.



![71ea855fa570b5dd7635206d9cd86b91e98a7d148b1db0a615988b9fc5a60bdf](https://user-images.githubusercontent.com/13328380/51435922-39c7ab80-1cc6-11e9-8e57-2dffd800368f.png)



해당 어플리케이션은 2개의 operation mode에 의해서 작은 웹 서비스를 따라한다.

-  **backend mode**: `gceme`은 8080포트로 request를 받고 Compute Engine instance의 metadata를  JSON format으로 돌려준다.
-  **frontend mode**: `gceme` **backend**의 `gceme` service에게 queries를 보내고 JSON 결과를 UI에 렌더링 한다.



![3f54f9241596a6b03888b7ff079f8ee9ab3ba2d2c4db960175ee7b8336704b3e](https://user-images.githubusercontent.com/13328380/51435953-a642aa80-1cc6-11e9-877e-296413cbc721.png)

​    

## Deploying the Application

우리는 어플리케이션을 다른 2개의 시스템에 배포할 것이다.

- Production : 사용자가 액세스하는 라이브 사이트
- Canary : 사용자 트래픽 중 일부만 수신하는 용량이 작은 사이트. 사용자에게 실제 배포전에 서비스의 유효성을 검증하는 용도



1. 논리적으로 배포를 분리하는 Kuberntes namespace를 생성한다.  

   ```bash
   $ kubectl create ns production
   >
   
   ```

2. canary deployment를 만든다.

   ```bash
   $ kubectl apply -f k8s/production -n production
   $ kubectl apply -f k8s/canary -n production
   $ kubectl apply -f k8s/services -n production
   ```

3. frontend의 제품 환경 scale을 확장한다.

   ```bash
   $ kubectl scale deployment gceme-frontend-production -n production --replicas 4
   ```

4. 5개의 pod들에는 4개는 기존 제품이 1개는 canary deployment가 있는 것을 확인하자

   ```bash
   $ kubectl get pods -n production -l app=gceme -l role=frontend
   ```

5. 2개의 pod에는 backend가 있으며 backend의 구성은 1개는 production, 1개는 canary deployment라는 것을 확인하자

   ```bash
   $ kubectl get pods -n production -l app=gceme -l role=backend
   ```

6. frontend의 외부 ip를 확인하자

   ```bash
   $ kubectl get service gceme-frontend -n production
   >
   NAME            TYPE          CLUSTER-IP     EXTERNAL-IP     PORT(S)  AGE
   gceme-frontend  LoadBalancer  10.79.241.131  104.196.110.46  80/TCP   5h
   ```

7. External IP에 접속하면 다음과 같은 화면을 확인할 수 있다.



   ![71ea855fa570b5dd7635206d9cd86b91e98a7d148b1db0a615988b9fc5a60bdf](https://user-images.githubusercontent.com/13328380/51435922-39c7ab80-1cc6-11e9-8e57-2dffd800368f.png)

8. 나중에 사용할 frontend service load balancer IP를 환경변수로 저장하자

   ```bash
   $ export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
   ```

9. frontend의 External IP를 이용해서 frontend service의 버전을 확인해보자

   ```bash
   $ curl http://$FRONTEND_SERVICE_IP/version
   ```


​    

## Creating the Jenkins Pipeline

### Creating a repository to host the sample app source code

1. `gceme` 예제 어플리케이션을 Cloud Source Repository에 Push 한다.

   ```bash
   $ gcloud alpha source repos create default
   ```

2. 예제 어플리케이션의 디렉토리를 git local repository로 초기화한다.

   ```bash
   $ git init 
   ```

3. 아래 명령어를 실행한다.

   ```bash
   $ git config credential.helper gcloud.sh
   $ git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/default
   ```

4. 몇가지 git 설정을 진행한다.

   ```bash
   $ git config --global user.email "[EMAIL_ADDRESS]"
   $ git config --global user.name "[USERNAME]"
   ```

5. 코드를 푸쉬한다.

   ```bash
   $ git add .
   $ git commit -m "Initial commit"
   $ git push origin master
   ```


​    

### Adding your service account credentials

jenkins가 코드 저장소에 접근할 수 있도록, credentials를 설정한다. jenkins는 여러분의 코드 저장소에서 코드를 다운로드 받기 위해서 해당 credentials를 사용한다.



1. jenkins UI에서 왼쪽 네이게이션바에서 **Credentials**을 클릭한다.

2. Jenkins를 클릭한다.

   ![4a2a8b96e06de77d5149a2d69d4666dbc2d14552425416c8354673eef692c9fa](https://user-images.githubusercontent.com/13328380/51436103-e3f50280-1cc9-11e9-979e-ac27272e1105.png)

3. Global credentials를 클릭한다.

4. Add credentials를 클릭한다.

5. drop down식으로 **Google Service Account from metadata**를 선택하고 **OK**를 클릭한다.



#### Creating the jenkins job

왼쪽의 네이게이션 바를 이용해서 pipeline job을 설정한다.



1. 왼쪽 네비게이션바에서 Jenkins > New item을 클릭한다.

   ![29707d831f01efb176ed7020bba6e05e57b41ef9e8111c253dff5fd96615a903](https://user-images.githubusercontent.com/13328380/51436122-3cc49b00-1cca-11e9-8ddf-9badd3891207.png)

2. Project 이름을 **sample-app**으로 하고, **Multibranch pipeline**옵션을 선택한 후 **OK**버튼을 클릭한다.

3. 다음 페이지에서 **Branch Source**에서 **Add source**를 클릭하고 **git**을 클릭한다.

4. sample app의 **HTTPS clone URL**을 복사 붙여넣기한다. 예시는 다음과 같다.

   ```bash
   $ https://source.developers.google.com/p/[PROJECT_ID]/r/default
   ```

5. 이전 스텝에서 생성한 credential 이름을 선택한다.

6. **Scan Multibranch Pipeline Triggers**에서 **Periodically if not otherwise run**에 체크하고 실행 간격은 1분으로 설정한다.

7. job configuration은 다음과 같이 될 것이다.

   ![1247bec5a13e4698ad6511bf1627f0f6398dc008022cc9a7536d6676af0a0dca](https://user-images.githubusercontent.com/13328380/51436160-17845c80-1ccb-11e9-84f5-fd8f7dfc02b8.png)



   ![8fb86e97f5d4fae9da858222769c3ae3efb940909fb0c1ecab4011cabca2866c](https://user-images.githubusercontent.com/13328380/51436159-17845c80-1ccb-11e9-8bc5-dbed671b0902.png)



   ![326c0b9796605a9065ac3fbdf9fafded4841ae756eaa5a6632f981a1f778f583](https://user-images.githubusercontent.com/13328380/51436158-16ebc600-1ccb-11e9-85ff-a60afc625d4c.png)

8. 다른 옵션은 그대로 두고, **Save**버튼을 누릅니다.

   해당 작업이 완료가 되면, **Branch indexing**이라는 작업이 실행된다. 해당 메타 작업은 저장소의 branch를 식별하고, 변경사항이 존재하는 branch에서 일어나지 않았는지 검사한다. 왼쪽 상단의 sample-app을 클릭하면 master 작업 표가 나타난다.

   > 처음 실행했을 때, Master job은 몇가지 코드를 수정하지 않는 한 실패할 수 있습니다.



이제 Jenkins의 파이프라인 설정을 완료했습니다.

이제 다음 섹션에서 CI를 위한 개발 환경을 만들겠습니다.

​    

## Creating the Development Environment

Development branches는 live site로 코드를 제출하기 전에 그 코드들을 테스트하는 branch다.  실습하는 환경은 실제 응용 프로그램의 축소버전이지만, 실제 환경은 동일한 메커니즘을 이용해서 배포해야한다.



#### Creating a development branch

1. 새로운 feature branch를 생성한다.

   ```bash
   $ git checkout -b new-feature
   ```

​    

#### Modifying the pipeline definition

`Jenkinsfile`은  [Jenkins Pipeline Groovy syntax](https://jenkins.io/doc/book/pipeline/syntax/)를 사용해서 작성된 pipeline 정의다. `Jenkinsfile`을 사용하면 전체 빌드 파이프 라인을 소스 코드와 함께 존재하는 단일 파일로 표현할 수 있다. 파이프 라인은 병렬 처리와 같은 강력한 기능을 지원한다.



`Jenkinsfile`을 다음과 같이 수정한다.

``` bash
def project = 'REPLACE_WITH_YOUR_PROJECT_ID'
def appName = 'gceme'
def feSvcName = "${appName}-frontend"
def imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
```

​    

#### Modify the site

1. `gceme`카드를 blue에서 orange색으로 변경한다.

   ```bash
   $ vim html.go
   >
   <div class="card orange">
   ```

2. `main.go`에서 버전을 변경한다.

   ```bash
   $ vim main.go
   >
   const version string = "2.0.0"
   ```



​    

### Kick off Deplyment

변경사항을 push한다.

```bash
$ git add Jenkinsfile html.go main.go
$ git commit -m "Version 2.0.0"
$ git push origin new-feature
```



이렇게 push하는 것은 개발환경의 빌드를 시작하게 된다. 변경사항이 git 저장소로 push된 이후 jenkins사이트에서 확인해보면 빌드가 시작된 것을 확인할 수 있다.

![1f1801643882b778f356f0983dfb9f57acbdedc4fc820538197ac2e43d2a299f](https://user-images.githubusercontent.com/13328380/51436307-0f79ec00-1cce-11e9-9761-05f830bc2316.png)







![dd22f64d13813a01cf0005d12e12c23049b22788742c9199007d9a36b2ec4e84](https://user-images.githubusercontent.com/13328380/51436306-0ee15580-1cce-11e9-92eb-59d93c71b8ca.png)



​    

## Deploying a Canary Release

변경사항을 canary deploy로 확인해볼 수 있다. 아래 명령을 순차적으로 실행하면 canary release가 되는 것을 확인할 수 있다.



```bash
$ git checkout -b canary
$ git push origin canary
$ export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
$ while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
```

​    

## Deploying to production

canary release를 통해서 production에 대한 검증이 끝났다고 가정하고, 이제 본 제품에 본격적으로 배포한다고 가정하자

```bash
$ git checkout master
$ git merge canary
$ git push origin master
$ export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
$ while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done

>
gcpstaging9854_student@qwiklabs-gcp-df93aba9e6ea114a:~/continuous-deployment-on-kubernetes/sample-app$ while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
2.0.0
2.0.0
2.0.0
2.0.0
2.0.0
2.0.0

```



이제 canary에서 본격적으로 제품에 release된 것을 확인할 수 있고 , external IP를 확인해서 직접 사이트를 확인해보자

```bash
$ kubectl get service gceme-frontend -n production
```



![9c52651ed64f23ff07bed08d4864660b2a9190c0f295fb6aff6d34c515ed50a7](https://user-images.githubusercontent.com/13328380/51436357-dd1cbe80-1cce-11e9-85d1-5a1d0b5722e0.png)

