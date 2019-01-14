# Kubeflow



이전에 Kubeflow가 탄생하게 된 배경과, Kubeflow의 대략적인 기능을 나열한 게시글을 쓴적이 있다. 이를 다시 한번 확인해보고, 이번에는 Kubeflow가 가지고있는 기능들을 조금 더 상세하게 살펴보려고 한다.



## Previous

​    

### 머신러닝(Machine learning)에서 3가지 도전

#### Composability

- 배포는 모델 생성했다고 끝나는 것이 아니라, 엄청 복잡한 단계를 거친다.
- 이를 쉽게 조립할 수 있게 만들어줘야한다.

#### Portability

- 머신러닝의 배포 스텝들은 완전히 다른 시스템 안에 속하는 경우가 있으며, 로우-레벨 컴포넌트에 의존적이기 때문에 복잡도가 많이 커진다.
- 이렇게 복잡도가 커진 시스템을 다른 시스템에 쉽게 이식할 수 있게 만들어줘야한다.

#### Scalability

- 실제 제품단계의 레벨에서는 대규모 트래픽 및 동시접속자가 발생하기 때문에 상황에 맞춰서 대규모 트래픽을 감당할 수 있을 정도로 Scale out / in 하는 유연하고 수용력이 강해야한다.

​    

### Kubernetes

위의 3가지 도전은 머신러닝 제품에만 해당하는 이야기는 아니다. 단지 머신러닝 프로젝트에서는 더욱 복잡해지고 어려워진다는 것을 의미하고 이를 해결하고자 나온 솔루션 중에 하나가 Kubernetes가 된다. 이를 해결하기 위해서 Kubernets를 사용하곤 하는데, 데이터 사이언티스트에게는 Kubernetes를 사용하는데 있어서 장벽이 굉장히 높고 까다롭다.

​    

### Kubeflow

위의 문제들을 모두 해결하고자 나온것이 2017년 말에 시작된 Kubeflow 프로젝트이며 목적과 제공하는 기능은 다음과 같다.

#### 목적

Kubernetes 위에서 돌고있는 머신러닝 프로젝트의 개발 / 배포 그리고  구성/이식성/확장성을 관리하는 것을 쉽게 만들어주는 것

#### 기능

- 협업과 인터렉티브 학습을 위한 JupyterHub
- Custom resource에서의 Tensorflow 학습
- Tensorflow 서빙 배포
- GitOps를 위한 Argo CD 지원
- non-Tensorflow Python model과 복잡한 추론을 위한 SeldonCore
- 리버스 프록시(Reverse proxy)
- 어디에서나 쿠베르네티스가 작동할 수 있게 연결

​    

## Kubeflow의 기능 소개

기능 소개는 내가 관심있거나 알고있는 내용을 위주로 살펴보려고 한다. 각 기능 기능을 상세하게 살펴보면 너무 많은 툴들이 서로 엮여있어서 **대략적인 각 기능 소개**를 통해 Kubeflow를 어디에 쓰면 좋겠다라는 개인적인 생각을 전달해보려고 한다.



*(Kubeflow의 설치방법은 아직 확실하지 않아서 언급하지 않을 예정이다. 본 글에서는 MiniKube 기반으로 설치하여 테스트를 진행했다.)*



이번에 소개할 기능은 전체목록 중 다음과 같다.

*(아래 기능은 위의 목차와는 많이 다르지만 내가 생각했을 때, kubeflow의 강력한 특징이라고 볼 수 있는 기능들을 추려봤다.)*

- **JupyterHub**
- **Distributed Training 학습**
- **Katib (Hyperparameter Tuning)**
- **Model Serving**
- ksonnet
- Pipelines

​    

### JupyterHub



Kubeflow를 설치하면 다음과 같이 진행사항을 확인할 수가 있다. 

