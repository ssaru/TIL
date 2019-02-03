# Clean Code [Chapter4]

디텍션 모임에서 프로젝트 진행시 Agile 및 CI, 전반적인 코드의 개선을 논의하고있음.

논의사항의 일부로써 다같이 Clean Code를 읽고 매 스프린트마다 Clean Code내용을 리뷰함



이번엔 클린코드 4장에 대한 내용을 간략히 리뷰

​    

## 주석

- 주석은 검증하기 힘들다.

- TODO 주석을 넣는 경우가 있는데, 주석은 그래봤자 주석이다.

- 코드의 내용을 표현하는 주석은 좋지 않다.

- api docs도 오류 가능성이 충분히 있다.

- 의무적으로 다는 api docs는 불필요하다.

- 이력관리 주석은 버전관리 시스템에서 하자

- 정보를 제공하지 않는 주석은 무의미하다.

- 주석을 코드로 표현할 수 있으면 주석없이 코드로 표현하는게 좋다.

- 배너주석의 경우에는 때로 유용하다.(코드 구분하는 주석)

- 닫는 괄호에서는 주석을 쓰지 말자

- 저자 표시 주석도 버전관리 시스템을 이용하자.

- html 주석은 정말 좋지 않다.

- 전역적인 주석은 불필요하다.

- 너무 많은 정보를 주는 주석 또한 좋지 않다.

- 주석이 충분하게 정보전달을 못하면, 혼선을 준다.

- 함수 헤더 주석은 의미가 없다.


주석은 아무리 잘 쳐줘도 필요악이다. 코드로 의도 표현을 못하는 경우 주석을 사용하게 되는데, 정말 어쩔 수 없을 때, 정보를 정확하게 전달하는 의미로 사용해야한다.