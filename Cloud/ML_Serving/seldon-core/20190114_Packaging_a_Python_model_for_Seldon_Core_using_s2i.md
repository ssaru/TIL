# Packaging a Python model for Seldon Core using s2i

[source-to-image app s2i](https://github.com/openshift/source-to-image)을 이용하여 Python model을 Seldon-Core에 배포가 가능한 Docker Image로 wrapping하는 방법에 대해서 소개한다.

#### 

Seldon-Core에서는 다음과 같은 언어로 작성된 모델을 랩핑할 수 있게 지원한다

- Python
- R
- Java
- NodeJs
- Go

#### 

나의 경우에는 주로 Python을 사용하기 때문에 Python만 설명하도록 하겠다

​        

## Step 1 - Install s2i

Ubuntu 18.04 혹은 17.04에서 다음과 같이 설치하였다.

#### 

- [S2I Release page](https://github.com/openshift/source-to-image/releases/tag/v1.1.13)에서 `v1.1.13-linux-amd64.tar.gz`다운로드
- unzip
- `cp /path/to/s2i /usr/local/bin`를 이용해 s2i 실행파일을 이동
- `s2i version` 입력해 설치 확인

​    

## Step2 - Create your source code

python 모델을 s2i builder image를 이용하여 패키징 하려면 다음과 같은 사항이 충족되어야함

- `class`로 구성되어있으며, 실제로 작동하는 python file
- `requirements.txt`나 `setup.py` 파일
- `.s2i/environment` - s2i 빌더가 모델을 정확하게 wrapping하기 위해 사용하는 모델 정의 파일  

#### 

### Python file

파일 이름과 같은 class 이름이 정의되어있어야 함. 예를들어 `wrappers/s2i/python/test/model-template-app/MyModel.py` 파일에 다음과 같은 Python 코드가 존재해야함

```python
class MyModel(object):
    """
    Model template. You can load your model parameters in __init__ from a location accessible at runtime
    """

    def __init__(self):
        """
        Add any initialization parameters. These will be passed at runtime from the graph definition parameters defined in your seldondeployment kubernetes resource manifest.
        """
        print("Initializing")

    def predict(self,X,features_names):
        """
        Return a prediction.

        Parameters
        ----------
        X : array-like
        feature_names : array of feature names (optional)
        """
        print("Predict called - will run identity function")
        return X
```

- 해당 python 파일은 `MyModel.py`이며 `MyModel` class를 정의함
- 해당 class는 `predict` 메소드를 포함하고 있어야하며, 인자는 numpy array타입인 `X`, 와 `features_names(optional)`을 가져야하며, numpy array 타입의 prediction return 값을 가져야함
- class 생성자에 요구되는 initialization 코드를 추가할 수 있음
- prediction return array는 최소 2차원이어야함

#### 

### requirements.txt

의존성 패키지를 설치하기 위한 파일이며, `pip`를 이용하여 설치할 수 있음. 원하는 경우 setup.py로 대체할 수 있음

#### 

###  .s2i/environment

모델을 wrapping하기 위한 Python builder image는 core parameter정의가 요구되며, 해당 정의의 예시는 다음과 같음

```bash
MODEL_NAME=MyModel
API_TYPE=REST
SERVICE_TYPE=MODEL
PERSISTENCE=0
```

*(해당 값은  이미지 빌드시 제공되거나 대체할 수 있음)*

​      

## Step 3  - Build your image

`s2i build`를 사용하는 것은 코드에서 Docker Image를 만들게 된다. 따라서 PC에는 Docker가 설치되어있어야하며, 만약에 코드가 git 서비스플랫폼의 public repository에 있다면 선택적으로 Git을 사용할 수 도 있다. 

#### 

3가지 Python builder image를 사용할 수 있는데, 해당 builder 종류는 다음과 같다.

- Python2 : seldonio/seldon-core-s2i-python2:0.4

- Python3.6 : seldonio/seldon-core-s2i-python36:0.4, seldonio/seldon-core-s2i-python3:0.4

  - Python3.7(Nov 2018)에서 Tensorflow를 사용하는 것에 이슈가 있으며, Python3.7은 공식적으로 Tensorflow를 지원하지 않는다.(Dec 2018)

    > Note there are [issues running TensorFlow under Python 3.7](https://github.com/tensorflow/tensorflow/issues/20444) (Nov 2018) and Python 3.7 is not officially supported by TensorFlow (Dec 2018).

- Intel nGraph를 이용해 Python 3.6 plus ONNX지원 : seldonio/seldon-core-s2i-python3-ngraph-onnx:0.1

#### 

`s2i`를 이용하면 git remote public repository 혹은 local source folder로 부터 이미지를 빌드할 수 있다. 

 [s2i docs](https://github.com/openshift/source-to-image/blob/master/docs/cli.md#s2i-build) 를 참조하자.

#### 

`s2i`를 이용하여 Image를 build하는 명령어는 다음과 같다.

```bash
$ s2i build <git-repo> seldonio/seldon-core-s2i-python2:0.4 <my-image-name>
$ s2i build <src-folder> seldonio/seldon-core-s2i-python2:0.4 <my-image-name>
```

*(python3을 사용한다면 `seldonio/seldon-core-s2i-python2:0.4`를  ` seldonio/seldon-core-s2i-python36:0.4, seldonio/seldon-core-s2i-python3:0.4`로 변경한다.)*

#### 

아래 명령어는 test template model을 사용하는 예제이다.

```bash
$ s2i build https://github.com/seldonio/seldon-core.git --context-dir=wrappers/s2i/python/test/model-template-app seldonio/seldon-core-s2i-python2:0.4 seldon-core-template-model
```

위의 명령어는 다음과 같은 절차를 따른다.

- remote git repository  <https://github.com/seldonio/seldon-core.git>를 이용한다.
- remote git repository의 directory를 ``wrappers/s2i/python/test/model-template-app`로 설정한다.
- builder image로 ``seldonio/seldon-core-s2i-python2`를 사용한다.
- `seldon-core-template-model`이라는 docker image를 만든다.

#### 

만약에 local source folder에서 실행하고싶다면, 위의 예제를 clone받아서 이를 실행한다.

```bash
$ git clone https://github.com/seldonio/seldon-core.git
$ cd seldon-core
$ s2i build wrappers/s2i/python/test/model-template-app seldonio/seldon-core-s2i-python2:0.4 seldon-core-template-model
```

#### 

`s2i`의 사용법에 대해서 더 자세히 알고싶으면 다음과 같은 명령어를 통해서 확인할 수 있다.

```bash
$ s2i usage seldonio/seldon-core-s2i-python2:0.4
$ s2i usage seldonio/seldon-core-s2i-python3:0.4
$ s2i build --help
```

#### 

### Using with Keras/Tensorflow Models

Flask가 Model을 초기화한 쓰레드와는 다른 별도의 쓰레드에서 prediction을 요청할 수 있기 때문에 Tensorflow 백엔드 기반의 keras모델이 정확하게 작동하는 것을 보장하기 위해서는 모델이 로드된 후 _make_predict_function()`을 호출해야한다. 

해당 내용은  [here](https://github.com/keras-team/keras/issues/6462) 를 참조해라

​     

## Reference

### Environment Variables

builder image이해할 수 있는 environment varialbe는 아래에 설명되어있다.

environment variable은 `.s2i/environment` 파일 혹은 `s2i build` 명령행을 통해 제공할 수 있다.

#### 

#### MODEL_NAME

model을 포함하는 class의 이름이다. 또한 이는 python file의 이름과 같다.

#### 

#### API_TYPE

생성할 API 타입. 이는 `REST`나 `gPRC` 둘 중 하나를 의미하며 이를 선택할 수 있다.

#### 

#### SERVICE TYPE

생성될 서비스의 타입. 

다음과 같은 서비스 타입이 존재한다.

- Model
- Router
- Transformer
- Combiner
- Outlier_detector

#### 

#### PERSISTENCE

`0`혹은 `1`로 설정되며, 기본값은 0이다.

- `1` : redis에 모델이 주기적으로 저장되며, 모델이 redis의 있는 경우에는 로드에서 사용한다. 만약에 초기에 모델이 redis에 없으면 새로 작성한다.

​        

### Creating different service types

다른형태의 서비스타입을 만드는 예제는 아래와 같다.

#### 

#### MODEL

- [A minimal skeleton for model source code](https://github.com/cliveseldon/seldon-core/tree/s2i/wrappers/s2i/python/test/model-template-app)
- [Example models](https://github.com/SeldonIO/seldon-core/tree/master/examples/models)

#### 

#### ROUTER

- [Description of routers in Seldon Core](https://github.com/SeldonIO/seldon-core/blob/master/components/routers/README.md)
- [A minimal skeleton for router source code](https://github.com/cliveseldon/seldon-core/tree/s2i/wrappers/s2i/python/test/router-template-app)
- [Example routers](https://github.com/SeldonIO/seldon-core/tree/master/examples/routers)

#### 

#### TRANSFORMER

- [A minimal skeleton for transformer source code](https://github.com/cliveseldon/seldon-core/tree/s2i/wrappers/s2i/python/test/transformer-template-app)
- [Example transformers](https://github.com/SeldonIO/seldon-core/tree/master/examples/transformers)

​    

## Advanced Usage

### Model Class Arguments

Kubernetes에 이미지를 배포시, SeldonDeployment에 정의된 파라미터에서 인자를 추가할 수 있다. 



예를들어  [Python TFServing proxy](https://github.com/SeldonIO/seldon-core/tree/master/integrations/tfserving) 에서 class 생성자는 다음과 같다.

```python
class TfServingProxy(object):

def __init__(self,rest_endpoint=None,grpc_endpoint=None,model_name=None,signature_name=None,model_input=None,model_output=None):
```



이러한 인자들은 Seldon Deployment에서 배포할 때, 설정할 수 있다. 

부분적으로 표시된 SeldonDeployment에 인수가 정의된  [MNIST TFServing example](https://github.com/SeldonIO/seldon-core/blob/master/examples/models/tfserving-mnist/tfserving-mnist.ipynb)에서 예제에서 이를 확인할 수 있다.

```json
 "graph": {
		    "name": "tfserving-proxy",
		    "endpoint": { "type" : "REST" },
		    "type": "MODEL",
		    "children": [],
		    "parameters":
		    [
			{
			    "name":"grpc_endpoint",
			    "type":"STRING",
			    "value":"localhost:8000"
			},
			{
			    "name":"model_name",
			    "type":"STRING",
			    "value":"mnist-model"
			},
			{
			    "name":"model_output",
			    "type":"STRING",
			    "value":"scores"
			},
			{
			    "name":"model_input",
			    "type":"STRING",
			    "value":"images"
			},
			{
			    "name":"signature_name",
			    "type":"STRING",
			    "value":"predict_images"
			}
		    ]
},
```

매개변수로 허용가능한 type은 [proto buffer definition](https://github.com/SeldonIO/seldon-core/blob/44f7048efd0f6be80a857875058d23efc4221205/proto/seldon_deployment.proto#L117-L131).에 정의된 값만 사용이 가능하다.

#### 

### Local Python Dependencies

`from version 0.5-SNPSHOT`

Python 의존 패키지를 설치하기 위해 private repository를 사용하고싶다면 다음과 같은 명령어를 이용한다.

```bash
$ s2i build -i <python-wheel-folder>:/whl <src-folder> seldonio/seldon-core-s2i-python2:0.5-SNAPSHOT <my-image-name>
```

해당 명령어는 PyPI에서 패키지를 찾아보고 없다면 local python wheel 파일을 `<python-wheel-folder>`에서  찾는다.

​    

### Custom Metrics

`from version 0.3`

custom metric을 response에 추가하고싶다면 optional method인 `metrics`를 클래스에 추가하고 return값을 metric dictionary를 갖는 list로 정하면 된다.

아래는 이에 대한 예시 스크립트이다.

```python
class MyModel(object):

    def predict(self,X,features_names):
        return X

    def metrics(self):
    	return [{"type":"COUNTER","key":"mycounter","value":1}]
```

custom metrics과 metric dictionary에 대해서 자세히 확인하고 싶다면 [here](https://github.com/SeldonIO/seldon-core/blob/master/docs/custom_metrics.md)를 참조하자.

​    

### Custom Meta Data

`from version 0.3`

custom meta data를 추가하고싶다면 class에 optional method인 `tags`를 추가하면 된다. tags method는 custom meta tags를 dictionary로 반환한다. 

아래는 이에 대한 예시 스크립트이다.

```python
class UserObject(object):

    def predict(self,X,features_names):
        return X

    def tags(self):
        return {"mytag":1}
```
