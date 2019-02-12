# Data Distillation: Towards Omni-Supervised Learning

​      

# Data Distillation

- Data distillation은 아래와 같은 과정을 따른다.
  - labeled data를 이용해서 모델을 학습
  - unlabeled data를 transformation시켜 multiple image를 학습된 모델에 통과한다.
  - prediction값을 ensembling을 통해서 하나의 label로 만든다.
  - labeled data와 generated label을 이용해서 다시 학습한다.

​    

## Multi-transform inference

- image data를 crop, scale 등과 같은 여러 transform 방식으로 변환한다.

​    

## Generating labels on unlabeled data

- 논문 저자들의 관찰의 핵심은 아래와 같다.

> 여러가지 prediction의 집합은 새로운 지식을 만들 수 있고, 이를 이용하여 label을 generating할 수 있다.

- Classification을 수행하는 모델의 경우, 여러 output vector들을 평균내서 사용하는 방법이 있다.
- 위와 같은 전략은 다음과 같이 2가지 문제가 있다.
  - 이는 *soft* label을 생성한다.  (*categorical label*이 아닌 *probability vector*)
    - *soft* label은 바로 이를 이용해서 model을 학습할 수 없다.
  - object detection이나 human pose estimation과 같은 기술들은 label을 average한다고 해서, 정확한 label이 생성되는 것이 아니다.
- 위의 2가지 문제를 고려해서 논문에서는 간단한 ensemble모델을 사용해 *hard* label을 생성한다.
  - ensemble 모델을 사용해서 나온 vector들을 NMS와 같이 merge하는 로직이 추가된다.

​    

### Knowledge distillation

- unlabeled data에서 *knowledge generate* 를 하는 새로운 방식은 model의 성능을 향상시킬 수 있다.
- model이나 loss function 아무것도 건들일 필요가 없지만, 2가지 요소를 고려했다.
  - minibatch에는 labeled data와 unlabeled data가 반드시 혼합되어있어야한다.
  - unlabeled 데이터가 추가됨에 따라서 training length가 더 길어져야한다.

​    

## Data Distillation for Keypoint Detection

​    

### Mask R-CNN

- Mask R-CNN keypoint detection variant를 사용함
- 첫번째 스테이지는 RPN으로 구성
- 두번째 스테이지는 bounding box classification, regression 그리고 각 RoI에 대한 key point predictor가 들어가있다.

​     

### Data transformations

- Data transformation에는 *geometric transformation*을 적용함.

- *Affine / Projective* Transformation을 한 경우에는 prediction을 merge하기 전에 *inverse* transformation을 해줘야함

- 본 논문에서는 유명한 2가지 transformation을 사용함

  - *scaling*

    - [400, 1200] pixel에서 step size를 100으로 결정함

      > 이는 resize로 인한 teacher model의 keypoint AP 변화를 고려한 값

  - *horizoontal flipping*

- test time시에는 baseline/distilled model에 transformation을 적용하지 않음

​    

### Ensembling

- 여러 transformation 방법을 사용했지만, 이는 모두 같은 RoI에서 나온 결과이다.
- 따라서 ouput의 probability값을 평균낼 수 있다.
- 이를 기반으로 제일 높은 확률값을 갖는(*argmax*) output을 취한다.

​     

### Selecting predictions

- 위와 같은 방법으로 좋은 label을 생성할 수 있지만, *false positives*가 포함될 가능성이 있다.
- 따라서 predicted detection score를 prediction quality로 사용하고, 특정 threshold값을 넘어가는 값에 대해서만 label을 생성한다.

- 본 논문에서는 score threshold값이 잘 작동하는 환경을 찾았는데, 이는 unlabeled image당 annotated instance의 평균 수다. 대략적으로 label image당 평균 instance 수와 같다.

  > 아직 이부분은 이해가 되지 않았다.
  >
  > => 일반적으로 occlusion이 일어났을 때, human pose estimation에서 keypoint의 개수가 고정된 range안에서 움직일텐데, 이의 개수를 이야기하는 듯 하다.

​    

### Generating keypoint annotations

- 각각 선택된 prediction은 $K$의 개별적인 keypoint로 구성되어있다.
- 개별적인 $K$값은 labeled된 data의 keypoint의 평균 개수로 선정하였다.

​    

### Retraining

- supervised image과 generated label image의 union set에서 student model을 학습함
- 2개의 다른 데이터를 고정된 sampling ratio를 이용해서 sampling한다.
- original image : generated labeled image는 $6:4$ 비율로 선택한다.
- training total iteration의 수를 증가한다.
- student model은 teacher model과 같은 아키텍쳐로 만든다.
- student model은 fine-tuned starting을 한다.
  - retraining 결과가 더 좋은 것을 확인함