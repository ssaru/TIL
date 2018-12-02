# 지옥에서 온 git



## git의 원리 (심화)

심화과정으로 내려가서 Git의 내부 원리에 대해서 알아본다.



### Branch의 원리

Git의 내부가 브랜치를 사용하는 것과 동시에 어떻게 변화하는지 알아보자



- git을 init(초기화)하게되면 `HEAD`라는 파일이 생김.

- `HEAD`라는 파일은 다음과 같은 파일을 참조하고있음

  - 예) ref: refs/heads/master

- 코드를 생성 후, commit을 하게되면 `refs/heads/master`라는 파일이 생김

  - 해당 파일을 클릭하면, 특정한 hash id를 가지고 있음
  - hash id를 쫒아서 들어가면, commit을 가르키는 내용이 적혀있음
  - 각 commit은 parents를 갖기 때문에, 이 parents를 통해서 이전 commit을 추적할 수 있음

- branch를 생성하게되면, `refs/heads/master`라는 파일 내용을 복사하여 `refs/heads/<branch>` 를 생성함

  - 즉, 브랜치 생성은 `refs/heads/master`라는 파일을 branch이름을 변경함과 동시에 복사하는 것을 의미함

- checkout을 하게되면, 현재 `HEAD`가 해당 branch name에 대응하는 `refs/heads/<brach name>` 파일을 참조함

  - checkout 할 경우 `HEAD` 파일 변화

    ```bash
    ref: refs/heads/master
    ->
    ref: refs/heads/<branch name>
    ```


### reset, checkout의 원리

Git에서 제일 중요한 기능이 과거로 돌아가는 기능인데, 여기에는 reset, revert명령어가 존재.

하지만 checkout 명령어를 통해서도 코드를 과거시점으로 돌릴 수 있음



- `reset` 명령어를 이용하면 코드를 과거로 돌릴 수 있음
  - `git reset --hard <commit hash id>`
  - commit이 이전 `HEAD`로 돌아감
  - 즉, `reset` 명령어는 `HEAD`라는 최신 commit 내용을 이전으로 돌려줌
    - 하지만 실제 Object file이나, commit 내용이 삭제되진 않음
    - 해당 내용은 `logs/refs/heads/<branch-name>` 혹은 `ORIG_HEAD`를 보면 확인할 수 있음
    - Git은 내용이 삭제될 수 있을 거 같은 위험한 명령이 떨어졌을 경우, 현재 상태를 `ORIG_HEAD`에 기록함
    - 실제로는 `logs/refs/heads/`를 보는게 더 효과적인데, 이를 직접보기 어렵기 때문에 다음과 같은 명령어를 이용한다.
      - `git reflog`
- `checkout` 명령어를 이용하여 코드를 과거로 되돌릴 수 있음
  - `git checkout <commit hash id>`
  - 이때, `HEAD`파일이 checkout한 `<commit hash id>`를 가르킴