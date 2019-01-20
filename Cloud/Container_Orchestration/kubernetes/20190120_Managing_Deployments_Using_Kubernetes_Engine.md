# Managing Deployments Using Kubernetes Engine - (KUBERNETES IN THE GOOGLE CLOUD 2주차)



이번 세션에서는 4강을 리뷰한다.

​    

## 배포 소개

이기종 배포란 서로 다른 2개 이상의 인프라 환경 또는 지역을 연결해 기술 또는 운영 상의 특정한 요구를 해결

이기종 배포는 배포 세부사항에 따라 다음과 같이 불린다.

- 하이브리드 배포
- 멀티 클라우드 배포
- 공개-비공개 배포



배포가 단일환경 혹은 지역으로 제한된 경우 다음과 같은 챌린지가 존재한다.

- 리소스 한도 도달
  - 단일환경, 내부환경에서 컴퓨팅, 네트워킹, 저장소 리소스가 프로턱션 요구사항을 충족하지 못함
- 제한적 지리적 범위
  - 멀리 떠어진 사용자들이 하나의 배포에 접근해야하므로, 트래픽에서 중앙이 되는 쪽으로 배포를 이동시켜야할 수 있다.
- 제한된 가용성
  - 웹 규모의 트래픽의 경우 어플리케이션의 내 결함성 및 탄력성 유지라는 어려움이 발생
- 제공업체 종속
  - 제공업체 수준의 플랫폼 및 인프라 추상화가 어플리케이션 포팅에 방해가 됨
- 경직된 리소스
  - 리소스가 특정한 컴퓨팅, 저장소 또는 네트워킹 제공 서비스로 제한된다.



이기종 배포가 위의 어려움을 해결하는데 도움이 될 수 있다. 이기종 배포는 프로그래밍 방식의 확정적인 프로세스 및 절차를 사용해 배포를 설계해야한다. 효ㅘ적인 배포 프로세스는 반복 가능해야하며 관리, 프로비저닝, 구성, 유지보수에 있어 검증된 방식을 사용해야한다.



이기종 배포에 대한 일반적인 3가지 시나리오로는 멀티 클라우드 배포, 프론트엔드 내부 데이터, 지속적 통합/지속적 배포(CI/CD) 프로세스가 존재



해당 강의에서는 Kubernetes와 이기종 배포 구현을 위한 그 밖의 인프라 리소스를 사용해 설계된 접근법과 이기종 배포에 대한 일반적인 사용사례에 대해서 다뤄보자

​    

## Deployment Object; 배포 객체

### Deployment Object란?

Deployment Object라는 것은 무엇일까?

먼저 Deployment Object에 대해서 알아보기 전에 Kubernetes에서 Object라는 것이 무엇인지부터 확인을 해보자.



**Object**란 `k8s의 상태를 나타내는 엔티티`를 의미한다. 따라서 Object는 K8s API의 Endpoint로써 작동한다. 

각각의 Object는 `Spec`과 `Status`라는 필드를 갖게되는데 K8s는 Object의 **Spec을 통해 사용자가 기대하는 상태(Desired State)가 무엇인지 알 수 있고**, 기대되는 값에 대비한 **현재의 상태를 Object의 Status를 통해 알 수 있다.**

이 때 **Object의 Status를 갱신하고, Object를 Spec에 정의된 상태로 지속적으로 변화시키는 주체를 Controller**라고 한다.

>    Object에는 Pod, Service, Volume, Namespace등이 포함되고
>    Controller에는 ReplicaSet(구 ReplicationController), Deployment, StatefulSets, DaemonSet, CronJob 등이 포함된다. 



자 이제 **Object**라는게 무엇인지 알았으니, **Deployment Object**가 어떤 역활을 하는지 알아보자.

> Deployment Object는 애플리케이션의 배포/삭제, scale out 의 역할을 한다.  Deployment를 생성하면 Deployment가 Pod과 ReplicaSets를 함께 생성한다. Pod에 containerized 애플리케이션들이 포함되고, Pod가 생성되면서 애플리케이션이 배포되는 원리이다. ReplicaSets는 replica 수를 지속 모니터링하고, 유지시켜준다. 만약 Pod이 삭제되어 replica 수와 맞지 않게 되면 ReplicaSets가 동작하여 지정된 replica 수가 되도록 Pod을 생성한다.



