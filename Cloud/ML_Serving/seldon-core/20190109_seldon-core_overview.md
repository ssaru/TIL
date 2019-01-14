# Introducing Seldon Core

Seldon-Core는 회사들의 ML/DL 프로젝트의 마지막 단계 - (Model을 Production에 넣어서 실제 문제를 풀수있게 만듬)를 해결하는데 초점을 둔다.



이를 통해서 데이터 사이언티스트들은 더 나은 모델을 만들기 위해서 집중하고, 개발자들은 그들이 이해한 툴을 이용해서 조금 더 효과적으로 ML Solution을 배포하고 관리할 수 있다.



Seldon-Core 플랫폼의 목적은 다음과 같다.

- 데이터 사이언티스트가 ML toolkit이나 Programming language를 이용해서 만들어진 모델을 배포할 수 있게 한다. 초기에서는 H20, Spark, Tensorflow나 Scikit-learn과 같은 Python 기반의 도구/언어들을 지원한다.
- 배포시에 ML/DL 프로젝트들이  REST / gRPC에 노출하여 예측이 필요한 비지니스 서비스 및 프로그램에 쉽게 통합할 수 있게 한다.
- 가동중지 없이 update runtime graph, scaling, monitoring, security를 포함해서 배포된 모델의 전체 라이프사이클 관리를 핸들링한다.

​    

## Inference Graph

API의 End-Point 뒤에 단일 모델만 두는 대신에 Seldon Core는 complex runtime inference graph를 마이크로 서비스로써의 컨테이너에 배포할 수 있게 한다.



여기서 이야기하는 Graph의 구성은 아래와 같다.

- **Models**: 하나 혹은 하나 이상 모델을 위한 실행 가능한 runtime inference
- **Router**: API요청을 sub-graph에 라우팅한다. E.g AB Test나 더 동적인 Multi-Armed Bandits과 같은 실험들을 제공한다.
- **Combiners**: sub-graph의 응답을 혼합한다. E.g 앙상블 모델
- **Transformer**: request나 respond를 변환한다. E.g 어떤 feature의 request를 변환함



![1 ybb5c8kdfgnb2djtzxyosa](https://user-images.githubusercontent.com/13328380/50886650-52b59e80-1434-11e9-879e-15bc3ff3b60b.png)

<center>
	[Example runtime Model Graph ]    
</center>

​    

## Why it this important?

### 1. Efficiency

전통적인 infrastructure 스택 및 DevOps 프로세스는 ML 도메인에 잘 적용이 되지 않으며, 해당 도메인에 있어서 이를 해결할 OpenSource가 형성되어있지 않아서 제한적이다. 이로인해 기업들은 막대한 비용을 들여서 자체 서비스를 구축하거나, 독점적인 서비스를 이용한다.



또한 DevOps와 ML에 걸쳐서 다양한 스킬셋을 가지고있는 데이터 엔지니어의 수는 극히 적다. 이러한 비효율적인 구조는 데이터 사이언티스트가 서비스 및 품질문제에 몰입하게 만들게 되며 이로인해 데이터 사이언티스트가 집중해야할 영역(더 나은 모델을 만드는 것)에서 멀어지게 만든다.



### 2. Innovation

모델이 실제 Production level에 있는 경우에만 실제 문제에 대한 모델의 성능을 측정할 수 있다. Seldon-Core는 데이터 과학분야를 application release cycle로 부터 분리하여 반복 주기를 더 가속화할 수 있다. AB 테스트나, Multi-Armed Bandit와 같은 실험 프로세스는 종종 기계학습을 위해 디자인되지 않은 툴을 사용하기 때문에 일반적으로 스택에서 굉장히 높은 곳에 위치하고 있다. inference graph와 모델들의 촘촘한 연결은 반복주기를 가속화하며, 유즈 케이스를 신속하게 구축/최적화 하여 혁신과 ROI를 가속화할 수 있다.

​    

### 3. Freedom

IDC 2017의 연구에 따르면, 상업적인 혹은 백업, 탄력성, 규제와 같은 이유/목적으로 인해서 40%의 유럽조직이 application을 클라우드로 확장하고 있다.  단일 클라우드 ML 시스템에 종속적이지 않은 플랫폼을 사용하면, 다양한 ML 클라우드 시스템 사용을 촉진할 수 있다. 또한 Tensorflow와 같은 DL을 위한 프레임워크가 지속적으로 나오고 있기 때문에 더 빠른 모델 개발을 위해 프레임워크에 종속되지 않는 플랫폼이 필요하다.

​    

## Overview Seldon-Core

Seldon Core은 아래의 그림과 같이 3단계로 작업이 진행된다.



- Seldon-Core 설치
- 내부 Seldon Microservice API에 맞게 runtime ML을 Docker Image로 Packaging(S2I 이용)
- ksonnet나 helm을 이용하여 runtime service graph를 Seldon Deployment resource로 정의하고, 배포



![steps](https://user-images.githubusercontent.com/13328380/50888497-23556080-1439-11e9-9a8e-92d323cba094.png)

​    

## Install Seldon-Core

Seldon-Core를 설치하는 방법에 대해서는 [가이드 문서](https://github.com/SeldonIO/seldon-core/blob/master/docs/install.md)에 잘 나와있으니, 이를 이용하여 설치하면 된다.

​    

## Wrap Your model

만약에 Serving을 원하는 컴포넌트(딥러닝 모델)가 있다면, 이를 Seldon-Core의 MicroService API와 상호작용이 되는 Docker Conatainer로 wrapping되어야 한다.



Seldon-Core는 다른 언어 및 프레임워크로 이미 작성된 Machine Learning 프로젝트 코드들을 Seldon-Core내부에서 작동될 수 있도록 Docker Container를 쉽게 만들어주는 wrapper를 제공한다. 현재 Seldon-Core는 RedHat의 Source-to-Image(S2I)라는 tool을 추천한다.

​    

## Define Runtime Service Graph

Kubernetes에서 원하는 Machine Learning Graph를 실행하려면 만들어진 컴포넌트를 어떻게 Service Graph와 맞춰서 표현할것인지에 대해서 정의해야한다. 이러한 것은  [SeldonDeployment Kubernetes Custom resource](https://github.com/SeldonIO/seldon-core/blob/master/docs/reference/seldon-deployment.md)에서 정의된다.



해당 내용에 대한 가이드는 [A guide to constructing this custom resource service graph is provided](https://github.com/SeldonIO/seldon-core/blob/master/docs/inference-graph.md)를 참조하자.



![graph](https://user-images.githubusercontent.com/13328380/51100540-ea8f0000-1819-11e9-8106-a5ddb4fa45ff.png)



​    

## Deploy and Serve Predictions

다른 서비스들을 kubernetes 올리는 것과 같이 ML 서비스를 `kubectl` 명령어를 이용해서 배포할 수 있다. 해당 내용은 [여기](https://github.com/SeldonIO/seldon-core/blob/master/docs/deploying.md)를 참조한다.



## Reference

[[1]. Introducing Seldon Core — Machine Learning Deployment for Kubernetes](https://www.seldon.io/2018/01/26/introducing-seldon-core-machine-learning-deployment-for-kubernetes/)