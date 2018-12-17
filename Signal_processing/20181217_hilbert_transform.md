# Hilbert Transform

Hilbert Transform 관련 Feature에 대해서 알아야할 일이 있어서 간략한 직관정도로 정리함    



## 복소지수 표현



<p align="center">

![1024px-euler s_formula](https://user-images.githubusercontent.com/13328380/50072131-73188180-0217-11e9-8cb8-c20336988533.png)

</p>

<center>
	[정현파 신호의 복소지수 표현]    
</center>



어떠한 정현파 신호 $cos\theta$가 있을 때, 이를 다음과 같은 복소지수로 표현이 가능함

- $ e^{j\theta} = \cos\theta + j\sin\theta $



복소지수로써 정현파 신호를 분석하자면, 다음과 같이 표현될 수 있음



<p align="center">

![complex-exponential-signal](https://user-images.githubusercontent.com/13328380/50072748-b83db300-0219-11e9-9ea6-961bdfc798a6.png)

</p>

<center>
	[정현파 신호의 복소지수 표현]    
</center>



우리는 이러한 복소지수 표현에서 다음과 같이 실수값만 취해서 사용을 함

- $\Re e (e^{j\theta})$



대표적인 예로 주파수 분석을 할 때, FFT를 주로 사용하는데, 주로 특정 주파수 대역의 Amplitude를 보는 실수부를 보게됨.

- $X(f) = \int_{\infty}^{\infty}{x(t) \space e^{-j2\pi ft}{dt}}$
- $e^{-j2\pi ft} = \cos(2\pi ft) - j\sin(2\pi ft)$ 이므로
- 결국 푸리에변환은 $x(t)$가 $\cos(2\pi ft)$와 $-j\sin(2\pi ft)$가 얼마나 닮았는지 알아보는 식이 됨. [(link)](https://wikidocs.net/6250)
- 여기서 $-j\sin(2\pi ft)$는 phase를 의미하는데, 개인적인 경험으로는 사용할 일이 거의 없었음
- 푸리에 변환같은 경우는 몇가지 특수한 경우(linear system 및 제약조건)에서만 사용할 수 있기 때문에 한계가 있음.



![fft](https://user-images.githubusercontent.com/13328380/50072948-4c0f7f00-021a-11e9-837a-4152f8ca3675.jpg)

<center>
    <a href="http://withrobot.tistory.com/217">
	[FFT 분석]    
    </a>
</center>

​    

## Hilbert Transform



어떠한 정현파 신호가 주어졌을 때, 해당 정현파 신호의 허수부 표현을 구하는 것이 Hilbert Transform

$\cos\theta$라는 정현파가 있을 때, 이의 복소표현은 $e^{j\theta} = \cos\theta + j\sin\theta$이다.



$\cos\theta$와 $\sin\theta$의 관계는 phase가 $ 90^{\circ}$차이가 남.



따라서 **Hilbert Transform**이라는 것은 어떤 신호의 phase를 $90^{\circ}$를 변경해주는 변환이라고 볼 수 있음

- $h(t) = \frac{1}{\pi t}$
- 자세한 유도는 [여기](https://wikidocs.net/4051)참조

​    

## Instantaneous attributes



Hilbert Transform을 사용하여 얻을 수 있는 Feature가 **Instantaneous attributes**인데, 종류는 다음과 같음

- Amplitude 
  -  $A(t)$
  - 포락선(envelop)을 의미
- Frequency
  -  $v(t) = \frac{1}{2\pi} \frac{d\theta(t)}{dt}$
  - 위상(phase)의 변화율
- Bandwidth 
  - $q(t) = \frac{1}{2\pi}|\frac{d}{dt}ln(A(t))|$
  - Bandwidth 크기
- Quality factor
  - $q(t) = \frac{v(t)}{2\sigma(t)}$
  - Bandwidth와 위상(phase)변화율의 비율
- Dominant frequency
  - $f_{d}(t) = \sqrt{v^{2}(t) + \sigma^{2}(t)}$
- Amplitude acceleration
  - $A_{c}(t) = \frac{d}{dt}ln(A(t))$
- Cosine of phase
  - $\cos\theta(t) = \frac{f(t)}{A(t)}$