자 이제 대략적인 이해는 했으니 이제 사용을 해보자.



아래와 같은 명령어로 Deployment object에 대해서 자세히 확인해볼 수 있다.

```bash
$ kubectl explain deployment
>
KIND:     Deployment
VERSION:  extensions/v1beta1
DESCRIPTION:
     DEPRECATED - This group version of Deployment is deprecated by
     apps/v1beta2/Deployment. See the release notes for more information.
     Deployment enables declarative updates for Pods and ReplicaSets.
FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources
   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds
   metadata     <Object>
     Standard object metadata.
   spec <Object>
     Specification of the desired behavior of the Deployment.
   status       <Object>
     Most recently observed status of the Deployment.
```



만약 모든 필드값들을 확인하고 싶다면 `--recursive`을 주게되면 모든 필드를 확인할 수 있다.

```bash
$ kubectl explain deployment --recursive
>
         controller     <boolean>
         kind   <string>
         name   <string>
         uid    <string>
      resourceVersion   <string>
      selfLink  <string>
      uid       <string>
   spec <Object>
      minReadySeconds   <integer>
      paused    <boolean>
      progressDeadlineSeconds   <integer>
      replicas  <integer>
      revisionHistoryLimit      <integer>
      rollbackTo        <Object>
         revision       <integer>
      selector  <Object>
...
```



만약 특정 필드값에 대해서만 확인하고 싶다면 아래의 명령어를 활용할 수 있다.

```bash
$ kubectl explain deployment.metadata.name
>
KIND:     Deployment
VERSION:  extensions/v1beta1
FIELD:    name <string>
DESCRIPTION:
     Name must be unique within a namespace. Is required when creating
     resources, although some resources may allow a client to request the
     generation of an appropriate name automatically. Name is primarily intended
     for creation idempotence and configuration definition. Cannot be updated.
     More info: http://kubernetes.io/docs/user-guide/identifiers#names
```

​    

### 배포만들기

퀵랩에서 `*.yaml`파일로 제공하는  deployment파일을 열어보면 다음과 같이 구성되어있다.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: "10Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
```



아래와 같은 명령어를 이용하여 배포 객체를 만든다.

```bash
$ kubectl create -f deployments/auth.yaml
>
deployment.extensions "auth" created
```



배포를 만든 이후에는 배포가 잘 만들어졌는지, 그리고 replicaset, pod가 잘 만들어졌는지 확인할 수 있다.

```bash
$ kubectl get deployments
>
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
auth      1         1         1            1           8m
```



```bash
$ kubectl get replicasets
>
NAME              DESIRED   CURRENT   READY     AGE
auth-6ccd6fd58c   1         1         1         8m
```



```bash
$ kubectl get pods
>
NAME                    READY     STATUS    RESTARTS   AGE
auth-6ccd6fd58c-snfrg   1/1       Running   0          9m
```

​    

### 서비스 만들기

서비스는 이전에 살펴보았으니, 따로 살펴보지 않겠다.

```bash
$ kubectl create -f services/auth.yaml
>
service "auth" created
```



인증 배포/서비스를 만들었으니, hello / frontend 배포 및 서비스를 똑같이 만들어준다.

```bash
$ kubectl create -f deployments/hello.yaml
>
deployment.extensions "hello" created
```



```bash
$ kubectl create -f services/hello.yaml
>
service "hello" created
```



```bash
$ kubectl create secret generic tls-certs --from-file tls/
>
secret "tls-certs" created

$ kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
>
configmap "nginx-frontend-conf" created

$ kubectl create -f deployments/frontend.yaml
>
deployment.extensions "frontend" created

$ kubectl create -f services/frontend.yaml
>
service "frontend" created
```



이제 외부 IP를 선택한 후, curl을 이용하여 프론트엔드와 상호작용을 한다.

```bash
$ kubectl get services frontend
>
NAME       TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)         AGE
frontend   LoadBalancer   10.51.241.22   35.224.166.168   443:31643/TCP   1m 

$ curl -ks https://35.224.166.168
>
{"message":"Hello"}
```



`kubectl`의 출력 템플릿 기능을 사용하면, curl을 한줄로 줄일 수 있다.

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
```



​    

### 배포확장

