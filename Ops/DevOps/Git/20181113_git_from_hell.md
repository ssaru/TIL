# 지옥에서 온 git



## 버전관리의 본질



### 1. git이 관리할 대상으로 파일 등록

​    

#### 1-1). 파일 추적

특정한 코드를 수정한 후에, 해당 파일을 git이 추적하도록 만들게 하고싶다면 다음과 같은 명령어를 이용합니다.

``` bash
git add <file>
```



> Note
>
> git을 이용하여 여러 파일을 한꺼번에 추적할 수 있지만, 실제 사용시 불필요한 파일이 포함될 수 있으므로 파일 하나하나씩 추적하기를 추천합니다.

​    

#### 1-2). 추적상태 확인

git이 추적하고 있는 파일과 추적하고있지 않은 여러 기타파일의 상태를 확인하고 싶다면, 다음과 같은 명령어를 이용하여 상태를 확인할 수 있습니다.

```bash
git status
```



### 2. 버전 만들기 (commit)

#### 2-1). 버전에 포함될 버전을 만든 사람에 대한 정보 설정

해당 설정은 초기에 단 한번만 진행하면 됩니다.

```bash
git config --global user.name "<your name>"
git config --global user.email "<your email>"
```



#### 2-2). 버전 메세지

버전 메세지는 다음과 같은 내용을 적습니다.

- 코드의 어떤 변화가 있었는지
- 코드가 왜 변경되었는지



버전 메세지는 다음과 같은 명령어로 기록합니다.

```bash
git commit
```



#### 2-3). git log 확인

commit을 완료 후에, 다음과 같은 명령어로 git의 log를 확인할 수 있습니다.

(나올 때는 `q`를 누르면 됩니다.)

```bash
git log
```



### 3. Stage area

git이 항상 commit하기 전에 `add`를 요구하는 이유는 선택적으로 파일을 포함시키기 위해서입니다.

즉, `add`를 통해 Staged된 파일은 commit에 포함되지만 `add`되지 않은 not Staged 파일은 commit에 포함되지 않습니다.



여기서 Staged된 파일들을 모아둔 공간을 `Stage area`라고 합니다.



### 4. 변경사항 확인하기



#### 4-1). 로그에서 출력되는 버전 간의 차이점을 출력하고 싶을 때,

로그에서 출력되는 버전간의 차이점을 확인하고 싶다면 다음과 같은 명령어를 이용합니다.



```bash
git log -p
```



#### 4-2). 특정 버전간의 차이점을 비교할 때,

특정 버전간의 차이점을 비교하고 싶다면 다음과 같은 명령어를 이용합니다.



```bash
git diff '버전 id'..'버전 id'

>> git diff 3707468e8f8b6b1057a2ad2e4f91f61c299d38ae..c06deb1c198517bc826f227b2e2b5f7cf588deb4

diff --git a/hello.md b/hello.md
index 1901d47..82334af 100644
--- a/hello.md
+++ b/hello.md
@@ -402,7 +402,7 @@ $ dcf function list
 Function               Image                   Maintainer      Invocations         Replicas    Status     Description
 
-
+<U+200B>    

```



#### 4-3). git add하기 전과 add한 후의 파일 내용을 비교할 때,

`add` 명령어를 수행하고 나서, `add`한 작업과 그 이전의 작업이 어떤 차이가 있는지 차이를 확인하고 싶을 때, 다음과 같은 명령어를 통해서 확인할 수 있습니다.

```bash
git diff
```



### 5. 과거의 버전으로 돌아가기

commit을 취소하는 내용이며, 이전 버전 id로 돌아가고 싶을 때, 다음과 같은 명령어를 통해서 수행할 수 있습니다.



```bash
git reset --hard "버전 id"

VS

git revert "버전 id"
```



해당 명령어 `reset`과 `revert`의 작동은 비슷하게 보일 수 있으나, 실질적으로는 차이가 있으니 주의해서 사용해야합니다.



- `reset` : 특정 commit 버전을 파라미터로 넣으면, 해당 커밋 이후의 commit이 삭제된다.
- `revert` : commit을 취소하면서 새로운 버전을 생성



> Note 1
>
> 실제로 git에서 reset이나 revert를 통해서 특정 commit을 삭제할 수 있으나 실제로 git은 파일을 삭제하지 않으므로 이를 다시 복구할 수 있다

> Note 2
>
> 이미 원격 레파지토리에 push가 되어있다면 절대로 commit을 reset하면 안됩니다.



### 6. 명령의 빈도와 메뉴얼을 보는 방법

어떤 학습을 할 때, 자주 사용되는(빈도수가 높은) 것을 제대로 알고 나면 그 이후의 나머지 부분은 스스로 찾아서 공부할 수 있습니다.



따라서 git에서 자주 사용하는 명령어만 잘 파악하고 사용하는 방법을 알고 있으면 git을 사용하기 수월할 것입니다.



![git command statics](https://user-images.githubusercontent.com/13328380/48402642-79769200-e76f-11e8-8941-3fdc77c35bcc.PNG)



#### 6-1). git manual을 보는 방법

`--help`옵션을 통해서 최대한 활용하여 특정 command에 대한 사용법이나 이해에 대해서 알 수 있습니다.



```bash
git --help
git commit --help
git log --help

... 
```

