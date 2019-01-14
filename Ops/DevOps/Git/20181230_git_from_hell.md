# 지옥에서 온 git



## pull과 fetch의 차이점

​     

### git pull

원격저장소의 master와 지역저장소의 master가 똑같은 commit을 가르키게 만든다.

​     

### git fetch

원격저장소의 master는 최신의 commit을 가르키지만, 지역저장소의 master는 fetch 이전의 commit을 가르킨다.



즉, git fetch는 원격 저장소에서 파일을 가져오고, git pull은 원격 저장소에서 파일을 가져오고, HEAD 변경(merge)도 같이 해준다.

​    