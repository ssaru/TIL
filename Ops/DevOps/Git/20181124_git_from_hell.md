# 지옥에서 온 git



## Branch 병합 시 충돌 해결



Merge를 하는 과정에서 충돌이 나는 경우가 있다. 이런 경우에서 어떻게 해야하는지 알아본다.



## Branch 병합

`A` branch와 `B` 브랜치가 원활하게 작업을 다 하고나서, merge를 하게되면 commit log가 `Merge`로 뜨게된다.

수정한 파일들이 다른 이름이라면 Merge가 원활하게 잘 된다.



하지만 각 브랜치에서 같은 파일을 수정한 후, Merge를 시도하게 되면 무조건 충돌이 나게 된다.



다음과 같은 상황을 가정해보자

**`master` 브랜치의 common.txt**

```python
def b(){
    
}
```

​    

**`A` 브랜치의 common.txt**

```python
def a(){
    pass
}

def b(){
    pass
}
```

​    

**B 브랜치의 common.txt**

```python
def b(){
    pass
}

def c(){
    pass
}
```

​    

이 때, 이 3개의 브랜치를 `master` 브랜치로 병합해보자.

- `master` 브랜치에 `A`브랜치를 병합 (성공)
- `master`브랜치에 `B`브랜치를 병합(성공)



병합 후, 파일 구조는 다음과 같이 변하게 된다.



**병합 후, `master` 브랜치의 common.txt**

```python
def a(){
    pass
}

def b(){
    pass
}

def c(){
    pass
}
```



여기서 확인할 수 있는 것은 같은 파일을 수정하더라도 수정 위치가 다르면 merge하는데 무리가 없다는 것이다. 그리고 git은 이러한 자동 merge가 문제를 일으키는 경우가 거의 없다.



## Branch 병합 시 충돌

위에서 언급했던 것 처럼 같은 파일을 수정하더라도 수정 위치가 다르면 merge하는데 무리가 없다고 이야기했다. 하지만 같은 파일에 수정위치가 같아면 어떻게 될까?



다음과 같은 상황을 가정해보고 확인해보자



**`master`브랜치의 common.py **

```python
def a(){
    pass
}

def b(){
    pass
}

def c(){
    
}
```





**`A`브랜치의 common.txt **

```python
def a(){
    pass
}

def b(A){
    pass
}

def c(){
    pass
}
```



**`B`브랜치의 common.py **

```python
def a(){
    
}

def b(B){
    pass
}

def c(){
    pass
}
```



각 브랜치를 `master` 브랜치로 병합하게 되면 다음과 현상이 나타난다.

- `A` 브랜치를  `master` 브랜치로 병합

- `B` 브랜치를 `master` 브랜치로 병합

- 충돌이 나며 아래와같은 메세지가 출력된다.

  `CONFLICT (content) : Merge conflict in common.txt, Automatic merge failed; fix conflicts and then commit the result`



이 때, `git status`를 확인하면 다음과 같은 로그를 볼 수 있다.

```bash
$ git status

>>
Unmerged paths:
both modified: common.txt
```



여기서 `both modified`란 우리의 관심 파일이 양쪽에서 수정되었기 때문에 수정이 필요하다는 이야기다. 이를 수정하기 위해 **common.py** 파일을 열어보면 다음과 같은 코드를 확인할 수 있다.

```python
def a(){
    pass
}
<<<<<<< HEAD
def b(A){
=======
def b(B){
    pass
}
>>>>>>> B

def c(B){
    pass
}
```



총돌 후, 파일을 열었을 때, 해당파일에 적혀있는 심볼들이 중요하다.

- `=` : 이전 파일과 현재 파일의 충돌지점을 나누는 구분자
- `<<<<<<< HEAD` 원래 대상 파일의 코드 구성을 이야기한다.
- `>>>>>>> B` 병합하려고했던 파일 코드의 구성이다.



병합을 해결하기 위해 `HEAD`와 `특정 branch이름`에서 각각 작성된 코드를 적합한 방향에 맞춰서 수정하고, 수정 후에는 `<<<<<<<< HEAD`, `=======`, `>>>>>>>> B`라는 심볼을 삭제한다.

