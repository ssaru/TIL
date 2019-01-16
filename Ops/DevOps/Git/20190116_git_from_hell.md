# 지옥에서 온 git

​    

## tagging

### Lighted weighted tag

tag를 했을 때, git내부에 어떤일이 생기는지 알아보자



tagging 시, `refs/tags/[tag version]`이라는 파일이 생긴다

해당 `refs/tags/[tag version]`은 특정한 objects를 가리키며, 이는 그 tagging시의 commit id이다.

​    

### Annotated tag

annotated tag는 위에서 설명한 Lighted weighted tag과는 조금 다르다.

`objects/[hash]/[hash]`라는 object가 생기고, 여기에는 tag message가 들어가있으며, `refs/tags/1.1.4`에는 annotated tag의 내용을 담고있는 object의 주소가 담겨있다. 

​    

## Rebase

rebase와 비교할 수 있는 기능은 merge다.

똑같이 병합하는 방법인데, 결과가 다르다.



merge와 rebase는  `branch를 분기해서 다른 branch가 수평으로 달리고 있다가 어느 한 지점에서 병합해야하는 구간`에서 큰 차이를 보인다.

​    

### merge

만약에 master의 내용을 feature로 가져오고 싶다하면 다음과 같은 명령어를 사용한다.

```bash
$ git checkout feature
$ git merge master
```



진행 후 노드는 feature와 master를 parents로 하고, 3-way merge를 통해서 병합하며 그게 불가능하다면 conflict를 발생시킨다.



그렇다면 해당 노드는 feature이며, feature 노드는 분기되었던 master의 노드들과 feature의 노드를을 모두 포함한다.

​    

### Rebase

feature와 master가 공통으로 가지고있는 최신의 parent node가 `base`가 된다. rebase라는 것은 이렇게 feature의 `base`를 현재 마스터가 가지고있는 최신 노드로 바뀌겠다는 명령이 된다.



다음과 같은 명령어를 입력하면

```bash
$ git checkout feature
$ git rebase master
```



- feature에서 생성된 노드들이 임시저장소(`temp`)에 모두 저장된다.
- master branch로 checkout된다.
- feature branch는 사라진다.
- 임시저장소(`temp`)에 저장되어있던 feature의 생성된 노드들을 최신 master branch와 병합한다.

​    

### Rebase와 merge 사이의 공통점과 차이

공통점

- merge의 결과의 내용와 rebase의 결과가 같다

차이점

- merge의 경우는 history가 병렬로 나타난다.
  - 쉽고, 안전하다
  - history를 보기 어렵다.
- rebase의 경우에는 history가 직렬로 나타난다.
  - 어렵고 위험하다.
  - history를 보기가 편하다