# flashing jetson in ConnectTech TX - 2 

보드레벨에서 딥러닝을 할때, 제일 쉽하게 접할 수 있는 보드는 [Nvidia TX-2 Develop kit](https://www.nvidia.com/ko-kr/autonomous-machines/embedded-systems-dev-kits-modules/) 인데, 생각보다 보드의 크기가 크다. [ConnectTech](http://connecttech.com/)란 회사에서 [Nvidia TX-2 Develop kit](https://www.nvidia.com/ko-kr/autonomous-machines/embedded-systems-dev-kits-modules/)의 TX-2보드에 따로 케리어 보드를 만들어 판매하고 있는데 상대적으로 크기가 많이 작은데(라즈베리파이 2개 포개놓은 정도) 이를 구매해서 Jetson을 올리는 과정을 기록한다.

​    

## 판매 링크

- [캐리어 보드(CTI Orbitty Carrier For NVIDIA Jetson TX2/TX1) &Jetson TX2 (Nvidia 임베디드 GPU 모듈)](https://www.wdlsystems.com/-NVIDIA-TX1-Integrated-Bundle_2)

- [Power Supply(DC 12V, 24W)](https://www.wdlsystems.com/CTI-Astro-65Watt-AC-DC-Power-Supply_4)

  110~240V AC input을 지원하나, 입력 인터페이스가 110v에 맞춰져있으므로 돼지코를 준비해야한다.

- [active heatsink](https://www.wdlsystems.com/CTI-NVIDIA-TX1-Module-Active-Thermal-Heatsink)

- micro SD card (ConnectTech 제품이 아닌 제품을 구매해도 상관없다.)

- [Enclosure](https://www.wdlsystems.com/CTI-OrbittyBox-Orbitty-Carrier-Board-Enclosure)

  하우징인데, 개발용도라면 꼭 구매해야할 필요는 못느껴 구매하진 않았다.

​    

## 구성사진

부품을 받아서 구매하게 되면 다음과 같이 생겼다.

![product](https://user-images.githubusercontent.com/13328380/50325706-b29de100-0529-11e9-9d5d-427d0f70713c.jpg)

​    

## Heatsink

heatsink 조립시에는 하단부의 스티커를 제거해준 후에, 같이 딸려온 4개의 나사를 빨간색 동그라미 부분에 장착해주면 된다.

![heatsink](https://user-images.githubusercontent.com/13328380/50325910-cc8bf380-052a-11e9-9b2e-f86f0ac58bb5.jpg)



Heatsink에 cooling fan을 연결하기 위한 커넥터가 있는데, 이는 측면부에 다음과 같이 연결해주면 된다.



![conenct heatsink](https://user-images.githubusercontent.com/13328380/50325909-cbf35d00-052a-11e9-89cd-4118a351c575.jpg)

​    

## Power line

보드를 받으면 (+, -)선으로 구성된 파워 커넥터가 구성품으로 있는 것을 확인할 수 있는데, 이는 Cooling fan을 연결했던 캐리어 보드 왼쪽부분에 극을 맞춰서 삽입 후, 일자드라이버로 조여주면 된다.

![kakaotalk_20181221_134728097](https://user-images.githubusercontent.com/13328380/50326035-62278300-052b-11e9-9ecd-140c10ee88c9.jpg)

​    

## Flashing Jetson into CTI Orbitty Carrier For NVIDIA Jetson TX2

조립이 완료되었다면, 이제 Jetson을 올려보자.

​    

### pre-requistes

- Ubuntu 16.04

  Ubuntu 18.04는 작동하지 않는 것을 확인했으며, Ubuntu 14.04는 확인해보지 못했다.

  마음 편히 진행하고 싶다면 Ubuntu 16.04에서 진행하길 추천한다.

​    

### Download JetPack 3.3

[JetPack 3.3](https://developer.nvidia.com/embedded/jetpack)을 다운받는다.

​    

### Execute JetPack 3.3

다운 받은 JectPack 파일을 실행한다.

```bash
$ sh JetPack-L4T-XX-linux-x64.run
```



만약 권한 문제로 실행이 안된다면 폴더 하나를 생성하여 실행파일을 폴더에 옮긴 후, 해당 폴더의 권한을 변경하고 실행하자

```bash
$ mkdir JetPack
$ cd Jetpack
$ mv JetPack-L4T-XX-linux-x64.run JetPack/
$ ls -ld
$ sh JetPack-L4T-XX-linux-x64.run
```



JetPack을 실행하면 다음과 같은 화면이 뜬다.

- TX2를 선택
- 설치할 파일을 선택(처음 시도한다면, default setting으로 두는 것을 추천한다.)



![jetpack1](https://user-images.githubusercontent.com/13328380/50326285-e3334a00-052c-11e9-8aa4-0697fe076a37.png)
![jetpack2](https://user-images.githubusercontent.com/13328380/50326286-e3334a00-052c-11e9-8efc-63ff35677a08.png)
![jetpack3](https://user-images.githubusercontent.com/13328380/50326287-e3cbe080-052c-11e9-999b-0e022370c580.png)



Next버튼을 누르면, 여러가지 파일을 다운받고 컴파일할 준비를 한다.

준비가 다 끝나면, 메세지창 하나가 뜨는데, 이를 확인하고 JetPack 설치화면을 종료한다. 

- 가끔 종료하다보면 Remove Download file이라는 체크버튼이 있을 건데, 여기에는 체크하지 않도록 한다.

​    

### Connect Tech Support Package Setup

[Connect Tech Support Page에서 Download란에서 L4T Board Support Package를 다운받는다.](http://connecttech.com/product/orbitty-carrier-for-nvidia-jetson-tx2-tx1/)

​    

**Unzip Support Package**

다운받은 L4T Board Support Pakage를 다음 위치로 옮긴 후 설치를 진행한다.

```bash
# move file
$ mv CTI-L4T-V###.tgz <JetPack_install_dir>/64_TX2/Linux_for_Tegra_tx2/
# unzip
$ tar -xzf CTI-L4T-V###.tgz
# install
cd ./CTI-L4T
sudo ./install.sh
```



### Flashing the Jetson

이제 보드에 OS를 Flashing하는 작업을 진행한다.

- 스마트폰을 충천할 때, 사용하는 5pin 커넥터를 보드에 연결
- 전원 연결
- 왼쪽 하단의 Recovery 버튼을 누른 상태에서, 바로 위의 Reset 버튼을 누른 후 손을 땜

![kakaotalk_20181221_134725666](https://user-images.githubusercontent.com/13328380/50326514-2b9f3780-052e-11e9-9d87-b839da415d04.jpg)



위의 과정까지 진행했다면 우분투에서 다음과 같은 명령어로 flashing 진행

- 설치 폴더로 진입

```bash
$ cd <JetPack_install_dir>/64_TX2/Linux_for_Tegra_tx2/
```

- 설치

```bash
$ ./flash.sh orbitty mmcblk0p1
```



설치가 성공적이면 다음과 같은 화면을 볼 수 있다.

![screenshot from 2018-12-18 12-29-25](https://user-images.githubusercontent.com/13328380/50326662-e2031c80-052e-11e9-85d8-b63dd57230c3.png)

​    

### 설치 확인

설치가 완료되었으면 모니터를 연결한 후 보드를 재부팅한다.

만약에 User passwd를 물어보면 기본 Username과 passwd는 다음과 같다

```bash
username : nvidia
passwd   : nvidia
```



![kakaotalk_20181221_134729553](https://user-images.githubusercontent.com/13328380/50326716-1d9de680-052f-11e9-9963-b98f5b3e780a.jpg)

​    

## Installing CUDA and Other Dependencies

위의 과정까지 진행했다면, Jetson은 설치하였지만 CUDA나 다른 의존성 패키지는 설치되지 않은 것이다.

이제 CUDA 및 의존성 패키지만 설치하면 된다.

보드와 우분투는 같은 라우터(공유기)에 연결하여 설치하는 방법을 추천한다.



보드의 Jetson에서 IP를 확인한다.

```bash
$ ifconfig
```



다시 JetPack을 실행한다.

```bash
$ sh JetPack-L4T-XX-linux-x64.run
```

JetPack 실행시, `Flash OS Image To Target`을 `no action`으로 변경한다.



![jetpack1](https://user-images.githubusercontent.com/13328380/50326285-e3334a00-052c-11e9-8aa4-0697fe076a37.png)

![jetpack2](https://user-images.githubusercontent.com/13328380/50326286-e3334a00-052c-11e9-8efc-63ff35677a08.png)

![jetpack5](https://user-images.githubusercontent.com/13328380/50326839-cb10fa00-052f-11e9-90a9-413bbe9447e0.png)



설정을 완료한 후에, Next를 누르면 IP를 입력하는 란이 나온다. 이때, 아까 확인해놨던 보드의 IP 및 보드의 Username과 User passwd를 입력해준다. 그리고 Next를 누르면 다음과 같이 진행사항을 확인할 수 있다.

- 만약 우분투 및 보드의 power setting이 5분에 sleep 모드로 전환되는 옵션으로 되어있어 설치 중 sleep 모드로 들어가게되면 설치가 중간에 끊기니, power setting에서 sleep 모드 off하는 것을 추천한다.

![jetpack6](https://user-images.githubusercontent.com/13328380/50326867-e3811480-052f-11e9-8978-aed454204f8b.png)

![screenshot from 2018-12-18 12-48-18](https://user-images.githubusercontent.com/13328380/50326889-f8f63e80-052f-11e9-8efa-77c269d10964.png)

​    

**CUDA 설치 확인**

보드를 재부팅하여 `nvcc` 컴파일러 버전을 확인할 수 있다.

![kakaotalk_20181221_134732031](https://user-images.githubusercontent.com/13328380/50326958-407cca80-0530-11e9-8e14-512858ec2b80.jpg)



## Reference

[[1] Jetson™ Flashing and Setup Guide for a Connect Tech Carrier Board](https://github.com/NVIDIA-AI-IOT/jetson-trashformers/wiki/Jetson%E2%84%A2-Flashing-and-Setup-Guide-for-a-Connect-Tech-Carrier-Board)

[[2] Orbitty Carrier for NVIDIA® Jetson™ TX2/TX2i/TX1](http://connecttech.com/product/orbitty-carrier-for-nvidia-jetson-tx2-tx1/)