배포를 성공적으로 했다면, 이제 배포를 확장해보자.

`spec.replicas` 필드를 업데이트하여 배포 확장을 할 수 있다.



먼저 `spec.replicas`필드를 확인하자

```bash
$ kubectl explain deployment.spec.replicas
>
KIND:     Deployment
VERSION:  extensions/v1beta1

FIELD:    replicas <integer>

DESCRIPTION:
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.
```



`kubectl scale` 명령을 사용하면, replica 필드값을 가장 쉽게 업데이트 할 수 있다.

```bash
$ kubectl scale deployment hello --replicas=5
>
deployment.extensions "hello" scaled
```



배포가 업데이트 되고 나면 kubernetes는 자동으로 관련 replicaset를 업데이트하고 새 포드를 시작해 총 포드 수를 5개로 만든다.



정말로 포드가 5개가 되었는지 다음 명령어로 확인할 수 있다.

```bash
$ kubectl get pods | grep hello- | wc -l 
>
5
```



이번에는 replica 필드를 축소해보자

```bash
$ kubectl scale deployment hello --replicas=3
>
deployment.extensions "hello" scaled
```



포드 수를 다시 한 번 확인해보자

```bash
$ kubectl get pods | grep hello- | wc -l
>
3
```

​    

### 지속적 업데이트

배포에서는 지속적 업데이트 매커니즘을 통해서 이미지가 새 버전으로 업데이트되도록 지원한다. 배포가 새 버전으로 업데이트되면서 새로운 복제본 세트가 생성되고, 기존 복제본 세트의 복제본이 줄어들면서 새 복제본 수가 점점 증가한다.

