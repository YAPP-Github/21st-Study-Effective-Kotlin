# 아이템 7: 공통 모듈을 추출해서 여러 플랫폼에서 재사용하라

### 풀스택 개발

- 코틀린은 자바에서 사용하는 프레임워크에서 대부분 사용 가능, 자바스크립트로 컴파일 될 수 있음
- 웹 개발시 백엔드와 프론트엔드를 둘 다 코틀린으로 만들 수 있음. 같은 언어를 사용하는 이점 뿐 아니라 코드를 공유해서 공통 코드, API 엔드포인트 정의, 기타 추상화등을 재사용 할 수 있음

### 모바일 개발

- 모바일 개발의 경우, 아이폰과 안드로이드의 동작이나 로직이 유사하지만 다른 언어와 도구를 사용해서 개발하고 있었음.
- 코틀린 멀티 플랫폼 기능을 이용하면 로직을 한번만 구현하고, 두 플랫폼에서 이를 재사용 할 수 있음.
- 공통 모듈을 만들어 비즈니스 로직을 구현하면 된다.
    - 비즈니스 로직은 플랫폼, 프레임워크에 종속되지 않고 독립적이어야함 (클린 아키텍처)
    - 코틀린/네이티브 컴파일러가 LLVM 에서 지원되므로 IOS에서도 모듈을 동시에 이용할 수 있다.