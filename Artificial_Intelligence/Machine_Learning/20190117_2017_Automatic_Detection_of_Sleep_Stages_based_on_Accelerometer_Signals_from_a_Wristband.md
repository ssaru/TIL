# Automatic Detection of Sleep Stages based on Accelerometer Signals from a Wristband

Wrist band의 accelerometer signal data와 polysomnography(PSG) data를 기반으로 5가지의 machine learning 알고리즘을 사용하여 sleep stage를 detection한 논문

​    

## Abstract

- wrist band에서 취득한 accelerometer data를 이용하여 Machine learning 알고리즘을 학습시켜 sleep scoring측정하는 method를 제안한다.
- 8시간동안 PSG data와 wrist band에서 accelerometer data취득해 데이터를 구성했으며, 총 데이터의 개수는 36개
- 전처리 및 sleep stage를 추정하는 중요한 feature들을 추출함
- 5개 machine learning 알고리즘을 사용하여 sleep stage를 추정함
- validation을 위해서 sleep clinician에 의해서 평가된 PSG scoring과 비교함
- accuracy는 90% 이상, sensitivity는 50~80%

​    

## Introduction

- Polysomnography(PSG)는 수면장애를 진행하는 방법 중에 제일 정확한 방법론이다.
- PSG 측정에는 많은 불편함이 있다.
  - PSG 과정이 복잡하고 불편하므로, 환자가 스트레스를 받아서 측정하는데 영향을 준다.
- PSG를 대체하려는 연구가 몇 있었다. [1]
  - 복잡한 과정을 대체하기 어렵다.
- wristband의 accelerometer 데이터를 이용하여 이를 대체하는 방법을 제안한다.

![psg](https://user-images.githubusercontent.com/13328380/51290428-2c50be00-1a47-11e9-94cd-23e2e113daa7.PNG)    

​    

## Method

- PSG와 ACC데이터를 함께 얻는다
- fifth-order Butterworth filter를 이용해서 movement artifacts [2, 3]을 제거한다.
- 5개의 machine learning 알고리즘으로 4개의 sleep stage를 분류한다.
- information gain을 계산해서 sleep stage를 분류하는데 최적의 10개의 feature를 선별한다.

![fig1](https://user-images.githubusercontent.com/13328380/51290764-04ae2580-1a48-11e9-8ebe-f88747a11bd2.PNG)

​    

### Experiment

- PSG
  - EEG
  - EOG
  - bi-lateral tribials EMG
  - ECG
  - airflow(nasal thermistor)
  - chest and abdominal excursion
  - oxyhemoglobin saturation
  - three-axis accelerometer sensor placed on the non dominant wrist

> accelerometer data의 sampling frequency는 100Hz

​    

### Dataset

- N1, N2 stage 데이터는 본질적으로 비슷하며 light stage로 병합된다.
- N3는 Deep sleep으로 간주된다
- Sleep stage는 4가지 (wake, rapid, eye movement[REM], light, deep sleep)로 구분된다.

>  Fig2는 각 sleep stage의 epoch수에 대한 분포를 의미한다.



![fig2](https://user-images.githubusercontent.com/13328380/51291069-3a074300-1a49-11e9-9a52-d6bbb4a14f71.PNG)

​    

### Preprocessing

- movement artifact를 제거하기 위해서 fifth-order Butterworth filter를 사용(MATLAB R 2014a)
- bandpass filter의 cut-off frequency 는 0.25~3 Hz

​    

### Feature Extraction

- 데이터 길이는 30초로 구성된다
  - GT는 PSG 결과로 비교한다
- 각 channels별로 feature를 추출한다
- feature 종류는 아래 table2 참조하자.





![table2](https://user-images.githubusercontent.com/13328380/51291494-ad5d8480-1a4a-11e9-931f-8c18c00f303b.PNG)

​    



### Classification

Classification 알고리즘은 `KStar`, `Random committe`, `Random subspace(RS)`, `Random forest(RF)`를 사용했다.

​    

### Feature study

- Sleep score classification을 위한 최적의 feature를 찾기 위해 각 Feature에 대한 information gain 계산했다

  $InfoGain(Class, Feature) = H(Class) - H(Class | Feature)$

- x축의 band energy (BE)가 0.32으로 가장 큰 information gain을 갖는다.

- y축의 skewness가 0.05로 가장 낮은 information gain을 갖는다.

- Classification 성능은 다양한 feature가 많아지면 많아질 수록 성능이 좋아지나, 본 논문에서는 최적의 feature 값을 찾고자 했다.

![fig3](https://user-images.githubusercontent.com/13328380/51294669-57430e00-1a57-11e9-8b0a-765edc155237.PNG)

​    

![fig4](https://user-images.githubusercontent.com/13328380/51294689-76da3680-1a57-11e9-965c-8f0a0e7fd2f0.PNG)

## Result

- performance 측정을 위해서 `accuracy`, `sensitivity`, `specificity` 를 다음과 같이 계산했다.

  $Accuracy = \frac{TP + TN}{TP + FP + TN + FN}$

  $Sensitivity = \frac{TP}{TP + FN}$

  $Specificity = \frac{TN}{FP + TN}$

- PSG 추정값은 GT로 사용했다.



- wake / REM / deep sleep stage의 평균 accuracy는 90% 이상
  - light stage의 accuracy는 80%
- light / REM stage의 평균 sensitivity는 90%
  - deep / wake stage는 50%
- REM / deep / wage의 평균 specificity는 90% 이상
  - light stage는 70%



#### information gain이 가장 큰 10개의 feature를 사용한 결과

- accuracy / sensitivities / specificity의 결과는 전체 feature를 사용한 결과와 유사하다.

![fig5](https://user-images.githubusercontent.com/13328380/51295512-59a76700-1a5b-11e9-9f4b-32d7208a8ca2.PNG)

![fig6](https://user-images.githubusercontent.com/13328380/51295514-5a3ffd80-1a5b-11e9-84bb-6576ce466a3c.PNG)

![fig7](https://user-images.githubusercontent.com/13328380/51295513-5a3ffd80-1a5b-11e9-881e-2a7198204a15.PNG)

![fig8](https://user-images.githubusercontent.com/13328380/51295515-5a3ffd80-1a5b-11e9-92ca-b8defab12934.PNG)

![fig9](https://user-images.githubusercontent.com/13328380/51295516-5a3ffd80-1a5b-11e9-86eb-7aafa988252c.PNG)

![fig10](https://user-images.githubusercontent.com/13328380/51295517-5ad89400-1a5b-11e9-99fb-9b34bcc3ef65.PNG)

![fig11](https://user-images.githubusercontent.com/13328380/51295511-59a76700-1a5b-11e9-8b53-ed2bb3f9609c.PNG)

​    

## Conclusion

성능은 잘 나왔으나, 데이터 imbalance문제와, 실험실 환경에서 PSG 측정으로 인한 에러가 있을 것이라고 생각함.



​    

## Reference

[Polysomnography (PSG) - AKA Sleep Study or Sleep Test](https://chicagosleepapneasnoring.com/polysomnography/)