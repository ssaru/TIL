# 지옥에서 온 git



## git의 원리 (심화)

심화과정으로 내려가서 Git의 내부 원리에 대해서 알아본다.



### 3 way merge

3 way merge는 우리도 쓰고, Git 자체적으로도 사용을 함

merge시, Git이 자동으로 파일을 병합하게되는데, 이때 사용되는 병합 방법이 3 way merge이다.



여기서는 2 way merge와 3 way merge의 차이점에 대해서 설명하고, 3 way merge에 대한 장점과 충돌의 유형을 알아본다.



| Me(현재 branch) | Base 코드 | Other(다른 branch) | 2 way merge<br />(Me, Other만 참조하여 merge) | 3 way merge<br />(Me, Base, Other를 참조하여 merge) |
| :-------------: | :-------: | :----------------: | :-------------------------------------------: | :-------------------------------------------------: |
|        A        |     A     |                    |                  ?(conflict)                  |                                                     |
|        B        |     B     |         B          |                       B                       |                          B                          |
|        1        |     C     |         2          |                  ?(conflict)                  |                     ?(conflict)                     |
|                 |     D     |         D          |                  ?(conflict)                  |                                                     |



## 원격저장소 (Remote Repository)



- Local Repository와 대비되는 개념
- 다른 사람들과 협업 등을 위해서 Local Repository의 내용을 인터넷에 올릴 때, 인터넷의 저장소를 원격 저장소라 칭함
- 원격저장소는 코드를 백업한다는 의미와 다른 사람과 협업한다는 중요한 의미를 갖는다.

​    

## 원격저장소 생성



### 일반적인 원격저장소 등록 및 사용

```bash
$ git init
$ git remote add origin <reomte repository url> or <your bare local repository path>
$ git fetch

# push or pull for sync with remote repository
$ git push origin master 
OR
$ git pull origin master
```



### local device에서 원격저장소 생성

원격 저장소를 local device에 생성할 수 있다.

```bash
$ git init --bare remote
```



`--bare`옵션을 주면, local device에 remote 저장소와 유사한 파일이 생기게되는데, 해당 파일들의 구성을 확인해보면 `.git`파일들로만 구성되어있는 것을 확인할 수 있다.

이는 remote repository가 순수한 `Git`파일로만 구성하여 관리한다는 것을 확인할 수 있는 내용이다.

`--bare` 옵션을 주면, 해당 폴더 내에서는 어떠한 작업도 할 수 없게 제약이 걸린다는 것을 인지해야한다.



### 원격저장소 삭제

```bash
$ git remote remove origin
```



### Git Push 방법 설정

1. `git config --global push.default matching`

   : Git이 알아서 push를 하겠다

2. `git config --global push.default simple`

   : 사람이 명시적으로 어디에서 어디로 push를 하는지 결정함

3. `git push --set-upstream origin master`

   : 해당 branch에서 push 명령어만 사용할 경우에는 origin의 master로 push를 하겠다고 설정해주는 명령어