삭제 한 후, 다시 `git add / git commit / git push `를 수행한다.



## Stash

stash는 `감추다`라는 뜻이다.



stash를 사용하는 경우에는 작업 도중에 다른 branch로 checkout하는 경우가 있는데, checkout은 add/commit을 안하면 곤란한 상황이 발생하기 때문에 이때 stash를 사용한다. 



stash는 특정 브랜치에서 작업 중, HEAD를 최근에 commit된 곳으로 옮겨주므로 현재 작업내용을 숨기는(stash) 역활을 한다.



다음과 같은 상황을 가정하자

**`master` 브랜치의 common.txt**

```bash
a
```



`master` 브랜치에서 `checkout -b B`를 이용하여 `B`라는 브랜치를 생성/checkout 한다.

그리고 `B`브랜치에서 다음과 같은 파일 수정을 진행한다.

**`B` 의 common.txt**

```bash
a
b
```

해당 내용은 add / commit되지 않은 상황이다.



갑자기 급하게 `master` 브랜치에서 작업할 내용이 필요해 다시 `master` 브랜치로 체크아웃한다.

```bash
$ git checkout master
```



그 이후 `status`명령어를 이용하여 작업내용을 확인해보면 다음과 같은 결과를 확인할 수 있다.

```bash
$ git status
>>>
modified: common.txt
```



`B`브랜치에서 작업한 결과가 `master`로 checkout한 상태에서도 `master`에 영향을 끼치게 된다.

이것이 위에서 언급한 add/commit을 안한 상황에서 stash를 사용하지 않고 checkout을 하는 경우 곤란한 상황이라는 것이다.



## 사용하기



stash는 `--help`옵션을 통해서 명령어 리스트를 확인하면 다음과 같은 명령어들이 있다.

```BASH
$ git stash list
$ git stash show
$ git stash drop
$ git stash pop / apply
$ git stash branch
$ git stash save
$ git stash clear
$ git stash create
$ git stash store
```



여기서는 간단하게 작업하던 내용을 저장하는 용도로 사용하므로 `git stash` 또는 `git stash save`를 이용한다.



### Save

작업한 내용을 stash를 이용하여 저장해보자

```bash
$ git stash
>>
Saved working directory and index state WIP on B: 9161e43 1
HEAD is now at 9161e43 1
```



이후 `status` 명령어를 통해서 작업 내용을 확인해보자

```bash
$ git status
>>
On branch B
nothing to commit, working directory clean
```



이때, **common.txt**파일 내용을 확인해보면, 작업한 내용이 없는 것을 확인할 수 있다.



### apply

`stash save`를 통해서 현재 작업하던 내용을 적용하고, 다른 브랜치에서 급하게 진행해야한 일을 마무리했다.

이제 다시 작업하던 브랜치로 돌아와서 하던 작업을 이어서 해야한다. 



이 때, `stash`로 저장한 내용을 불러와 작업을 진행해야하는데, `git stash apply` 명령어를 이용하여 작업하던 내용을 불러온다.



```bash
$ git stash apply
```



git stash에 save된 작업 내용은 git stash 명령어를 이용해서 삭제하지 않으면 stash에 남아있기 때문에 `reset` 명령어로 HEAD를 초기화해도 stash를 이용해서 다시 불러올 수 있다.



### drop

`stash drop`은 최신의 저장된 stash가 삭제된다.

따라서 `apply`와 `drop`을 같이 이용하면 `save`된 stash를 불러오고 삭제할 수 있다.



### pop

`apply`와 `drop`을 순차적으로 작동시키는 것이 `pop` 명령어다.



stash는 working directory에 있는 변경사항을 감춘다. 이말은 즉, 추적되고있는 파일은 stash가 되지만, 추적되지 않는 파일은 stash가 되지 않는다. 이럴 때는 `git add`를 통해서 working directory에 올려주고 나서 stash하는 것이 좋다.