# 지옥에서 온 git



## Branch 수련



이번장에서는 [git 공식 홈페이지](https://git-scm.com/https://git-scm.com/)에서 Documentation의 Book란에서 [`3.2 Git 브랜치 - 브랜치와 Merge 기초`](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EC%99%80-Merge-%EC%9D%98-%EA%B8%B0%EC%B4%88)란을 보면서 이야기한다.



상황에 따라서 여러가지 브랜칭 방식이 있는데, 크게 `fast forward / not fast forward `로 나눌 수 있다.



### Fast Forward

 이런 상황을 가정해보자



- 기존의 master 브랜치에서 이슈가 생김
- `issue` 브랜치를 새로 팜
- 진행 중에, master 브랜치에서 긴급한 이슈가 발생(hotfix)
- `hotfix` 브랜치 생성
- `hotfix`가 해결되면, master 브랜치로 먼저 merge함



이때 master 브랜치에서 hotfix 브랜치의 내용을 merge하면 `Fast Forward`라는 로그가 발생

- `Fast Forward` : 빨리 감기



즉, master를 부모로 하는 issue / hotfix 브랜치가 있다고 했을 때, hotfix를 먼저 master에 머지하게 되면 master의 `HEAD`가 `hotfix`로 이동하게 되며, 이를 `Fast Forward`라고 함



`Fast Forward`방식은 master 브랜치가 hotfix 브랜치로 이동하기 때문에 별도의 commit을 생성하지 않는다.



### Recursive strategy

이제 `hotfix`가 해결되었으니, `hotfix`브랜치를 삭제하고, `issue` 브랜치의 작업을 계속한다.



`issue` 브랜치의 작업이 끝난 이후, `issue` 브랜치를 `master`에 merge해야하므로, 이를 진행하면 터미널 로그에 `merge made by the 'recursive' strategy`라는 로그를 남긴다.



`master`에서 떨어져나온 `issue`브랜치와 현재 `master` 브랜치의 `HEAD`가 다르므로 현재 `master`와 `issue` 브랜치는 merge할 수 없다. 따라서 다음과 같은 방법을 사용한다.



- `issue`브랜치와 `master` 브랜치의 공동의 부모를 찾는다.
- `3-way` merge를 통해서 현재 `master`브랜치와 `issue` 브랜치를 merge한다.
- 이 둘을 합쳤다는 별도의 commit을 생성한다.



`Fast Forward`방식은 commit을 생성하지 않지만, `Fast Forward`가 아닌 방식은 `merge, commit`이라는 방식을 사용함