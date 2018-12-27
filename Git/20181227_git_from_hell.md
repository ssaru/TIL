# 지옥에서 온 git



## 자동 로그인 (My Server)

내가 만든 원격 Server에서 자동로그인 하는 방법을 알아보자



### SSH를 자동으로 로그인하는 방법

이를 하게되면, Git remote repository가 ssh로 연결하기 때문에, 이와 연결되어 자동으로 이를 할 수 있게 된다.



홈 디렉토리(`~`)에 들어가서 `ls -al`을 확인하면 `.ssh` 디렉토리를 확인할 수 있다.

여기서 `.ssh`에서 설정을 변경하여 자동로그인을 할 수 있다.(이때, `.ssh`폴더를 백업해두자)

백업이 완료되었다면 다음과 같은 절차를 따른다.



```bash
$ ssh-keygen -t rsa
> Generating public/private rsa key pair.
Enter file in which to save the key (/Users/study/.ssh/id_rsa):
Created directory '/Users/study/.ssh'
Enter passphrase(empty for no passphrase): <Enter>
```



해당 명령어를 입력하면 `.ssh`라는 폴더가 생기고, 그 안에 rsa 파일 2개(`id_rsa`, `id_rsa.pub`)이 생긴 것을 확인할 수 있다. 해당 파일에 대한 설명은 이전에 설명했으므로 생략한다.



`id_rsa.pub`키의 내용을 출력한 후, 이를 복사하여 원격 서버에 저장한다.

```bash
$ cat id_rsa.pub
> ssh-rsa AAAAB#...
```

`id_rsa.pub`의 내용이 출력된 것을 복사해서 원격 서버에 접속한 후, `.ssh` 폴더에서 `authorized_keys`라는 파일을 생성한 후 복사한 `id_rsa.pub`의 내용을 복사 붙여넣기 한다.



이를 완료했다면, ssh를 이용해 원격서버에 붙을 때, 더 이상 암호를 물어보지 않게된다.

```bash
$ ssh <host name>@<ip address>
```



`id_rsa.pub`을 조금 더 손쉽게 복사하고 싶다면 다음과 같은 명령어를 사용하면 된다.

```bash
$ ssh-copy-id <host name>@<ip address>
```



위의 과정을 모두 잘 진행했다면, git의 push / pull을 진행할 때, 더 이상 원격 서버의 비밀번호를 물어보지 않게 된다.

​    

## 원격 저장소의 원리

내부 저장소와 원격저장소가 상호작용을 할 때, Git에서는 어떤 작용이 일어나는가에 대해서 알아본다. 

​    

### git remote add origin `remote url`

이때, `config`파일 속 [remote "origin"]이라는 파일 정보가 등록되며, `url`라는 곳에는 원격 저장소의 url이, `fetch`라는 곳에는 원격 저장소 어디에서, 지역저장소 어디로 가져올 것인지에 대한 내용이 기록된다.



### git push --set-upstream origin master

- 지역 저장소의 master 브랜치와, 원격 저장소의 master 브랜치를 연결

  `config` 파일속에 [branch "master"]에 대한 정보가 새로 등록되며, remote 명과 remote에 merge될 브랜치 이름이 등록된다.

- 업로드

  `refs/remotes/origin/master`라는 파일이 새로 등록되며, 업로드 한 object 파일의 해쉬가 적혀있다. 지역저장소의 commit인 `refs/heads/master`와 내용이 같게 된다.



`git log --decorate --graph`를 확인하면 최근 commit의 해쉬의 우측 끝에 `HEAD -> master, origin/master`로 변경되어있는 것을 확인할 수 있다.

여기서 파일을 일부 수정하고, 지역저장소에 commit을 하게되면, `HEAD->master`와 `origin/master`의 위치가 서로 어긋나는 것을 확인할 수 있는데, 이는 지역 저장소는 commit이 완료되었으나, 원격 저장소에는 push가 일어나지 않아서 생기는 현상이다.

이를 표현 가능한 이유는 위에서 이야기했던 `refs/remotes/origin/master`와 `refs/heads/master`가 있기 때문이다.

​    