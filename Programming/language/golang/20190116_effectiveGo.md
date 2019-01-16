# Effective Go

업무상 golang을 사용해야해서, 기본적인 Golang 스터디를 진행한다.

​    

## Formatting; 포맷팅

많은 사람들이 같은 언어에서 다른 코딩 포맷팅(코드 컨벤션)을 적용해서 사용하고있고, 이는 사람들 사이에서 가장 큰 논쟁거리 중에 하나다. golang은 이를 최소화하기 위해서 모든 사람들이 같은 스타일을 고수하게끔 만들어서 이런 논쟁을 없애려고 시도하였다.



이를 해소하는 것이 `gofmt`이며, `gofmt`를 사용하면 코드를 go의 포맷팅에 맞춰서 변환해준다.

아래와 같이 `formatting.go`라는 파일을 작성하고, 다음 명령어를 이용하면 go의 포맷에 맞춰서 코드가 변경된 것을 확인할 수 있다.

```go
package main
type T struct{
name string//name of the object
value int// its value    
}
```



```bash
$ gofmt -w formatting.go
```



```go
package main

type T struct {
        name  string //name of the object
        value int    // its value
}
```



go의 표준 패키지들의 go코드는 `gofmt`로 포매팅되어있다.

몇가지 포맷팅에 대한 상세한 내용은 요약하면 다음과 같다.

- 들여쓰기

  > 들여쓰기를 위해 탭(tabs)을 사용하며, `gofmt`는 기본값으로 탭을 사용한다. 만약 꼭 써야하는 경우에만 스페이스(spaces)를 사용하라.

- 한 줄 길이

  > Go는 한 줄 길이에 제한이 없다. 길이가 길어지는것에 대해 걱정하지 마라. 만약 라인 길이가 너무 길게 느껴진다면, 별도의 탭을 가지고 들여쓰기를하여 감싸라

- 괄호

  > Go는 C와 Java에 비해 적은 수의 괄호가 필요하다. 제어 구조들(`if`, `for`, `switch`)의 문법엔 괄호가 없다. 또한 연산자 우선순위 계층이 간단하며 명확하다. 아래를 보자.

  ```go
  x<<8 + y<<16
  ```

  > 이는 다른언어와는 다르게 스페이스의 사용이 함축하는 바가 크다.

​        

## Commentary; 주석

- Go언어는 C언어 스타일의 /* ***/ 블럭주석과 C++스타일의 // 한줄(line) 주석을 제공한다
  - 한줄주석은 일반적으로 사용한다.
  - 블럭주석은 대부분의 패키지(package)주석에 나타난다



프로그램 및 웹서버이기도 한 `godoc`은 패키지의 내용에 대한 문서를 추출하도록 Go 소스 파일을 처리한다. 최상위 선언문 이전에 끼어드는 줄바꿈 없이 주석이 나타나면 그 선언문과 함께 추출되어 해당 항목의 설명으로 제공된다. 이러한 주석의 스타일과 유형은 `godoc`이 만들어내는 문서의 질을 결정하게 된다.



```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```



모든 패키지(package)는 패키지 구문 이전에 블럭주석형태의 패키지 주석이 있어야 한다. 여러 파일로 구성된 패키지의 경우,  패키지 주석은 어느 파일이든 상관없이 하나의 파일에 존재하면 되고 그것이 사용된다. 패키지 주석은 패키지를 소개해야하고, 전체  패키지에 관련된 정보를 제공해야한다. 패키지 주석은 `godoc` 문서의 처음에 나타나게 되니 이후의 자세한 사항도 작성해야 한다.



만약 패키지가 단순하다면, 패키지 주석 또한 간단할 수 있다.



```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```



