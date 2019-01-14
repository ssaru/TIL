# 지옥에서 온 git



## 원격 저장소를 지역 저장소로 복제(Github)



원격 저장소를 제공해주는 서비스들이 많은데(Bitbucket, Github), 그 중 Github을 이용하여 원격 저장소를 이용하는 방법을 알아본다.



## Clone

Github의 어떤 저장소에 들어가서 `Clone or downlaod`버튼을 누르게되면, 해당 저장소의 HTTPS 주소가 나오게 되는데 이를 이용하여 원격저장소를 지역 저장소로 복제하는 방법에 대해서 알아본다.



```bash
$ git clone <repository url> <local repository directory>

or

$ git init
$ git remote add origin <repository url>
$ git fetch
$ git pull origin master
```

​    

### 처음 로그 확인하기

저장소의 첫로그를 확인하기 위해서 `git log`를 거꾸로 해서 볼 수 있는데, 다음과 같은 명령어를 사용한다.

```bash
$ git log --reverse
```



이런 형태로 엄청 규모가 큰 코드를 log 단위로 쪼개서 흐름순으로 본다면 코드를 익히기 좋다.



## 원격저장소 만들기 (Github)



- Github에 Login을 한다.
- New Repository를 누른다.
- 저장소의 타입(Public / Private)을 선택한다
  - Public은 무료
  - Private은 유료
- Create Repository를 누른다.



`Quick setup`에서 HTTPS 주소를 확인한다. 해당 주소가 원격 저장소의 HTTPS의 주소를 의미한다.



### 빈 local repository 생성

```bash
$ git init
```

​    

### 원격 저장소 등록

```bash
$ git remote add origin <remote repository url>
```



#### 원격저장소 정보 확인

```bash
$ git remote -v
```



#### 원격저장소 삭제

```bash
$ git remote remove <remote repository name>
```

​    

### 원격 저장소 정보 가져오기

```bash
$ git fetch
```

​    

### 원격저장소 내용 합치기

```bash
$ git pull origin <branch name>
```

​    

### 원격저장소에 local repository staging

```bash
$ git status
$ git add <file>
$ git commit
```

​    

### local repository에 commit 내용을 원격저장소에 올리기

```bash
$ git push origin <branch name>
```



