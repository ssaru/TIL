# Packaging a Python model for Seldon Core using s2i

[source-to-image app s2i](https://github.com/openshift/source-to-image)을 이용하여 Python model을 Seldon-Core에 배포가 가능한 Docker Image로 wrapping하는 방법에 대해서 소개한다.



## Step 1 - Install s2i

Ubuntu 18.04 혹은 17.04에서 다음과 같이 설치하였다.



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

​    

### requirements.txt

의존성 패키지를 설치하기 위한 파일이며, `pip`를 이용하여 설치할 수 있음. 원하는 경우 setup.py로 대체할 수 있음

​    

###  .s2i/environment

모델을 wrapping하기 위한 Python builder image는 core parameter정의가 요구되며, 해당 정의의 예시는 다음과 같음

```bash
MODEL_NAME=MyModel
API_TYPE=REST
SERVICE_TYPE=MODEL
PERSISTENCE=0
```

*(해당 값은  이미지 빌드시 제공되거나 대체할 수 있음)*



## Step 3  - Build your image

