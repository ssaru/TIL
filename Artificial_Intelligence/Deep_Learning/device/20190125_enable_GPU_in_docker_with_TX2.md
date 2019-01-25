# How to Enable GPU in Docker Container with TX-2  

 `nvidia-docker`에서는 아직 TX-2를 지원하고 있지 않음.

일반적으로 Tegra device에서 사용하는 NVIDIA driver는 외부 GPU를 사용하는 일반적인 시스템의 driver와 NVML과 같은 라이브러리가 굉장히 다름.

> nvidia-docker는 NVML을 사용하는데 Tegra device는 NVML을 지원하지 않는 듯.



다행하게 GPU를 사용하게 하기 위한 특정 라이브러리를 container에 파라미터로 통과시킬 수 있고, 이를 이용하여 Docker에서 TX-2의 NVIDIA driver를 사용할 수 있음.



## Device parameters

Docker container가 device에 접근하기 위한 파라미터는 다음과 같음

- `/dev/nvhost-ctrl`
- `/dev/nvhost-ctrl-gpu`
- `/dev/nvhost-prof-gpu`
- `/dev/nvmap`
- `/dev/nvhost-gpu`
- `/dev/nvhost-as-gpu`



위의 파라미터는 `--device`라는 명령어를 통해서 넣을 수 있음

​    

#### Drivier Library Files

Container는 driver에 붙을 수 있어야하는데, Tegra는 `/usr/lib/aarch64-linux-gnu/tegra`에 위치함. `-v`통해서 container에 해당 경로를 추가해야함



`/usr/lib/aarch64-linux-gnu/tegra`에는 CUDA application에 의해 동적으로 로드될 수 있는 라이브러리를 포함하고 있음. 따라서 해당 path는 Dockerfile에서 `LD_LIBRARY_PATH`라는 환경변수에 입력되어야함

​    

## Dockerfile

​     

#### Compile deviceQuery 

cuda를 설치하면 예제로 주는 **deviceQuery**라는 코드를 빌드하여 예제로 사용하자.

위치는 `/usr/local/cuda/samples/1_Utilities/deviceQuery`에 있다.

```bash
$ cd /usr/local/cuda/samples/1_Utilities/deviceQuery
$ sudo make
```



위의 명령어를 치면 `deviceQuery`라는 실행파일이 생긴다.

해당 폴더 위치에서 `Dockerfile`을 아래와 같이 생성한다.

```dockerfile
FROM arm64v8/ubuntu:16.04

ENV LD_LIBRARY_PATH=/usr/lib/aarch64-linux-gnu/tegra
RUN mkdir /cudaSamples
COPY deviceQuery /cudaSamples/

CMD /cudaSamples/deviceQuery
```

​    

#### Docker Image build

아래의 명령어로 작성한 `Dockerfile`을 빌드한다.

```bash
$ docker build -t device_query .
```

​    

#### Run

아래 명령어로 작성한 도커 이미지를, Tegra GPU driver에 물려서 실행한다.

``` bash
$ docker run -it --device=/dev/nvhost-ctrl --device=/dev/nvhost-ctrl-gpu --device=/dev/nvhost-prof-gpu --device=/dev/nvmap --device=/dev/nvhost-gpu --device=/dev/nvhost-as-gpu -v /usr/lib/aarch64-linux-gnu/tegra:/usr/lib/aarch64-linux-gnu/tegra device_query /bin/bash
```

​    

#### Test

다음과 같은 명령어로 `deviceQuery`실행파일이 있는 디렉토리로 들어간 다음에 GPU device에 대해서 Query를 날리면 다음과 같이 docker container안에서 GPU를 확인할 수 있다.



```bash
$ cd /cudaSamples
$ ./deviceQuery
```



![docker_gpu](https://user-images.githubusercontent.com/13328380/51730493-0c5e7180-20bb-11e9-9cb4-7034f25f6c33.png)







## Reference

[[1]. Tegra-Docker](https://github.com/Technica-Corporation/Tegra-Docker)