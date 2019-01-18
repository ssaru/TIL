# Stress Detection by Measuring Heart Rate Variability

사람의 스트레스 지표를 생체신호로 부터 획득하는 기술에 관해서 리서치 중임. 그 중 HRV라는 지표를 이용해서 사람의 스트레스 지표를 확인하는 논문에 대해서 리뷰하겠음

​    

## Abstract

- 건강에 `Stress`는 굉장히 중요한 지표다.
- 기존까지 `Stress`를 측정하고 해결하는 방법은 clinician의 경험을 통해서 해결하는 방법에 의존했었다.
- clinician의 경험으로 이를 측정하는 방법은 여러가지 문제로 인해 오탐지의 확률이 있다.
- `GSR(Galvanic Skin Response)`, `HR(Heart Rate)`, `Body Temperature`, `Blood Pressure(BP)` 등을 통해서 사람의 마음 상태에 대한 정보를 얻을 수 있다.
- 이러한 파라미터들은 사람의 나이 / 성별 / 경험에 따라서 사람마다 격차가 있다.
- 본 논문에서는 `Heart Rate Variability(HRV)`를 통해서 이를 해결하고자 한다.

​    

## Introduction

- 개개인의 스트레스를 탐지하고 계산하기 전에, 우리는 먼저 스트레스에 대한 개념을 알고 있어야 한다.

- 스트레스는 `어떠한 자극에 대해서 몸이 반응하는 방법`이라고 정의할 수 있다.

  > A way in which a body responds to any kind of demand

- 스트레스는 아래와 같은 종류의 스트레스가 존재한다.

  - 생존 스트레스; Survival Stress
  - 내부적인 스트레스; Internal Stress
  - 환경적인 스트레스; Environmental Stress
  - 피로 스트레스; Fatigue & Overwork

- 스트레스가 얼마나 중요한지 알고 있으나, 스트레스 레벨을 지속적으로 모니터링하는게 불가능하다.

- 본 논문에서는 stress level을 탐지하고 계산하는 device를 소개한다.

​    

## Literature survay

### A. Definition and mechanisms of heart rate variability

- ANS(automatic nervous system)은 사람의 몸을 제어하고, 안정적인 상태를 유지하는 메카니즘이다. 이는 2가지 주요 부분으로 나뉜다.; SNS / PNS
- SNS(sympathetic nerve system)은 몸이 위협에 대응해서 싸울지 도망갈지 반응을 준비한다.
- PNS(parasympathetic nerve system)은 SNS와 다르게 몸의 상태를 안정적인 상태로 돌려주는 역활을 한다.
- SNS는 심장 박동을 증가하게 한다.
- PNS는 심장 박동을 낮춰준다.
- HRV는 동정 및 미주 신경의 활동을 반영하는 비침습적 심전도 측정기
- HRV는 HR과 RR 간격 모두의 변화량을 묘사합니다.



