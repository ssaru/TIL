# 지옥에서 온 git



## Branch 소개

파일의 내용이 수정될 때마다 일부의 버전을 수정해서 기존의 메인스트림에서 다양한 버전으로 관리해야할 때, 기존 소스코드의 흐름에서 여러가지 갈래로 개발하는 것을 Branch라고 한다. 



다양한 버전으로 관리하다가, 다시 병합해야하는 필요가 생기면 다시 메인스트림에으로 합칠 수 있다.

여기서 `브랜치를 생성` 한다는 것은 기존의 메인스트림에서 새로운 버전의 스트림을 생성한다는 것을 의마한다.



이전의 버전관리시스템에서는 Branch가 무겁거나, 위험하고 굉장히 안좋았었는데, Git은 이를 획기적으로 개선했기 때문에 Git에서 Branch의 개념은 굉장히 중요하다.



## Branch 만들기

Branch를 만들어 되야하는 경우는 다음과 같다



- 필요한 feature만 개발해야하는 경우
- 작업내용을 배포 및 테스트를 해야하는 경우
- rollback이 예상되는 코드 개발의 경우
- 기타



기본적으로 Git을 생성하면, `master`라는 Branch가 기본적으로 생성되게 된다.

다음 명령어를 치면 다음과 같은 화면이 콘솔창에 뜨는 것을 확인할 수 있다.

```bash
$ git branch
*master
```

- `*`는 현재 내가 바라보고있는 브랜치를 의미한다.



여기서 새로운 Branch를 생성하려면 다음과 같은 명령어를 이용하여, 브랜치를 분기한다.

```bash
$ git branch <branch name>

>> git branch feature
   git branch
*master
feature
```

- `git branch <branch name>`명령어를 입력하면, 입력한 브랜치 이름으로 새로 브랜치가 생기게된다
- 이를 `git branch`명령어를 이용하여 branch를 확인할 수 있다.



현재 코드의 흐름을 특정 branch로 이동하고 싶으면 다음과 같은 명령어를 이용하여 이동할 수 있다.

```bash
$ git checkout <branch name>

>> git checkout feature
master
*feature
```

- `checkout`이라는 명령어를 이용하여 해당 브랜치로 이동할 수 있다.



이미 작업이 진행된 브랜치로 `checkout`을 이용하면 폴더에 있던 코드의 구성이 변경된 것을 확인할 수 있다.

(초기 브랜치를 생성하게되면, 생성한 브랜치의 코드를 그대로 복사하게 된다.)



## Branch 정보 확인

특정 브랜치와 다른 브랜치와의 차이를 확인은 다음과 같은 명령어로 확인할 수 있다.



```bash
$ git log --branches --decorate
```

- `git log`는 해당 브랜치에 대한 log만 보여준다.
- `git log --branches`는 모든 브랜치에 대한 log만 보여준다.
- `git log --branches --decorate`각 브랜치의 각 branch의 `HEAD`의 commit을 확인할 수 있다.



이를 조금 더 시각적으로 편리하게 보려면 다음과 같은 명령어를 확인할 수 있다.

```bash
$ git log --branches --decorate --graph
```

- 이를 이용하면, graph형태의 commit을 확인할 수 있다. 이는 각 branch가 어떻게 흘러가는지 확인하기 쉽다.



commit의 내용이 크므로 전체적인 Git의 branch 흐름을 확인하고 싶다면 다음과 같은 명령어를 사용한다.

```bash
$ git log --branches --decorate --graph -oneline
```



이러한 방식의 콘솔에서의 시각화 방식은 우리에게 실제로 좋은 시각화를 보여주지 않기 때문에, branch의 흐름을 더 정확하게 보고 싶다면, sourcetree나 크라켄을 사용하면 조금 더 깔끔하게 시각화된 branch를 볼 수 있다.



버전과 버전 사이의 차이를 확인하고 싶다면, 다음과 같은 방식을 사용하면 된다.



**B라는 브랜치에는 있고 A라는 브랜치에는 없는 것을 확인하고싶을 때**

```bash
$ git log A..B
```



**A라는 브랜치에는 있고 B라는 브랜치에는 없는 것을 확인하고싶을 때**

```bash
$ git log B..A
```



이에 대해 소스코드까지 확인해야한다면, 다음과 같은 명령어를 사용한다.

```bash
$ git log -p B..A
```



각 commit별로 브랜치 사이의 차이첨을 확인하고 싶으면, `git diff`를 사용하면 된다.

```bash
$ git diff A..B
```



## Branch 병합

branch를 나누어서 작업하다가, 여러가지 branch를 하나의 branch로 병합(merge)하는 방법을 알아본다.



현재 branch의 현황은 위에서 소개한 명령어를 통해서 시각화할 수 있다.

```bash
$ git log --branches --decorate --graph -oneline
```



2개의 branch `A`와 `B`가 있다고 했을 때, `B`를 `A`에 병합하고 싶다면(`B->A`, 최종적으로 합쳐진 결과는 `A` 브랜치) 다음과 같은 명령어를 이용하여 병합(merge)한다.

```bash
$ git checkout A
$ git merge B
```

- 먼저 합쳐서 내가 남겨두고 싶은 branch로 checkout을 한다. (여기서는 `A` branch)
- checkout 후에 `merge` 명령어를 이용하여 `B` branch를 병합한다.



병합(merge)되고 난 후의 HEAD commit은 2개의 부모 commit을 갖는다.

- 원 branch `A` branch의 마지막 commit 
- 병합된(merge) `B` branch의 마지막 commit



병합이 된 후에 남은 branch `B`는 그대로 상태 그대로 남아있게 된다.

만약에 `B` branch도, `A` branch의 내용을 병합(merge)하고, 시각화해보면 branch `A`, `B`의 내용이 같아진 것을 확인할 수 있다.(하나의 line으로 표현됨)



작업이 완료 후, `B` branch를 삭제하고 싶다면 다음과 같은 명령어를 이용하여 branch를 삭제한다.

```bash
$ git branch -d B
```



