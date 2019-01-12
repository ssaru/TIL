# Travis-CI

> **Travis CI** is a hosted, distributed[[2]](https://en.wikipedia.org/wiki/Travis_CI#cite_note-2) [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration) service used to build and test software projects hosted at [GitHub](https://en.wikipedia.org/wiki/GitHub).[[3\]](https://en.wikipedia.org/wiki/Travis_CI#cite_note-3)



Travis CI는 오픈소스 커뮤니티를 위한 지속적 통합(continuous integration:CI) 서비스이다. Github에서 관리되고 있다.



Github으로 오픈소스가 활발해지고, 다양한 환경과 문화가 구성되었으나, 그 중에 빠져있는 것이 CI 서버였다. 그리서 오픈소스로 개발을 하려고하더라도 지속적인 통합을 위해서는 자체 서버에 허드슨이나 젠킨스를 두어야했다. Travis CI는 오픈소스의 이러한 인프라 공백을 메워주는 서비스다.

​    

## Travis CI 시작하기

Travis CI의 사용법은 [Travis CI Tutorial](https://docs.travis-ci.com/user/tutorial/)문서에 잘 나와있으니 참조하면 좋다.

Travis CI는 Github과 연동해서 동작하기 때문에 Github 아이디가 있어야한다. 만약 없다면 하나 생성하도록 하자.



Travis CI 홈페이지에서 Github 아이디로 로그인한 후 아래와 같은 작업을 진행해준다.

- green *Activate*을 클릭한다.
- Travis CI로 관리할 Repository의 범위 혹은 특정 Repository를 설정한다.



설정이 완료되면 아래와 같은 화면을 볼 수 있다.

![travis-ci_initsetting](https://user-images.githubusercontent.com/13328380/50836080-af18af80-139b-11e9-8b8f-3732c5a57a99.png)

​    

## Travis CI 설정하기

특정 내 Git Local Repository에서 `.travis.yml` 파일을 다음과 같이 생성한다.

```yaml
language: python
python:
    - "3.6"
    
# command to install dependencies
install: "sudo pip3 install -r requirements.txt"

before_script:
    - flake8 --count --exclue ./tests,./doc --ignore E402,F401,E501

# commnad to run tests
script: python3 manage.py test lists/
sudo: required
```



해당 옵션에 관련된 내용은 다음과 같다.

| language      | 해당 프로젝트의 주 언어를 설정한다.                     |
| ------------- | ------------------------------------------------------- |
| python        | 특정 프로그래밍 언어의 버전을 명시한다.                 |
| install       | 의존성 패키지를 설치하는 명령어를 입력한다.             |
| before_script | 테스트를 수행하기 전에 사용자 정의 스크립트를 실행한다. |
| script        | 테스트를 수행한다.                                      |
| sudo          | 실행 권한을 조정한다.                                   |

​    

실제 사용방법에 맞춰서 위와 비슷하게 설정하고 Git으로 해당 Repository에`.travis.yml`파일을  `push`하면 Travis CI 해당 코드에 대해서 빌드 테스트를 진행한다.

다음은 테스트화면에 대한 Travis CI의 결과이다.



![build_result](https://user-images.githubusercontent.com/13328380/50836639-053a2280-139d-11e9-8927-f3e4151094f5.png)

​    

## Travis CI Badge

Travis CI로 해당 Repository 테스트를 진행한다면, 진행 결과를 README에 badge를 달아서 이를 표현할 수 있다.



아래 스크린샷에서  `ssaru/travis-ci-with-flake8` 오른쪽에 `build/unkwnon`을 클릭하면 다음과 같은 화면이 뜨는 것을 확인할 수 있다.

![build_result](https://user-images.githubusercontent.com/13328380/50836639-053a2280-139d-11e9-8927-f3e4151094f5.png)



**Source Image**의 `url`을 복사해서 README.md파일에 `![Travis CI](url)` 의 형태로 넣어주면, Travis CI의 badge를 확인할 수 있다.

![badge](https://user-images.githubusercontent.com/13328380/50837104-2a7b6080-139e-11e9-9ae0-0bef2910718c.png)



정확히 Travis CI의 badge를 README.md파일에 썼다면 아래와 같은 결과를 확인할 수 있다.

![badge_result](https://user-images.githubusercontent.com/13328380/50837201-6dd5cf00-139e-11e9-8229-3c80cbb1f017.png)