![fig1](https://user-images.githubusercontent.com/13328380/51367901-0eab5380-1b30-11e9-9a0f-25850a3419af.PNG)



​    

### B. Measurement of HRV

- 심박 신호는 automatic tone인해 sinus variation이 생긴다. 이로 인해 ARV 측정은 다양하고 연속적인 RR 간격으로 구성되어있는 신호를 측정해야하는 어려움이 있다.

- HRV는 일반적으로 24시간 Holter 기록을 기본적으로 하거나 심전도를 이용해서 0.5~5분동안짧은 구간을 측정해서 사용한다.

- HRV triangular index는 density distribution의 적분을 maximum density distribution으로 나누는 것을 통해서 구할 수 있다.

- 추정된 값은 다른 scale에서 NN interval을 측정하는 것을 이용해서 발견될 수 있다.

  *(???)*

​    

### C. Role of neural network

- Neural Network은 2가지 Layer를 둔다.

  - Kohonen Layer
  - Grossberg Layer

- Kohonen Layer는 받은 가중치를 받은 입력값을 모두 더한다.

  > $z = \Sigma{Wx}$

- Grossberg Layer는 자신과 active된 kohonen neuron 사이의 가중치를 출력한다.



#### Why Two Different Types of Layers?

각각의 Layer는 목적이 다르다.

- Kohonen layer는 입력을 별도의 클래스로 분류한다. 
  - 만약에 input이 같은 클래스라면 같은 kohonen neuron이 activation될 것이다.
- Grossberg layer는 가중치를 조절하여 각 클래스에 적합한 출력을 얻는다.



Neural Network model은 스트레스를 탐지하는데 사용한다.



![fig3](https://user-images.githubusercontent.com/13328380/51370324-1b34a980-1b3a-11e9-9642-c51c33f039e0.PNG)



​    

### D. Time domain analysis

- ECG를 24시간 동안 측정시 sinsus depolarization이나 instantaneous heart rate로 인해 `normal RR interval`과 `QRS complex`가 감지된다.
- 통계적으로 time domain은 beat-to-beat interval과 인접한 NN interval의 차이로부터 파생된 interval로 나뉜다.
- `SDNN`은 HRV의 전역적인 지표다. `SDNN`은 변동성을 위해서 책임을 갖는 long term components를 반영한다.
- `SDANN`은 24시간 동안 평균 5분 간격의 변동성 지표다.
- `SD`는 HRV의 밤과 낮의 변화를 고려해서 반영한다.
- `RMSSD`와 `PNN50`은 interval에서의 차이를 기반으로하는 공통된 파라미터다
  - 이 파라미터는 주야에 의존받지 않으며 단기 HRV변화에 반응한다.
  - `RMSSD`가 조금 더 안정성이 있어서 진료시에는 `RMSSD`를 주로 사용한다.

![table1](https://user-images.githubusercontent.com/13328380/51371732-5e911700-1b3e-11e9-9f14-8b553823e3fb.PNG)



​    

### E. Geometric methods

- 파생 / 구성된 NN interval의 시퀀스가 변화하는 것은 geometric method를 제공한다. 
- HRV에 접근이 가능한 유효한 geometrical 형태가 있다.
  - 24-hour 히스토그램
  - 수정된 HRV triangular index
  - NN interval histogram의 triangular interpolation
  - Lorendz나 Poincare plot기반의 접근
- 24시간 RR interval variation과 탐지된 RR interval의 전체 수 사이의 관계는 24-hour histogram을 통해서 접근할 수 있다.
- triangular HRV index는 삼각형 모양의 histogram의  measure peak을 가지고있다. 
  -  높이는 빈번하게 관측된 RR interval의 duration과 일치한다. 
  - width는 RR interval의 변동성의 양이다.
  - area는 구성된 모든 RR interval의 수 이다.
- geometrical method는 기록된 데이터의 품질 영향을 덜 받으며, 쉽게 얻을 수 없는 통계 파라미터를 대체할 수 있다.

​    

### F. Frequency domain analysis

- 다른 주파수와 진폭으로 분해된 주기적인 심박 신호는 frequency domain analysis를 이용해 분석할 수 있다.
- 주파수영역에서의 분석은 심장의 sinus 리듬에서 intensity의 상대적인 양을 알 수 있다.
- power spectral analysis는 다른 색깔과 wavelength를 얻기위해서 백색광이 프리즘을 통과시키는 방법으로 얻는다.
- power spectral analysis는 2가지 방법으로 수행할 수 있다.
  - non-parametric method 
    - FFT
  - parametric method
    - auto regressive model estimation
- parametric method는 복잡하고 모델에 대한 적합성을 평가해야하지만 FFT같은 non-parametric method는 간단하고 빠르게 구현할 수 있다.



![table2](https://user-images.githubusercontent.com/13328380/51374076-2e994200-1b45-11e9-947c-f60d86d8b119.PNG)



![table3](https://user-images.githubusercontent.com/13328380/51374077-2f31d880-1b45-11e9-9d2b-fab9ba5aa1b6.PNG)



    ## Architecture