![screenshot from 2019-01-03 10-59-11](https://user-images.githubusercontent.com/13328380/50670762-12f44e80-1011-11e9-8045-ef9236acd6eb.png)



설치 후에 kubernets의 pods를 확인해보면 다음과 같이 여러 pods들이 활성화 되어있는것을 확인할 수 있다. 그 중 우리가 주목해야할 것은 `jupyter-0`, `jupyter-admin`이다.



![screenshot from 2019-01-03 10-53-08](https://user-images.githubusercontent.com/13328380/50670848-6d8daa80-1011-11e9-90e9-d3b084996bd0.png)



`jupyter`라는 이름을 갖는 포드들을 확인한 후에는 `127.0.0.1:8080`을 통해서 kubeflow UI에 접속할 수 있다.

UI화면에는

- Kubeflow docs
- JupyterHub
- TFjob Dashboard
- Katib Dashboard
- Pipeline dashboard

이 존재하는데, 우리는 여기서 `jupyterhub`를 확인해보겠다.



![screenshot from 2019-01-03 11-08-55](https://user-images.githubusercontent.com/13328380/50670877-97df6800-1011-11e9-84b5-22bb776372cf.png)    



`jupyterhub`를 클릭하면 로그인화면이 뜨는데, id에는 `admin` passwd에는 `스페이스바 한칸`을 치고 접속하면 된다.

접속하면 다음과 같은 화면을 볼 수 있다.

여러가지 옵션이 안보일 수도 있는데, 그럴 땐 `Toggle Advanced`를 눌러주면 저 아래화면이 활성화된다. Image는 Jupyter notebook을 위한 도커 이미지를 선택하는 곳이고, custom image 주소를 통해서 이를 받아올 수도 있다. 



그 외의 기능으로는 CPU / Memory / Workspace Volume / Data Volumes / Extra Resources(GPUs)를 선택할 수 있는데 이를 조정했을 때, 실질적으로 Kubernets가 어떻게 움직이는지는 확인하지 못했다. 다만 이런 기능이 있다는 정도로 소개할 수 있을 것 같다.



이를 설정해주고 `Spawn`을 눌러주면, 로딩화면이 뜨면서 Jupyter notebook을 구동한다.



![screenshot from 2019-01-03 11-09-41](https://user-images.githubusercontent.com/13328380/50670928-e5f46b80-1011-11e9-85e4-deb2d497951a.png)



구동이 완료되면 아래와 같은 화면을 볼 수가 있다.

맞다. 그냥 우리가 아는 그 `Jupyter notebook`이다.



![screenshot from 2019-01-03 10-53-54](https://user-images.githubusercontent.com/13328380/50670988-5b603c00-1012-11e9-8987-eae0e65cdada.png)



[예제파일](https://github.com/kubeflow/examples/tree/master/github_issue_summarization)을 돌려보면 잘 돈다. :)



![screenshot from 2019-01-03 10-53-37](https://user-images.githubusercontent.com/13328380/50671009-7468ed00-1012-11e9-84d8-580781d00dfd.png)



Kubeflow의 `JupyterHub`기능을 써보면서 느낀점은....
그냥... 하드웨어의 사양을 조정하면서 쓸 수 있는 Cloud `Jupyter Notebook`의 느낌이었다. 

​    

### Distributed Training 학습

위의 kubeflow의 소개글이 1년정도 지난 글이라, 현재 kubeflow 프로젝트를 확인하면 TensorFlow 뿐만 아니라

- Chainer
- MXNet
- Pytorch
- Tensorflow

의 학습을 지원한다. 

*기타로 MPI(Message Passing Interface)를 지원한다*



조금 눈의 띄는 점은 `TensorFlow` 부분에만 `TensorFlow Training(TFJob)`이라고 적혀있다. `(TFJob)`이라고 명칭이 별도로 붙은 이유는 `Tensorflow`에 특화되어있는건가?라는 느낌이 들었다. 음.. 구글이 만든 프로젝트니까?



Kubeflow에서 Training을 지원한다는 것에 특별한 점은 Kubernets를 이용해 distributed training이 가능해진다는 점이다. Kubeflow에서 distributed training을 지원하는 딥러닝 프레임워크는 위에서 언급한 저 프레임워크 모두를 지원한다는 점에서 굉장히 매력적일 수 있다.



여기서 유독 눈에 띄는건 TFJob이라고 따로 빠져나와있는 건데, 이 부분에 대해서 자세히 살펴보면 distributed TensorFlow job은 일반적으로 아래 프로세스 중 0개 이상을 포함함

- Chief : training & performing task 작업들(checkpoint save 등등)의 오케스트래이션을 위한 책임자
- Ps : parameter servers; 모델의 파라미터들을 위한 분산 데이터 저장
- Worker : 실제 모델을 training *(만약 worker가  0이면 이 역활을 Chief가 한다.)*
- Evaluator : 학습된 모델의 평가 메트릭을 계산

​    

### Katib (Hyperparameter Tuning)

[Google Vizier](https://static.googleusercontent.com/media/research.google.com/ja//pubs/archive/bcb15507f4b52991a0783013df4222240e942381.pdf)로부터 영감을 받은 프로젝트.

Katib은 scalable, flexible한 hyperparameter tuning framework이며, kubernets와 통합해서 사용할 수 있으며, 별도의 딥러닝 프레임워크(`Tensorflow`, `Pytorch`, `MXNet`)들과 의존관계가 없다.

Katib은 다음과 같은 개념이 존재

- Study
- Trial
- Suggestion



#### Study

혹시 오역이 있을까봐 원문을 첨부한다.

> Represents a single optimization run over a feasible space. Each Study contains a configuration describing the feasible space, as well as a set of Trials. It is assumed that objective function f(x) does not change in the course of a Study.

Study는 feasible space*(hyper parameter space를 이야기하는 듯 하다)*에서 하나의 최적화된 실행을 표현한다. 이 때 Study의 Objective function 자체는 변하지 않는 것을 가정한 상태에서 시도할 수 있는 feasible space를 포함한다.



#### Trial

> A Trial is a list of parameter values, x, that will lead to a single evaluation of f(x). A Trial can be “Completed”, which means that it has been evaluated and the objective value f(x) has been assigned to it, otherwise it is “Pending”. One trial corresponds to one job, and the job kind can be k8s Job, TFJob or PyTorchJob, which depends on the Study's worker kind.

Trial은 f(x)의 single evaluation으로 값을 가져올 수 있는 값 x들의 list이다. Trial은 `Completed`가 될 수 있는데, 이게 의미하는 것은 평가되고 objective value f(x)가 할당되었다는 것을 의미한다. 그 외의 값들은 `Pending`이다.  하나의 Trial은 하나의 job과 매칭된다. 여기서 job이라고 하면 k8s / TF / PyTorch Job같이 Study의 작업에 따른 Job을 의미한다.

*(Hyper Parameter Space에서 값들이 돌다가 값이 결정이 나면 `Completed` 아니면 `Pending`을 의미하는 것 같다.)*



#### Suggestion

parameter set을 수행하는 알고리즘을 제안한다. 현재 Katib이 제공하는 탐색 알고리즘은 다음과 같다.

- random
- grid
- hyperband
- bayesian optimization



Katib을 자세하게 살펴보지는 않았지만, Model의 Hyper Parameter를 최적의 값으로 찾아주는 Framework로 보인다.



### Model Serving

Model Serving 관련 kubeflow는 `Seldon-core` / `Istio` / `NVIDIA TensorRT Inference Server`를 지원한다. 이번글은 `Seldon-cre`와 `Istio`에 대해서 간략히 살펴본다.

​    

#### Seldon-core

Machine Learning Deployment는 많은 challenge를 가지고 있는데, `Seldon-core` 는 이를 해결하기위해 디자인됨

여기서 challenge란 다음과 같음

- 쉬운 배포

- 그래프 실행 및 scale out/in, rolling update

- health check 및 failed components의 recovery 보장

- ML을 위한 Infrastructure 최적화

- Latency 최적화

- APIs / RESTful / gRPC를 이용한 비지니스 어플리케이션 연결

- 복잡한 런타임 micro service graphs 구성

  - request 라우팅
  - 데이터 변환
  - 앙상블 결과

- 다양한 개발방식 허용

  - 동기식
  - 비동기식
  - 배치

- 버전 정리 및 감사 허용

  - CI
  - CD

- 모니터링 제공

  - 기본적인 메트릭 : Accuracy, request latency, throughput

- 복잡한 메트릭

  - concept drift
  - bias detection
  - outlier detection

- 최적화 허용

  - AB Tests
  - Multi-Armed Bandits

  

위의 challenge에 따른 `Seldon-core`의 high level goals는 다음과 같음

- 데이터 사이언티스트가 machine learning toolkit 및 programming language를 이용하여 모델을 작성할 수 있게 함
  - Python
    - tensorflow
    - sklearn
  - Spark
  - H2O
  - R
- 배포시 자동으로 RESTful 및 gRPC을 사용하도록 설정
- micro service에 복잡한 runtime graph를 배포할 수 있음. 여기서 복잡한 runtime graph란 다음과 같이 구성될 수 있음
  - Models
  - Router
  - Combiners : 그래프의 응답에 대한 결함(앙상블 모델)
  - Transformers : request 혹은 response에 대한 변환
- 모델의 전체 라이프 사이클 관리
  - 가동중지시간(downtime)없이 runtime graph update(rolling update)
  - Scaling
  - Monitoring
  - Security



![graph](https://user-images.githubusercontent.com/13328380/50679392-86fd1980-1046-11e9-803c-3e48d7dce14d.png)

​    

#### Istio



`Seldon-Core`가  딥러닝 모델을 위한 micro service였다면, `Istio`는 범용적인 micro service이다. `Istio`는 다음과 같은 기능을 제공한다.

- HTTP, gRPC, WebSocker 및 TCP 트래픽에 대한 로드밸런싱
- 풍부한 라우팅 규칙
- 엑세스 제어
  - rate limits and quotas를 지원하는 pluggable policy layer 및 configuration API 지원
- 메트릭, 로그 및 추적
- 강력한 ID 기반 인증 및 권한 부여를 이용한 보안



![screenshot from 2019-01-04 17-32-42](https://user-images.githubusercontent.com/13328380/50679450-c0ce2000-1046-11e9-8c82-1246ee588fe3.png)



kubeflow 에서는 Serving 모델이 있는 Tensorflow만 `Istio`를 이용해 배포하는 것을 가이드하고 다른 딥러닝 모델은 `Seldon-core`를 이용해 배포하는 방향으로 가이드 한다.

​    

## Reference

[[1] Kubeflow: Cloud-native machine learning with Kubernetes](https://opensource.com/article/18/6/kubeflow)

[[2] Kubeflow docs](https://www.kubeflow.org/docs/)

[[3] Katib : Hyper parameter Tuning](https://github.com/kubeflow/katib)