# Data Distillation: Towards Omni-Supervised Learning

​      

# Abstract

- Omni-supervised learning에 대해서 연구함
  - Omni-supervised learning이란 특정한 도메인에서 label data와 unlabeled data를 이용해서 semi-supervised learning을 하는 것을 이야기한다.
- 본 논문에서는 unlabeled data의 multiple transformation된 결과를 이용해서 단일 모델에서 ensembles prediction을 진행하며, 그 결과로 새로운 annotation을 generate한다.
- 해당 결과 덕분에, 실제 세상에서 self-training이 충분히 유효하게 작동할 수 있음을 확인했다.

​    

# Introduction

- 대부분의 *semi-supervised learning* 의 경우 완전히 label이 되어있는 dataset을 이용해서 label/unlabeld 데이터를 분리하고 실험했기 때문에 이는 *upper-bounded*되었다고 이야기할 수 있다.

  *(full annotated data는 dataset을 수집할 때, 부터 실험실 환경에서 제작되었으므로, 실험 조건이 실험실 환경과 크게 다를 바 없다는 것을 의미하는 듯 하다.)*

- omni-supervised learning은 unlabeled 데이터를 순수 인터넷환경에서 획득했기 때문에 *lower-bounded*되었다고 이야기할 수 있다.

- 실험 결과를 통해서, *fully supervised baseline*을 압도한 것을 확인할 수 있다.

- 본 논문에서는 *omni-supervised learning*을 하기 위해서 *data distillation*을 제안한다. 이는 이전에 *model*을 이용한 *knowledge distillation*에서 영감을 받았다.

- *data distillation*은 *unlabeled data*를 *multiple transformation* (flip, scaling)을 통한 후, *single model*을 통과시켜서 *prediction*결과를 얻은 다음 얻은 결과들을 *ensemble*모델을 통해서 *unlabeled data*의 *annotation*을 생성한다.

- 생성한 *annotation*을 이용해서 *student model*을 학습한다.

- *knowledge distillation*에서는 *unlabeled data*를 *multiple transformation* 을 수행한 것이 아닌, *unlabeled data*를 *multiple model*에 *prediction*해서 이를 *ensemble*했다면, *data distillation*은 역으로 *data distillation*을 했다고 보는 관점이다.



![fig1](https://user-images.githubusercontent.com/13328380/52413239-55cfa780-2b24-11e9-832b-dbc367141d07.png)

​    

- 이러한 *data distillation*은 *data cleaning heuristic*의 비용을 줄여주는 이점이 있다.
- *data distillation*을 적용해서 학습한 모델의 경우 2 point의 AP가 상승한 결과를 확인했다. 이는 수동으로 *unlabeled data*를 *labeling*해서 ~3 point의 AP를 올렸다는 연구결과와 비교했을 때, 고무적인 결과이다.

​    

# Related Work

- *Ensembling multiple model*에 대한 소개
- student - teach model 에 대한 소개
- self-training method에 대한 소개

- 서로 다른 class를 학습한 model들이 unlabeled data의 annotation을 생성하는 모델 소개
- multiple-geometric transformation을 이용한 key-point augmentation에 대한 내용 소개

​    

# Data Distillation

