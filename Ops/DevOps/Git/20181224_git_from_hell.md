# 지옥에서 온 git



## 원격 저장소와 지역 저장소의 동기화 방법(Github)



하나의 원격저장소를 이용하여 2개의 지역 저장소를 동기화하는 방법에 대해서 알아본다.



일반적으로 지역저장소가 2개 이상이라면, 하나의 지역저장소(*집*)에서 원격저장소로 push한 후, 다른 지역저장소*(회사*)로 이동하게 되면 무조건 처음으로 pull을 한 이후 사용해야한다.

​      

## ssh를 이용해서 로그인없이 원격저장소 사용하기(Github)

Secure Shell(ssh)를 이용하여 원격저장소에 접근하는 법을 알아본다. 



Github는 `Clone or download`버튼에서 `HTTPS` 방식과 `SSH` 방식을 지원하고 있다. `HTTPS`방식은 Id와 Passwd만 있으면 간단하게 사용할 수 있는 장점이 있다. 하지만 원격저장소에 접촉할 때마다 Id와 Passwd를 매번 입력해줘야한다는 단점이 있다. 만약 `SSH`방식을 사용한다면 별도의 Id Passwd의 입력없이 쉽게 사용할 수 있다.

*(SSH 통신방식은 자동로그인을 위한 수단이 아니며 단지 통신방식이 다를 뿐이고, 자동 로그인 기능이라는 편의기능을 제공할 뿐이다.)* 

​    

### SSH-keygen

```bash
# key의 저장 위치를 잘 확인한 후, Enter를 3번 쳐준다.
$ ssh-keygen
```

key 저장 위치에는 `id_rsa`, `id_rsa.pub`파일이 생겼을 것이다.

이 파일들은 `private key`와 `public key`를 의미한다.



![ssh](https://user-images.githubusercontent.com/13328380/50394840-93aa8380-07a3-11e9-9b14-15d6316bf887.PNG)



`private key`와 `public key`를 이용한 통신방식은 아래와 같다.



![method](https://user-images.githubusercontent.com/13328380/50394864-bfc60480-07a3-11e9-9529-57a582db3fb1.PNG)



우리가 접속하고자 하는 remote server를 상단, 우리가 작업하는 환경을 하단의 노트북이라고 가정했을 때, `public key`는 노트북과 remote server에 전송하고, `private key`는 본인이 보관한다. 



이를 이용하여 통신 내용을 `private key`를 이용해 암호화하고, 서버에서는 `public key`로 암호화된 통신 내용을 복호화 한다. 거꾸로 remote server에서 줄 내용은 `public key`로 암호화하고 내 노트북의 `private key`를 이용하여 해당 내용을 복호화한다. 



이렇게 특정 key로 암호화된 통신 내용은 반대 쌍의 key가 없으면 복호화 할 수 없다.



### private key copy하기

`cat` 명령어를 이용하여 `private key`내용을 출력하고 복사한다.

```bash
$ cat id_rsa.pub
```



- 복사를 완료했으면, `Github` 홈페이지의 `Personal setting`에서 `SSH and GPG keys`란에 들어간다.

-  `New SSH key`를 클릭하여, Title에 지역저장소의 이름을 적는다.

- `Key`란에 아까 복사했던 `private key`를 붙여넣는다.

- `Add SSH key`를 누른다.



이와 같은 절차는 아까 위에서 언급했던 

```
public key는 노트북과 remote server에 전송
```

와 같은 의미가 된다.

​    

### 새로운 저장소를 생성

`Github`에서 새로운 저장소를 생성했다면, `Clone or download`에서 `Use SSH`를 선택해서 주소를 복사한다.



복사를 완료했다면, `git clone`혹은 

```bash
$ git init
$ git remote add origin <repository ssh address>
```

를 진행한다.



이때, 실제로 연결할 것인지 안할 것인지에 대한 질문이 나오는데 `yes`를 입력하고 지나간다.

​    

### 편하게 작업하기

코드를 수정하고, commit하고 push를 진행해보면 따로 Login에 대한 질의 없이 곧바로 push가 되는 것을 확인할 수 있다.

