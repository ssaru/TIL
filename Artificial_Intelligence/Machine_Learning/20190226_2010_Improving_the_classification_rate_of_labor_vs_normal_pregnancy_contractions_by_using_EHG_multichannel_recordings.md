# Improving the classification rate of labor vs. normal pregnancy contractions by using EHG multichannel recordings

​    

# Abstract

- 같은 수축기동안 다른 장소에서 발행한 EHG 신호의 synchronization에 대한 대부분의 연구는 2개의 채널을 사용하는 것 때문에 한계점이 많았다.
- EEG에서는 Multichannel 테크닉이 광범위하게 적용되었으나, EHG신호에는 그러지 못하고 있다.
- 본 논문에서는 수축 검출을 위해 Multichannel EHG 신호를 사용한다.
- 2개의 채널과, 4x4 채널에서 출산과 일반 상황에서의 pregnancy contraction 구분하기 위한 phase synchronization의 성능을  비교한다. 
- phase synchronization을 측정하기 위해 *mean phase coherence*와 *phase entropy* 2가지의 index를 사용한다.

​    

# Introduction

- biological system의 기능을 이해하기 위해서는 signal간의 관계를 검출하는 것은 중요한 방법이다.
- EHG신호는 부드러운 자궁근육으로부터 산모의 복부로 전파되는 신호다.
- 여기서 관계;*relationship*이라고 하는 것은 신호간의 *correlation*, *synchronization*, *interdependency*, *coupling* 이라고 이야기하는 것과 같다.
- 신호의 관계를 검출하는 것은 EEG 신호를 분석하는 도메인에서는 널리 쓰이는 방법이지만[1], EHG신호에서는 드물게 쓰이는 방법이다[2].
- 최근까지 2개의 EHG신호간의 *correlation/synchronization(*예를들어 *linear correlation coefficient*[3], *wavelet coherence*[4], [5], *mutual information*) 검출하는 것에 대한 관심이 많아진 것을 확인할 수 있다.
- 언급한 연구들은 모두 2개의 채널에서 연구가 일어났지만, 최근에는 3x4 행렬의 bipolar 전극에서 기록된 bipolar signal에서  *nonlinear correlation coefficient*를 사용한 multichannel 분석을 수행하고 있다[7]. 해당 연구는 pregnancy에서 labor의 correlation이 증가했음을 보고하고있다.
- 2개의 신호의 phase들은 그들의 amplitude들이 uncorrelate 되어있어도 synchronize될 수 있다.
- 본 논문의 목표는 multichannel EHG 기록과 2 channel 기록에서 pregnancy와 labor사이의 phase synchronization를 비교하는 것이다.
- 임신상태 중이거나, 출산상태에서 측정된 EHG신호를 분류하기 위한 2가지 방법을 정량적으로 평가하기 위해서 ROC curve를 사용한다.