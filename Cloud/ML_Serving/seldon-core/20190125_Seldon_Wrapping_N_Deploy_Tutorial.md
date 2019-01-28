# Seldon Wrapping & Deploy Tutorial



이제 본격적으로 Seldon Core를 이용해서 내 모델을 Wrapping하고 Deploy를 진행해보자.

​     

## Dependencies

​    

### S2I 설치

[Source-to-Image](https://github.com/openshift/source-to-image)는 소스 코드에서 Docker이미지를 생성해주는 툴킷이다. 조금 더 자세한 설명은 추후 시간이 될 때, 진행하도록 하도록 하겠다. Seldon Core는 [S2I](https://github.com/openshift/source-to-image)를 이용해서 딥러닝 모델을 Seldon Core에 맞게끔 Docker Image로 만들어준다.



[S2I](https://github.com/openshift/source-to-image)를 먼저 설치하자. [S2I](https://github.com/openshift/source-to-image) Github 저장소에 설치하는 방법을 자세히 설명해두었으니, 자세한 내용이 궁금하면 참고하도록 하자.

​    

#### Installation

1. Using `go get`

   ```bash
   $ go get github.com/openshift/source-to-image/cmd/s2i
   ```

2. Mac

   ```bash
   $ brew install source-to-image
   ```

3. Linux

   [S2I](https://github.com/openshift/source-to-image)는 리눅스를 위한 Binary 파일을 [Release](https://github.com/openshift/source-to-image/releases/tag/v1.1.13)에서 제공하고 있다. 

   (1)의 `go get`을 통해서 받고 source compile을 진행할 수도 있지만, 번거롭다면, Binary 파일을 받고, 이를 이용해서 설치하는 방식을 사용할 수 있다.

   **압축 해제 및 실행환경 설정**

   ```bash
   $ tar -xvf release.tar.gz
   $ cp /path/to/s2i /usr/local/bin
   ```

4. Windows

   [S2I](https://github.com/openshift/source-to-image)는 Window도 지원하지만, 여기서는 Window를 사용하지 않기 때문에 생략한다. Window유저라면 [S2I](https://github.com/openshift/source-to-image) Installation란을 참고하자.

​    

#### Test

성공적으로 [S2I](https://github.com/openshift/source-to-image)를 설치했다면, 아래 명령어로 잘 실행이 되는지 확인해보자

```bash
$ s2i version
$ s2i
```



![s2i_test](https://user-images.githubusercontent.com/13328380/51732274-1c794f80-20c1-11e9-8918-70d4a889ad84.png)

​    

### Helm 설치

[Helm](https://docs.helm.sh/)는 Kubernetes 위에 application을 빌드하기 위한 package manager다.



#### Installation

[Helm](https://docs.helm.sh/)는 각 OS에 맞는 binary파일들을 제공하고있다. 

binary 파일은 [Release](https://github.com/helm/helm/releases)에서 받을 수 있다. 페이지에서 원하는 Helm의 버전을 받아서 압축을 풀어주고, 환경변수 설정에서 해당 경로를 추가해준 후, 환경변수를 적용해준다.



```bash
# Helm
export PATH=$PATH:<your helm path>/helm-v2.12.3-linux-amd64/linux-amd64
```

```bash
$ source ~/.bashrc

or

$ source ~/.zshrc
```

​    

#### Test

설치를 완료했다면, helm가 잘 작동하는지 확인해보자

![helm_result](https://user-images.githubusercontent.com/13328380/51736413-69fbb980-20cd-11e9-87b4-35e9b00ab4a1.png)

​    

### Seldon Core

이제 `pip` 모듈을 이용해서 `seldon-core`를 설치해주자.

```bash
$ pip3 install seldon-core --user
```



그리고 [Seldon-Core github Release](https://github.com/SeldonIO/seldon-core/releases) 페이지에서 원하는 버전의 source파일을 받은 후, 특정 폴더에 압축을 풀어준다.

```bash
$ unzip seldon-core-0.2.5.zip
```

​    

### Minikube

kubernetes를 로컬환경 돌리고싶다면, Minikube를 사용한다



Minikube의 설치는 [다음](https://kubernetes.io/docs/tasks/tools/install-minikube/)을 참고하자.

```bash
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
$ sudo cp minikube /usr/local/bin && rm minikube
```

​    

## Setting

MNIST 모델을 이용해서 Seldon-Core에 배포하도록 하겠다.

모델 이름은 `DeepMnist`이다.

먼저, 특정 폴더를 만들었다면 폴더 구조는 다음과 같은 형태를 띄어야한다.

여기서 `options`라는 것은 `seldon-core`가 필수적으로 요구하는 구조는 아니라는 의미다.

> 실은 없어도 된다.

하지만 `contract.json`, `DeepMnist.py`, `.s2i`폴더는 필수다.

```bash
.
├── contract.json
├── DeepMnist.py
├── model # (weights파일 저장 경로, options)
├── .s2i
├── test.py # train 모델을 확인해보기 위한 파일(options)
└── train_mnist.py # train을 위한 파일(options)
```

- `contract.json` :  seldon-core와 통신하기 위한 모델의 input, output 명세
- `DeepMnist.py` : 딥러닝 모델
- `.s2i` : seldon-core에 배포하기 위한 모델 명세 및 의존성 패키지 명시 파일을 가지고 있는 폴더



`DeepMnist.py`는 마지막에 설명하고, 먼저 `contract.json`파일과, `.s2i`폴더에 대해서 살펴보자

​    

#### Contract.json

`contract.json`파일의 구성은 다음과 같다.

```bash
{
    "features":[
	{
	    "name":"x",
	    "dtype":"FLOAT",
	    "ftype":"continuous",
	    "range":[0,1],
	    "repeat":784
	}
    ],
    "targets":[
	{
	    "name":"class",
	    "dtype":"FLOAT",
	    "ftype":"continuous",
	    "range":[0,1],
	    "repeat":10
	}
    ]
}
```

- `features` : `입력 노드의 이름`과, `데이터 타입`, `데이터 종류`, `데이터 범위`, `차원`등을 명시한다.
- `targets` : `출력값의 이름`, `데이터 타입` , `데이터 종류`, `데이터 범위`, `차원`등을 명시한다.

이를 이용하여 request로 들어온 값과, model의 input/output값을 parsing해서 사용하는 듯 하다.

​    

#### .s2i

`.s2i` 폴더 구조는 다음과 같다.

```bash
.s2i
├── environment
└── requirements.txt
```

- `environment` : 모델 명세
- `requirements.txt` : `DeepMnist.py`에 필요한 의존성 파일

​    

### Model

이제 본격적으로 모델을 학습시키고, 학습키긴 것을 wrapping해서 seldon-core에 배포해보자

​    

#### Train Model

MNIST 모델을 Tensorflow를 이용해서 먼저 학습을 진행해보자

` train.py` 코드는 다음과 같다.

```python
from tensorflow.examples.tutorials.mnist import input_data
import tensorflow as tf
import numpy as np
import math

(x_train, y_train), (x_test , y_test) = tf.keras.datasets.mnist.load_data()
x_train = (x_train/255).astype(np.float32).reshape(-1, 784)
x_test = (x_test/255).astype(np.float32).reshape(-1, 784)

val_range = int(len(x_test) * 0.8)

x_val = x_test[:val_range, :]
y_val = y_test[:val_range]

x_test = x_test[val_range:, :]
y_test = y_test[val_range:]

batch_size = 100
learning_rate = 0.1
step_num = 2000

image_size = 28
class_num = 10

if __name__ == "__main__":

    x = tf.placeholder(tf.float32, [None, image_size * image_size], name="x")

    W1 = tf.Variable(
        tf.truncated_normal([image_size * image_size, 128], stddev=1.0 / math.sqrt(float(image_size * image_size))))
    b1 = tf.Variable(tf.zeros([128]))
    h1 = tf.nn.relu(tf.matmul(x, W1) + b1)

    W2 = tf.Variable(tf.truncated_normal([128, 64], stddev=1.0 / math.sqrt(float(128))))
    b2 = tf.Variable(tf.zeros([64]))
    h2 = tf.nn.relu(tf.matmul(h1, W2) + b2)

    W3 = tf.Variable(tf.truncated_normal([64, class_num], stddev=1.0 / math.sqrt(float(64))))
    b3 = tf.Variable(tf.zeros([class_num]))
    model = tf.add(tf.matmul(h2, W3), b3, name="y")


    y_ = tf.placeholder(tf.int64, [None])
    y_ = tf.to_int64(y_)

    cross_entropy = tf.nn.sparse_softmax_cross_entropy_with_logits(labels=y_, logits=model)
    loss = tf.reduce_mean(cross_entropy)

    optimizer = tf.train.GradientDescentOptimizer(learning_rate)
    global_step = tf.Variable(0, name='global_step', trainable=False)
    train_op = optimizer.minimize(loss, global_step=global_step)

    init = tf.initialize_all_variables()

    sess = tf.Session()
    sess.run(init)

    batch_num = 0

    for step in range(step_num):
        batch_indices = np.random.choice(range(x_train.shape[0]), size=batch_size, replace=False)
        batch_xs = x_train[batch_indices]
        batch_ys = y_train[batch_indices]

        val_indices = np.random.choice(range(x_val.shape[0]), size=batch_size, replace=False)
        val_xs = x_val[val_indices]
        val_ys = y_val[val_indices]

        _, train_loss = sess.run([train_op, loss], feed_dict={x: batch_xs, y_:batch_ys})

        val_loss = sess.run(fetches = loss, feed_dict={x:val_xs, y_: val_ys})

        if (step % 100 == 0 and step != 0):
            print("epoch : {:3}, train_loss : {:.2f}, val_loss : {:.2f}".format(step, train_loss, val_loss))

    yhat = np.argmax(sess.run(model, feed_dict={x: x_test}), axis=1)
    print("acc : {:.2%}".format(np.mean(yhat == y_test)))

    saver = tf.train.Saver()
    saver.save(sess, "model/deep_mnist_model")

```

해당 코드는 누구나 잘 아는 MNIST코드니 별도로 설명하지 않겠다.

여기서 Seldon-Core로 배포하기 위해서 중요한 것은 아래와 같다.

​    

##### Model Input/Output

```python
# model input
x = tf.placeholder(tf.float32, [None, image_size * image_size], name="x")
# model output
model = tf.add(tf.matmul(h2, W3), b3, name="y")
```

여기서 input과 output의 이름을 `x`, `y`라고 맞췄는데, 이는 `contract.json`의 `features`와 `targets`의 이름과 매칭되는 부분이다. 이를 꼭 매칭해줘야한다.

​    

`contract.json`파일을 다시 한번 확인해보면, `name`이라는 엘리먼트가 `x`, `y`로 매칭이 되는 것을 확인할 수 있을 것이다. 모델의 input/output 노드의 이름과 `contract.json`파일의 input/output 이름이 매칭이 되지 않으면 seldon-core는 입력/출력 값을 인식할 수가 없다. 그러니 Tensorflow Graph를 그릴때, 꼭 이를 매칭해주도록 하자.

```bash
{
    "features":[
	{
	    "name":"x",
	    "dtype":"FLOAT",
	    "ftype":"continuous",
	    "range":[0,1],
	    "repeat":784
	}
    ],
    "targets":[
	{
	    "name":"class",
	    "dtype":"FLOAT",
	    "ftype":"continuous",
	    "range":[0,1],
	    "repeat":10
	}
    ]
}
```

​    

##### Path of Weights

코드를 확인해보면, 학습 후에 학습 가중치 파일을 저장하는 것을 확인할 수 있다. `train.py`는 학습만 하고, 실제로 inference는 다른 파일을 작성해서 사용하니, 해당 가중치 파일 경로를 잘 기억하고 있어야한다.

```python
saver = tf.train.Saver()
saver.save(sess, "model/deep_mnist_model")
```

​    

#### Inference Class

이제 `train.py`에서 학습한 가중치 파일을 불러와서 inference하는 model class를 생성해줘야한다.

model class는 seldon-core에서 요구하는 포맷에 맞춰서 작성해야한다.



seldon-core가 요구하는 class 포맷은 다음과 같다.

1. class 이름과, python code의 이름이 같아야한다.
2. ` __init__`, `predict` 이 존재해야한다.



해당 포맷을 맞춰서 `DeepMnist.py`라는 파이썬 파일을 만들었고, 코드는 다음과 같이 작성했다.

```python
import tensorflow as tf
import numpy as np

class DeepMnist(object):
    def __init__(self):
        self.class_names = ["class:{}".format(str(i)) for i in range(10)]
        self.sess = tf.Session()
        saver = tf.train.import_meta_graph("model/deep_mnist_model.meta")
        saver.restore(self.sess, tf.train.latest_checkpoint("./model/"))

        graph = tf.get_default_graph()
        self.x = graph.get_tensor_by_name("x:0")
        self.y = graph.get_tensor_by_name("y:0")

    def predict(self, X, feature_names=None):
        predictions = self.sess.run(self.y, feed_dict={self.x:X})
        return predictions.astype(np.float64)
```



코드의 작동방식은 

1. `__init__`함수에서 `class_name`, `session`, `graph`,`input/output`값을 명시한다.
2. `predict`함수에서 입력값을 받아 연산 후, `numpy array`로 출력값을 반환한다.

이다.



여기서 중요하게 확인해야하는 점은 다음과 같다.

​    

##### 1. class 파일 이름, class 이름, `environment`의 `MODEL_NAME`이 같아야한다. 

- Class 파일 이름 : `DeepMnist.py`
- Class 이름 : `DeepMnist`
- `MODEL_NAME` : `DeepMnist`

​    

##### 2. `__init__`함수에서 session과 가중치 파일을 로드해야한다.

- session

  ```python
  sess = tf.Session()
  ```

- 가중치 파일 로드

  > 여기서 가중치 파일은 `train.py`에서 저장한 가중치 파일 경로와 같다는 점을 확인하자.

  ```python
  saver = tf.train.import_meta_graph("model/deep_mnist_model.meta")
  saver.restore(self.sess, tf.train.latest_checkpoint("./model/"))
  ```

​    

##### 3. input / output node를 Graph에서 가져와서 지정한다.

```python
self.x = graph.get_tensor_by_name("x:0")
self.y = graph.get_tensor_by_name("y:0")
```

​        

#### Test

`train.py`로 학습을 완료했고, `DeepMnist.py`도 잘 만들었다면 다음과 같은 코드로, Mnist모델이 잘 작동하는지 확인하자.

```python
import tensorflow as tf
import numpy as np
from DeepMnist import DeepMnist

mnist = DeepMnist()

(x_train, y_train), (x_test , y_test) = tf.keras.datasets.mnist.load_data()
x_train = (x_train/255).astype(np.float32).reshape(-1, 784)
x_test = (x_test/255).astype(np.float32).reshape(-1, 784)

val_range = int(len(x_test) * 0.8)

x_val = x_test[:val_range, :]
y_val = y_test[:val_range]

x_test = x_test[val_range:, :]
y_test = y_test[val_range:]

print(mnist.predict(x_test))
```

​    

Seldon-Core에 배포할 모델 준비가 완료되었다.

이제 Seldon-Core에 배포할 수 있도록 Docker Image로 Wrapping을 해보자.

​    

### Wrapping Model

작성한 모델을 S2I를 이용하여 Docker Image로 Wrapping 작업은 정말 쉽다.

아래와 같은 명령어를 이용해서 Wrapping 할 수 있다.

> 여기서 Python2.x와 Python3.x에 따라서 builder가 다르게 사용되는데, 이는 [다음](https://github.com/SeldonIO/seldon-core/blob/master/docs/wrappers/python.md)을 참고하자

```bash
$ s2i build . seldonio/seldon-core-s2i-python3:0.4 deep-mnist:0.1
> 
---> Installing application source...
Build completed successfully
```

​    

해당 작업이 완료되었으면, Docker에서 `deep-mnist:0.1`이라는 이미지를 확인해볼 수 있다.

```bash
$ docker images
>
REPOSITORY                         TAG                       IMAGE ID
deep-mnist                         0.1                       8e639d430215
```

​    

#### Test

Docker image를 만들었다면, Docker Image가 잘 작동하는지 확인해야한다.

아래 명령어를 이용해서 docker container를 만들고, seldon-core가 제공해주는 tester를 이용해서 테스트 해보자

```bash
$ docker run --name "mnist_predictor" -d --rm -p 5000:5000 deep-mnist:0.1
> 
6afefec1781e482bb986977d8940a3ec5f06336a6d3bb333803d7a27335ac356

$ docker ps
>
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS
6afefec1781e        deep-mnist:0.1      "/s2i/bin/run"      3 seconds ago       Up 

$ seldon-core-tester contract.json 0.0.0.0 5000 -p
>
----------------------------------------
SENDING NEW REQUEST:
{'meta': {}, 'data': {'names': ['x1', 'x2', 'x3', 'x4', 'x5', 'x6', 'x7', 'x8', 'x9', 'x10', 'x11', 'x12', 'x13', 'x14', 'x15', 'x16', 'x17', 'x18', 'x19', 'x20', 'x21', 'x22', 'x23', 'x24', 'x25', 'x26', 'x27', 'x28', 'x29', 'x30', 'x31', 'x32', 'x33', 'x34', 'x35', 'x36', 'x37', 'x38', 'x39', 'x40', 'x41', 'x42', 'x43', 'x44', 'x45', 'x46', 'x47', 'x48', 'x49', 'x50', 'x51', 'x52', 'x53', 'x54', 'x55', 'x56', 'x57', 'x58', 'x59', 'x60', 'x61', 'x62', 'x63', 'x64', 'x65', 'x66', 'x67', 'x68', 'x69', 'x70', 'x71', 'x72', 'x73', 'x74', 'x75', 'x76', 'x77', 'x78', 'x79', 'x80', 'x81', 'x82', 'x83 ...
0.378, 0.05, 0.87, 0.651, 0.174, 0.449, 0.863, 0.297, 0.588, 0.45, 0.03, 0.9, 0.631, 0.155, 0.61, 0.766, 0.735, 0.197, 0.734, 0.941, 0.098, 0.855, 0.641, 0.147, 0.298, 1.0, 0.545, 0.914, 0.243, 0.232, 0.134, 0.326, 0.048, 0.836]]}}

RECEIVED RESPONSE:
{'data': {'names': ['class:0', 'class:1', 'class:2', 'class:3', 'class:4', 'class:5', 'class:6', 'class:7', 'class:8', 'class:9'], 'ndarray': [[0.9650402069091797, -4.994695663452148, 4.181480407714844, 8.419814109802246, -7.819960594177246, 6.499182224273682, -4.877837657928467, -3.646813154220581, 7.951652526855469, -6.298679828643799]]}, 'meta': {}}

Time 0.015633344650268555
```

test가 잘 작동하는 것을 확인할 수 있다.

이제 배포를 진행해보자!

​         

### Deploy to Seldon-Core

이제 배포에 대해서 알아보자.

배포의 부분은 Kubernetes와 Helm을 더 잘아야하는 영역이기 때문에 자세히 설명하지는 않으며, 이렇게 하면 쉽게 마이크로 아키텍쳐에 배포할 수 있음을 확인할 수 있는 정도로 넘어가자.

​    

#### Minikube Start

로컬에서 구동할 수 있는 kubenetes인 Minikube를 구동시키고 몇가지 설정을 진행한다.

```bash
$ minikube start
>
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```

​     

#### Cluster role을 서비스 계정에 바인딩

```bash
$ kubectl create clusterrolebinding kube-system-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

​    

#### Rollout 상태 확인

``` bash
$ kubectl rollout status deploy/tiller-deploy -n kube-system
```

​         

#### Helm init

```bash
$ helm init
>
Creating /home/keti-1080ti/.helm 
Creating /home/keti-1080ti/.helm/repository 
Creating /home/keti-1080ti/.helm/repository/cache 
Creating /home/keti-1080ti/.helm/repository/local 
Creating /home/keti-1080ti/.helm/plugins 
Creating /home/keti-1080ti/.helm/starters 
Creating /home/keti-1080ti/.helm/cache/archive 
Creating /home/keti-1080ti/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /home/keti-1080ti/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```

​    

초기화된 Helm을 이용하여, Seldon-Core관련 Package를 설치한다.

```bash
$ helm install <seldon-core path>/helm-charts/seldon-core-crd --name seldon-core-crd  --set usage_metrics.enabled=true
>
NAME:   seldon-core-crd
LAST DEPLOYED: Fri Jan 25 18:26:07 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ServiceAccount
NAME                        SECRETS  AGE
seldon-spartakus-volunteer  1        0s

==> v1beta1/ClusterRole
NAME                        AGE
seldon-spartakus-volunteer  0s

==> v1beta1/ClusterRoleBinding
NAME                        AGE
seldon-spartakus-volunteer  0s

==> v1/Pod(related)
NAME                                         READY  STATUS             RESTARTS  AGE
seldon-spartakus-volunteer-7d9895b6fb-dr6tm  0/1    ContainerCreating  0         0s

==> v1/ConfigMap
NAME                     DATA  AGE
seldon-spartakus-config  3     0s

==> v1beta1/CustomResourceDefinition
NAME                                         AGE
seldondeployments.machinelearning.seldon.io  0s

==> v1beta1/Deployment
NAME                        DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
seldon-spartakus-volunteer  1        1        1           0          0s


NOTES:
NOTES: TODO

```

​    

```bash
$ helm install /home/keti-1080ti/Downloads/seldon-core-0.2.5/helm-charts/seldon-core --name seldon-core
>
NAME:   seldon-core
LAST DEPLOYED: Fri Jan 25 18:26:35 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/RoleBinding
NAME    AGE
seldon  0s

==> v1/Service
NAME                          TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)                        AGE
seldon-core-seldon-apiserver  NodePort   10.107.148.47  <none>       8080:31939/TCP,5000:30630/TCP  0s
seldon-core-redis             ClusterIP  10.110.178.74  <none>       6379/TCP                       0s

==> v1beta1/Deployment
NAME                                DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
seldon-core-seldon-apiserver        1        1        1           0          0s
seldon-core-seldon-cluster-manager  1        1        1           0          0s
seldon-core-redis                   1        1        1           0          0s

==> v1/Pod(related)
NAME                                                 READY  STATUS             RESTARTS  AGE
seldon-core-seldon-apiserver-6f5fb6bf69-47tjj        0/1    ContainerCreating  0         0s
seldon-core-seldon-cluster-manager-75bf58f585-g98sh  0/1    ContainerCreating  0         0s
seldon-core-redis-7d47cb884c-8npm6                   0/1    ContainerCreating  0         0s

==> v1/ServiceAccount
NAME    SECRETS  AGE
seldon  1        0s

==> v1beta1/Role
NAME          AGE
seldon-local  0s


NOTES:
Thank you for installing Seldon Core.

Documentation can be found at https://github.com/SeldonIO/seldon-core
```

​     

#### Minikube의 docker환경 설정 확인

```bash
$ eval $(minikube docker-env)
```

​    

#### DeepMnist 배포를 위한 `deep_mnist.json` 작성

kubernetes에서 pod를 생성하기 위해, pod에 대한 명세인 `deep_mnist.json`파일을 다음과 같이 생성한다.

```json
{
    "apiVersion": "machinelearning.seldon.io/v1alpha2",
    "kind": "SeldonDeployment",
    "metadata": {
        "labels": {
            "app": "seldon"
        },
        "name": "deep-mnist"
    },
    "spec": {
        "annotations": {
            "project_name": "Tensorflow MNIST",
            "deployment_version": "v1"
        },
        "name": "deep-mnist",
        "oauth_key": "oauth-key",
        "oauth_secret": "oauth-secret",
        "predictors": [
            {
                "componentSpecs": [{
                    "spec": {
                        "containers": [
                            {
                                "image": "deep-mnist:0.1",
                                "imagePullPolicy": "IfNotPresent",
                                "name": "classifier",
                                "resources": {
                                    "requests": {
                                        "memory": "1Mi"
                                    }
                                }
                            }
                        ],
                        "terminationGracePeriodSeconds": 20
                    }
                }],
                "graph": {
                    "children": [],
                    "name": "classifier",
                    "endpoint": {
			"type" : "REST"
		    },
                    "type": "MODEL"
                },
                "name": "single-model",
                "replicas": 1,
		"annotations": {
		    "predictor_version" : "v1"
		}
            }
        ]
    }
}
```

위의 내용은 kuberntes에 대해서 알아야하는데, 이 부분은 추후에 kubernetes관련 내용을 소개할 때, 이야기하도록 하겠다.

​         

#### deep_mnist 배포

이제 DeepMnist 모델을 배포하자

```bash
$ kubectl create -f deep_mnist.json
>
seldondeployment.machinelearning.seldon.io/deep-mnist created
```

​    

#### 배포 확인

```bash
$ kubectl get seldondeployments deep-mnist -o jsonpath='{.status}'
>
map[predictorStatus:[map[replicasAvailable:0 name:deep-mnist-single-model-8969cc0 replicas:1]] state:Creating]%
```

​    

#### Test

``` bash
$ seldon-core-api-tester contract.json \
    `minikube ip` `kubectl get svc -l app=seldon-apiserver-container-app -o jsonpath='{.items[0].spec.ports[0].nodePort}'` \
    --oauth-key oauth-key --oauth-secret oauth-secret -p
>
----------------------------------------
SENDING NEW REQUEST:
{'meta': {}, 'data': {'names': ['x1', 'x2', 'x3', 'x4', 'x5', 'x6', 'x7', 'x8', 'x9', 'x10', 'x11', 'x12', 'x13', 'x14', 'x15', 'x16', 'x17', 'x18', 'x19', 'x20', 'x21', 'x22', 'x23', 'x24', 'x25', 'x26', 'x27', 'x28', 'x29', 'x30', 'x31', 'x32', 'x33', 'x34', 'x35', 'x36', 'x37', 'x38', 'x39', 'x40', 'x41', 'x42', 'x43', 'x44', 'x45', 'x46', 'x47', 'x48', 'x49', 'x50', 'x51', 'x52', 'x53', 'x54', 'x55', 'x56', 'x57', 'x58', 'x59', 'x60', 'x61', 'x62', 'x63', 'x64', 'x65', 'x66', 'x67', 'x68', 'x69', 'x70', 'x71', 'x72', 'x73', 'x74', 'x75', 'x76', 'x77', 'x78', 'x79', 'x80', 'x81', 'x82', 'x83', 'x84', 'x85', 'x86', 'x87', 'x88', 'x89', 'x90', 'x91', 'x92', 'x93', 'x94', 'x95', 'x96', 'x97', 'x98', 'x99', 'x100', 'x101', 'x102', 'x103', 'x104', 'x105', 'x106', 'x107', 'x108', 'x109', 'x110', 'x111', 'x112', 'x113', 'x114', 'x115', 'x116', 'x117', 'x118', 'x119', 'x120', 'x121', 'x122', 'x123', 'x124', 'x125', 'x126', 'x127', 'x128', 'x129', 'x130', 'x131', 'x132', 'x133', 'x134', 'x135', 'x136',
...
0.6197065346204125, 0.9799256224030894, 0.8687334200214144, 0.6577776091375379, 0.8558685246516896, 0.6180778489426687, 0.8622054164141099, 0.9507921310323462, 0.8055754923427366, 0.29225393937287625, 0.6254351758805801, 0.6768212323965608, 0.3116472268027898, 0.6557058548834688, 0.23119707992895044, 0.13190338977615745, 0.40901988080335794, 0.03917536894326523, 0.42151904014089125, 0.8668169345292223, 0.6036702501313279, 0.07353956005827444, 0.2530749715664924, 0.4841378663510716, 0.10674014689819211, 0.9295736771669293, 0.9133569415419456, 0.7492267055523861, 0.7607158556939608, 0.9108712699619236, 0.6216360186043961, 0.3719583802066939]]}}

Getting token from http://192.168.39.58:31939/oauth/token
{"access_token":"93689b1e-13e1-4bb3-a3fc-5532510cee9b","token_type":"bearer","expires_in":43199,"scope":"read write"}
RECEIVED RESPONSE:
{'code': 103, 'info': 'java.net.NoRouteToHostException: Host is unreachable (Host unreachable)', 'reason': 'Microservice error', 'status': 'FAILURE'}
```

​    

## Reference

[[1]. Source-To-Image (S2I)](https://github.com/openshift/source-to-image)