- 별로 줄을 그어 쓰는 지나친 포맷은 주석에 필요없다. 
- 생성된 출력이  고정폭 폰트로 주어지지 않을 수도 있으므로, 스페이스나 정렬등에 의존하지 말라. 
- 포맷팅은 gofmt과 마찬가지로 godoc이  처리한다. 
- 주석은 해석되지 않는 일반 텍스트이다. 그래서 HTML이나 `_this_` 같은 주석은 작성된 그대로 나타날것이다. 그러므로 사용하지 않는것이 좋다. 
- `godoc`이 수정하는 한 가지는 들여쓰기된 텍스트를 고정폭 폰트로 보여주는 것으로, 프로그램 코드조각 같은 것에 적합하다. [fmt package](https://golang.org/pkg/fmt/)의 패키지주석은 좋은 예이다. 
- 상황에 따라, `godoc`은 주석을 재변경 하지 않을 수 있다. 그래서 확실하게 보기좋게 만들어야 한다. 정확한 철자, 구두법, 문장구조를 사용하고 긴문장을 줄여야한다
- 패키지에서 최상위 선언의 바로 앞에있는 주석이 그 선언의 문서주석으로 처리된다. 
- 패키지 내부에서 최상위 선언 바로 이전의 주석은 그 선언을 위한 문서주석이다. 프로그램에서 모든 외부로 노출되는 (대문자로 시작되는) 이름은 문서주석이 필요하다.
- 문서 주석은 매우 다양한 자동화된 표현들을 가능케 하는 완전한 문장으로 작성될 때 가장 효과가 좋다. 첫 문장은 선언된 이름으로 시작하는 한 줄짜리 문장으로 요약되어야 한다.



```go
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```



만약 모든 문서의 주석이 그 주석이 서술하는 항목의 이름으로 시작한다면, godoc의 결과는 grep문을 이용하는데 유용한 형태로 나오게 될 것이다. 만약 당신이 정규표현식의 파싱 함수를 찾고 있는데 "Compile"이라는 이름을 기억하지 못해 다음의  명령어를 실행했다고 상상해보자,



```bash
$ godoc regexp | grep parse
>
    Compile parses a regular expression and returns, if successful, a Regexp
    parsed. It simplifies safe initialization of global variables holding
    cannot be parsed. It simplifies safe initialization of global variables
```

만약 패키지 내부의 모든 문서주석이 "This function.."으로 시작된다면, grep명령은 당신이 원하는 결과를 보여줄 수 없을것이다. 그러나 패키지는 각각의 문서 주석을 패키지명과 함께 시작하기 때문에, 아래와 같이 당신이 찾고 있던 단어를  상기시키는 결과를 볼 수 있을 것이다.



Go언어의 선언구문은 그룹화가 가능하다. 하나의 문서주석은 관련된 상수 또는 변수의 그룹에 대해 설명할 수 있다. 하지만 이러한 주석은 선언 전체에 나타나므로 형식적일 수 있다.

```go
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```



그룹화는 항목 간의 관련성을 나타낼 수 있다. 예를 들어 아래 변수들의 그룹은 mutex에 의해 보호되고 있음을 보여준다.

```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

​    

## 명칭;Names

- 이름의 첫문자가 대문자이냐, 소문자이냐에 따라서 패키지 밖에서의 노출 여부가 결정된다.

​    

### 패키지명; Package names 

```go
import "bytes"
```

를 진행하면 `bytes.Buffer`를 사용할 수 있음



- package명은 소문자로 구성된 한 단어로만 구성

- 언더바(`_`)나 대소문자 혼용 금지

- 중복된 패키지명으로 충돌할 걱정은 필요없음

  - 패키지명은 파일명과 같음

  - 소스 디렉토리 이름 기반

    `src/encoding/base64`는 `encoding/base64`로 import

    - `encoding_base64`나 `encodingBase64`를 사용하지 않음

- `import .`표현 사용 금지



이러한 패키지 규칙 때문에 `ring.Ring`이라는 구조체의 인스턴스를 만드는 함수는 보통 `NewRing`으로 사용하지만 패키지 자체가 `ring`으로 불리기 때문에 이 함수는 그냥 `New`라고 부르고 `ring.New`로 사용한다.



*이를 통해서 golang이 얻고자 하는 것*

- *가독성*
- *코드의 구문을 작게 만듬*

*철학*

- *변수 이름을 길게 사용하면 가독성이 떨어지니 문서에 주석을 다는 편이 좋다.*

​    

### 게터; Getters

golang은 getters와 setters를 자체적으로 제공하지 않는다.스스로 getter와  setter를 만들어야한다.



- getter의 이름에 `Get`을 넣는것은 Go언어 답지않으며, 필수적이지 않다.

  만약 owner라는 필드를 가지고 있다면, getter 메소드는 `GetOwner`가 아닌 `Owner`로 사용하는 것을 권장한다.

- setter의 경우에는 `SetOwner`라고 사용한다.

  ```go
  owner := obj.Owner()
  if owner != user{
      obj.SetOwner(user)
  }
  ```



​    

### 인터페이스명; Interface names

관례적으로 하나의 메소드를 갖는 인터페이스는 메소드 이름에 `-er` 접미사를 붙이거나, 에이전트 명사를 구성하는 유사한 변형에 의해 지정됨

- Reader
- Writer
- Formatter
- CloseNotifier



이런 것들과 이를 통해 알 수 있는 함수 이름들을 지켜나가는 것은 생산적임



- 용법과 의미가 매칭이 안되는 메소드 이름을 사용하면 안된다.

  - 용법과 의미가 매칭이 같다면, 기존의 메소드와 같은 이름을 부여하면 된다.

    `ToString`이 아닌 `String`

​    

### 대소문자 혼합; MixedCaps

여러 단어로된 이름을 명명할 때, 언더바(`_`)대신 대소문자 혼합(`MixedCaps`나 `mixedCaps`)를 사용해라.











