![f8e46de0cb7e111](https://user-images.githubusercontent.com/13328380/51435356-c4a1a980-1cb8-11e9-832e-56b20e429e25.png)



#### 지속적 업데이트 트리거

배포를 업데이트는 아래의 명령어를 통해서 실행할 수 있다.

```bash
$ kubectl edit deployment hello
```



위의 명령어를 실행하면 텍스트 편집기가 뜨는데, 여기서 `image`를 변경하고 종료하면 다음과 같은 화면이 뜬다.

```bash
...
containers:
- name: hello
  image: kelseyhightower/hello:2.0.0
...

>
deployment.extensions "hello" edited
```



이렇게 배포를 업데이트하면, 배포가 클러스터에 저장이 되고, Kubenetes에서 지속적 업데이트를 시작한다.

kubenetes에서 만든 새 복제본 세트를 조회하자

```bash
$ kubectl get replicaset
>
NAME                  DESIRED   CURRENT   READY     AGE
auth-6ccd6fd58c       1         1         1         13m
frontend-5f79fbf477   1         1         1         13m
hello-5d479547f       3         3         3         2m
hello-c7f8d5464       0         0         0         13m
```



롤 아웃 내역에서 새 항목을 확인할 수 있다.

```bash
$ kubectl rollout history deployment/hello
>
deployments "hello"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

​    

#### 지속적 업데이트 일시중지

실행 중인 롤아웃에서 문제가 발견되면, 일시중지시켜 업데이트를 중단한다.

업데이트 일시중지는 아래 명령어로 수행할 수 있다.

```bash
$ kubectl rollout pause deployment/hello
>
deployment.apps "hello" paused
```



롤 아웃 상태를 확인해보자

```bash
$ kubectl rollout status deployment/hello
>
deployment "hello" successfully rolled out
```

*나의 경우에는 명령어를 너무 늦게 입력하는 바람에 이미 roll out이 끝난 상황이었다.*



롤 아웃 상태는 Pod에서도 확인할 수 있다.

```bash
$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

>

auth-6ccd6fd58c-fpt42           kelseyhightower/auth:1.0.0
frontend-5f79fbf477-z4qgb               nginx:1.9.14
hello-5d479547f-8btxn           kelseyhightower/hello:2.0.0
hello-5d479547f-l7gqg           kelseyhightower/hello:2.0.0
hello-5d479547f-mhcck           kelseyhightower/hello:2.0.0
```

​    

#### 지속적 업데이트 재개

롤 아웃이 일시 정지된 경우 포드 중 일부만 새버전이고 나머지는 기존 버전으로 유지된다.

`resume`명령어를 이용하여 업데이트를 재개할 수 있다.

```bash
$ kubectl rollout resume deployment/hello
>
deployment "hello" successfully rolled out
```

​    

#### 업데이트 롤백

새 버전에서 버그가 발견됬다고 가정했을 때, 우리는 시스템을 롤백하고 싶을 것이다.

롤백은 `rollout` 명령어를 이용하여 이전 버전으로 롤백할 수 있다.

```bash
$ kubectl rollout undo deployment/hello
>
deployment.apps "hello"
```



내역에서 롤백을 확인해보자

```bash
$ kubectl rollout history deployment/hello
>
deployments "hello"
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```



마지막으로 모든 포드가 롤백되었는지 확인하자

```bash
$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
>
auth-6ccd6fd58c-fpt42           kelseyhightower/auth:1.0.0
frontend-5f79fbf477-z4qgb               nginx:1.9.14
hello-c7f8d5464-2jvkp           kelseyhightower/hello:1.0.0
hello-c7f8d5464-4x8g2           kelseyhightower/hello:1.0.0
hello-c7f8d5464-75ls6           kelseyhightower/hello:1.0.0
```

​    

### 카나리 배포

카나리 배포는 새 버전의 별도 배포와 안정적인 일반 배포 및 카나리 배포를 타겟팅하는 서비스로 이루어진다.

![f9a05e6929fac02a](https://user-images.githubusercontent.com/13328380/51435411-3f1ef900-1cba-11e9-825b-92eef9290927.png)



먼저 새 버전에 해당하는 새로운 카나리 배포를 만든다.

```bash
$ cat deployments/hello-canary.yaml
>
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-canary
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello
        track: canary
        # Use ver 1.0.0 so it matches version on service selector
        version: 1.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
...
```



만약 버전을 `hello:1.0.0`으로 업데이트 해야한다고 가정하자



카나리 배포를 만든다.

```bash
$ kubectl create -f deployments/hello-canary.yaml
>
deployment.extensions "hello-canary" created
```



카나리 배포를 만들면, 배포가 2개(`hello`, `hello-canary`)가 생긴다.

이를 확인해보자

```bash
$ kubectl get deployments
>
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
auth           1         1         1            1           23m
frontend       1         1         1            1           22m
hello          3         3         3            3           23m
hello-canary   1         1         1            1           40s
```

`hello` 서비스에서 선택기가 `app:hello` 선택기를 사용해 프로덕션 배포 및 카나리 배포 **모두**의 포드를 일치시킨다.

하지만 카나리 배포에는 포드 수가 더 적으므로 일부 사용자에게만 표시된다.

​    

#### 카나리 배포 확인

curl 요청에서 `hello` 버전을 확인할 수 있다.

curl 요청을 5번 정도한 결과를 아래에 적어보았다. 결과에서 몇개의 return값이 결과가 다른 것을 확인할 수 있다.

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

>
{"version":"2.0.0"}
{"version":"2.0.0"}
{"version":"1.0.0"}
{"version":"1.0.0"}
{"version":"1.0.0"}
```



이를 여러 차례 실행하면 요청 중 일부를 hello 1.0.0에서 사용하고 작은 하위 집합(1/4 = 25%)을 2.0.0에서 사용하는 것을 볼 수 있다.



#### 프로덕션 카나리 배포 - 세션 연관성

만약에 한 사용자가 어떤 제품에 접속하여 서비스를 이용하고있는데, 어떤 버튼같은 요청을 할 때마다 시스템 버전이 변경된다면 혼란스러울 수 있다.

> 특히 UI의 변경이 각 버전마다 다른 경우 사용자는 요청해서 페이지를 넘어갈 때마다 서로 다른 버전의 UI/UX를 접하게 되기 때문에 굉장히 불편할 수 있다.

이럴 때, 우리는 한 사용자에 한해서 두 배포 중 하나만 계속 사용하도록 하고 싶을 것이다. 이는 세션 연관성을 사용한 서비스를 만들면 해결할 수 있다. 이렇게 설정한 경우 한 사용자가 항상 동일 버전만 사용하게 됩니다. 아래 예를 살펴보면 서비스는 전과 동일하며 새 `sessionAffinity` 필드가 추가되었고 ClientIP로 설정되어 있다. IP 주소가 동일한 모든 클라이언트의 요청이 동일한 버전의 `hello` 애플리케이션으로 전송된다.



```bas
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  sessionAffinity: ClientIP
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
```

​    

 #### 정리

카나리 배포가 작동함을 확인했으니 이제 카나리 배포와 함께 만든 서비스를 삭제해보자.

```bash
$ kubectl delete deployment hello-canary
>
deployment.extensions "hello-canary" deleted
```

​    

### Blue-Green 배포

지속적 업데이트는 최소한의 오버헤드, 성능 영향, 다운타임으로 천천히 애플리케이션을 배포할 수 있다는 점에서 이상적이다. 완전히 배포된 후에는 새 버전을 가리키도록 부하 분산기를 수정하는 편이 유용하다. 이러한 경우 Blue-Green 배포가 효과적이다.



![f7556b63b780a36a](https://user-images.githubusercontent.com/13328380/51435499-b739ee80-1cbb-11e9-863c-b2a58afb9e4e.png)

Kubernetes에서는 기존 ‘Blue' 버전과 새로운 ‘Green' 버전의 배포를 각각 하나씩 생성하여 이를 구현합니다. ‘Blue' 버전에는 기존 `hello` 배포를 사용한다. 이 배포에는 라우터로 기능하는 서비스를 통해 액세스합니다. 새로운 ‘Green' 버전이 실행되면 서비스를 업데이트하여 이 버전을 사용하도록 전환합니다.

> Blue-Green 배포의 단점은 애플리케이션을 호스트하려면 클러스터의 리소스가 2배 이상 필요하다는 것입니다. 두 버전의 애플리케이션을 동시에 배포하기 전에 클러스터에 충분한 리소스를 확보해야 한다.



#### 서비스

기존 `hello` 서비스를 사용하되 다음과 같은 선택기를 포함하도록 업데이트합니다. `app:hello`, `version: 1.0.0` 이 선택기는 기존 ‘Blue' 배포와 일치할 것이다. 하지만 다른 버전을 사용하므로 ‘Green' 배포와는 일치하지 않는다.

서비스를 이에 맞게 업데이트해보자.

```bash
$ kubectl apply -f services/hello-blue.yaml
>
service "hello" configured
```

​     

#### Blue-Green  배포를 사용한 업데이트

Blue-Green 배포 스타일을 위해 새 버전에 해당하는 새로운 ‘Green' 배포를 만든다. Green 배포는 버전 라벨 및 이미지 경로를 업데이트한다.



```bash
$ vim deployments/hello-green.yaml
>
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-green
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
```



Green 배포를 만들자.

```bash
$ kubectl create -f deployments/hello-green.yaml
>
deployment.extensions "hello-green" created
```



Green 배포를 만들었고 배포가 정상적으로 시작되면 현재 버전인 1.0.0을 아직 사용 중인지 할 수 있다.

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
>
{"version":"1.0.0"}
{"version":"1.0.0"}
{"version":"1.0.0"}
{"version":"1.0.0"}
{"version":"1.0.0"}
```



이제 새 버전을 가르키도록 서비스를 업데이트하자

```bash
$ kubectl apply -f services/hello-green.yaml
>
service "hello" configured
```



서비스가 업데이트되는 즉시, `Green`배포가 사용된다. 이를 확인해보자

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
>
{"version":"2.0.0"}
{"version":"2.0.0"}
{"version":"2.0.0"}
{"version":"2.0.0"}
{"version":"2.0.0"}
```

​    

#### Blue-Green 롤백

필요한 경우 동일한 방법을 통해 기존 버전으로 롤백할 수 있다.  `Blue` 배포가 실행 중인 상태에서 서비스를 기존 버전으로 업데이트한다.



```bash
$ kubectl apply -f services/hello-blue.yaml
>
service "hello" configured
```



이제 제대로 롤백되었는지 확인해보자

```bash
$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
>
{"version":"1.0.0"}
{"version":"1.0.0"}
{"version":"1.0.0"}
{"version":"1.0.0"}
{"version":"1.0.0"}
```



​    

## Reference

[[1]. kubernetes 01 - Pod](https://blog.2dal.com/2018/03/28/kubernetes-01-pod/)

[[2]. Kubernetes 활용(1/8) 시작하기](http://tech.cloudz-labs.io/posts/kubernetes/getting-start/)

