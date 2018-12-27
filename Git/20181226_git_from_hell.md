# 지옥에서 온 git



## 자기 서버에 원격 저장소 만들기 (My Server)

나만의 원격 저장소를 만들어보자



### 원격 서버에 저장소 만들기

홈 디렉토리에서 원격 저장소를 만들 폴더를 생성한 다음에 

다음과 같은 명령어를 입력하여 원격 저장소를 만들어준다.



```bash
$ git init --bare remote
```

​    

## 로컬 저장소에서 내가 만든 원격 저장소에 연결하기

다음과 같은 명령어로 로컬 저장소에서 내가 만든 원격 저장소를 연결한다.



```bash
$ git remote add origin ssh://<user-name>@<ip address>/<remote repository directory>
```



해당 명령어가 잘 수행이 됬는지 확인하기 위해서 다음과 같은 명령어로 원격 저장소를 확인한다.

```bash
$ git remote -v
```



만약에 원격저장소에 있는 프로젝트를 클론하고싶다면 다음과 같은 명령어로 클론한다.

```bash
$ git clone ssh://<user-name>@<ip address>/<remote repository directory> <local repository directory>
```

​    

## push & pull (My Server)

git pull과 push는 My server와 Github은 다를게 별로 없다.