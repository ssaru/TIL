# 지옥에서 온 git



## Git의 원리



### 3. Objects 파일명의 원리

내용이 같으면, Object의 같다는 의미는 내용을 기반으로 파일이름이 정해진다는 의미. 이번장에서는 해당 메카니즘이 무엇을 의미하는지 소개함.

`SHA1`  해쉬 알고리즘을 이용하여 입력을 파일 내용해서 출력결과를 얻으면, Object의 파일 이름과 같은 형태의 출력값을 얻을 수 있는데, 거기서 앞글자 2글자를 폴더 명으로 그리고 그 나머지를 Object 파일 명으로 사용한다.

> Note
>
> 실제로 Git은 SHA1 알고리즘을 이용하여 Object 파일 이름을 정한다.
>
> 하지만 내용만으로 이름을 정하진 않으며, 부가적인 정보를 이용하여 파일이름을 `SHA1`에 넣어 이름을 결정함



​        

### 4. commit의 원리

커밋 후에는 갱신되는 파일 목록이 5개가 된다.

`COMMIT_EDITMSG`, `logs/HEAD`, `logs/refs/heads/master`, `objects/XX/XXXX...XX`, `refs/heads/master`



#### 커밋메세지의 정보가 `objects/XX/XXXX...XX`에 저장된다.

일종의 커밋도 하나의 버전 혹은 파일처럼 다루어진다. : Commit도 Object다.

커밋 Object안에는 commit 내용과 누가 commit을 했는지 적혀있으며, `tree`라고 적혀있는 것 옆에는 object가 링크되어있다. 해당 `tree` object 내부를 살펴보면, commit한 `파일들의 이름`, `objects name`이 적혀있다.



파일을 수정하고나서 다시 commit을 해서 확인해보면 commit object안에는 `parent`라는 요소가 추가되어있고, 해당 내용에는 이전 commit 내용이 적혀있는 것을 확인할 수 있다.



중요한 것은 수정된 파일과 수정되기 전의 파일의 object name은 서로 다르다



해당 내용을 보고, 결론을 내보자면 Git은 특정한 시점의 코드 snapshot을 tree 구조로 가지고있는 것을 확인할 수 있다.

​    

### 5. status의 원리

여기서는 `index`라는 파일은 무엇이 status가 어떻게 작동하는지 지레짐작(?)해보기로 한다.



#### `git status`가 어떻게 추가할 파일이 있고 없는지를 알 수 있는가?

`index`파일에 연결되어있는 이전 commit과 현재 commit그리고 파일의 내용의 차이를 비교하면 차이가 없으면 추가할 파일이 있는지 없는지 알 수 있다.

또한 같은 이름의 파일의 해쉬값이 이전 commit의 해쉬값과 다르다면 내용이 바뀐 것으로 인지할 수 있다.

`git add`를 하면 `index`과 `object`파일이 갱신되어 현재 파일과 매칭이 되므로  staged 된 것을 확인할 수 있다.

(commit이 된 것인지 아닌지는 commit message의 여부를 통해서 알 수 있다.)