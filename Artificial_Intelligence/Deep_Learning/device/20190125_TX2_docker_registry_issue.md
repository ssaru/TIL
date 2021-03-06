# TX2 docker registry issue 

기본적으로 TX-2에는 docker가 설치되어있다.



`docker pull [image]` 명령어 입력시 다음과 같은 에러가 뜬다.

> Error response from daemon: Get https://registry-1.docker.io/v2: dial tcp: lookup registry-1.docker.io on 127.0.1.1.53: read udp 127.0.0.1:52427 -> 127.0.1.1:53 i/o timeout

​    



## 시도했던 방법



### /etc/resolv.conf 내용 변경

해당 파일을 열면 아래와 같이 적혀있는데 `nameserver`를 `8.8.8.8`로 변경 후, docker restart

```bash
# Generated by networkManager
nameserver 127.0.1.1
```

​    

```bash
# Generated by networkManager
nameserver 8.8.8.8
```



```bash
$ sudo service docker restart
```



### /etc/docker/daemon.json DNS 정보 변경



`/etc/docker`폴더에 들어가면 `daemon.json`이라는 파일이 없는데, 새로 만들어준 후에, 아래와 같은 내용을 입력한다.

```bash
{
    "dns" : ["8.8.8.8", "8.8.4.4"]
}
```



저장 후, Docker를 재시작한다.

```bash
$ sudo service docker restart
```



​    

## 해결

다음과 같이 docker image가 잘 받아졌다.



![result](https://user-images.githubusercontent.com/13328380/51729213-1467e280-20b6-11e9-81fb-bc08d309beb2.png)