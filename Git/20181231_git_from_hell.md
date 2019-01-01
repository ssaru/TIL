# 지옥에서 온 git

​      

### tag

release는 git만들고 있는 소스코드에서 사용자에게 제공되도 되는 의미있는 코드 뭉치들을 모아놓은 곳이다. release에서 시멘틱 버저닝이 되어있는곳이 가르키고 있는 곳은 특정한 git의 commit 버전이다.

​    

### light weight tag

tagging을 위한 명령어는 다음과 같다.

```bash
$ git tage <version name> <commit name or branch>
```



위 명령어를 입력하게되면 해당 commit name에 대해서 version name으로 tagging이 되게 된다.



tagging 후에는 아래와 같은 명령어로 tag에 checkout이 가능해진다.

```bash
$ git checkout <tagging name>
```

​    

### annotated tag

tagging시 좀 더 상세한 tagging을 하고싶다면 `annotated tag`를 하면된다.



```bash
$ git tag -a <version name> -m <message>
```

위와 같이 입력하면 `-m <message>`에 대한 설명 및 tagging을 붙인 사람의 정보가 입력되게된다. `-m` 옵션을 제외하면 조금 더 상세한 설명을 입력할 수 있다.

​    

### tag push

일반적으로 `git push`로는 tag가 원격저장소에 push 되진 않는다.

다음과 같은 명령어로 tag를 push할 수 있다.

```bash
$ git push --tags
```



github의 기능으로 tagging작업을 지원하고 있으니, 이를 이용해도 된다.

 tagging시 versioning의 원칙은 [시멘틱 버저닝](https://semver.org/) 문서를 참조한다.



### tag의 삭제

위의 명령어를 입력하면 해당 tag를 삭제할 수 있다.

```bash
git tag -d <tag version>
```

​    

## tag의 원리

tag 명령어를 입력하면 `refs/tags/<tag version name>`이라는 파일이 새로 생기며, 해당 해쉬값은 `<tag version name>`의 commit head 해쉬값이 연결된다.