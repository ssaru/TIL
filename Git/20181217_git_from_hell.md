# 지옥에서 온 git



## git의 원리 (심화)

심화과정으로 내려가서 Git의 내부 원리에 대해서 알아본다.



### reset으로 알아보는 working copy, index, repository

git reset 명령어는 `--soft`, `--mixed`, `--hard`, `--merge`, `--keep` 등이 있다.

주로 언급되는 명령어는

- `--soft`
- `--mixed`
- `--hard`

*`--hard`옵션이 위험하지만 제일 간단함*



git reset의 옵션에 따라서 삭제되는 영역이 다름

| Working directory<br />(실제 작업이 일어나는 곳)<br />_working tree_<br />_working copy_ | index<br />( `git add`시 포함되는 곳)<br />_staging area_<br />_cache_ | repository<br />(version이 `commit`되는 곳)<br />_history_<br />_tree_ |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              | `git reset --soft`                                           |
|                                                              | `git reset --mixed`                                          | `git reset --mixed`                                          |
| `git reset --hard`                                           | `git reset --hard`                                           | `git reset --hard`                                           |



`git reset --soft` 후에 다시 복원하고 싶다면, `ORIG_HEAD`을 이용하여 다시 복원할 수 있다.

*기본적인 `git reset`은 `git reset --mixed`로 작동하므로 `--soft`옵션을 명시적으로 준다.*

​    

### merge & conflict

충돌이 일어났을 때, git 내부적으로 어떻게 작동하는가를 살펴봄



각자 다른 브랜치에서 같은 파일을 수정 후에, `merge`를 수행하게되면 conflict가 남.

conflict시 `index`파일을 살펴보면, 한 파일에 대한 object파일 hash 뒤에 숫자값이 붙음

- 1: parents의 내용
- 2: A <- B로 merge시 A의 내용
- 3: A <- B로 merge시 B의 내용



git은 기본적으로 이 3가지 파일을 기준으로 자동으로 merging을 수행함



`MERGE_HEAD` : merge가 될 대상의 최신`commit`

`MERGE_MSG` : 다음 commit시 자동으로 생성되어 제공되는 `commit-log`

`ORIG_HEAD` : 병합 이전의 HEAD

`object/xx/hash` : 충돌이 난 파일의 구성



병합시 하나하나 수정하면서 진행할 수 있지만 병합을 도와주는 툴을 사용할 수 도 있음.

- `kdiff3`
- `bc`
  - 설치를 해야함
- `git config --global merge.tool kdiff3`
  - `merge`명령을 입력시 `kdiff3`를 이용하여 merge
  - 자동으로 merge tool로 등록

​    

`git mergetool`이라는 명령어를 통해서 충돌난 코드를 merge tool이 해결할 수 있게 함

- `kdiff3`사용시, `base`, `A`, `B`의 코드 중 어떤것을 채택하여 merge conflict를 해결할 것인지 결정하게 함
- `base`, `A`, `B` 모두 채택 가능
- 직접 수정하고 싶다면, `base`, `A`, `B` 중 하나를 선택한 후, 직접 수정 후 저장
- 저장 버튼을 누르는 순간 자동으로 merge가 완료됨



그 이후의 git의 동작을 살펴보면 

- 변경된 새로운 object가 생성됨
- index`는 변경된 파일의 object를 갖음
- `file.orig`는 만약의 사태를 대비해 백업된 파일