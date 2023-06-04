---
layout: post
title: 상위호환성과 하위호환성에 대해서
tags:
    - compatibility
    - oop
authors: [len]
---



 외부에 노출하는 REST API 를 작성하다보면 자연스럽게 고민의 영역으로 불리는 용어가 있다. 바로 **하위호환성** 입니다.

흔히 **하위호환성**에 대해서 V1 이였던 서버의 API 가 필드의 추가 또는 수정에 의해 V2 가 되었을 경우, V1 의 API 를 사용하던 클라이언트는 V2를 사용하기 전 V1에 대해서도 사용 가능함을 의미합니다.

여기서 말한 하위 호환성에 대해서는 어느정도 이해가 될 수 있다. 그렇다면 상위호환성은 어떤 것일까? 상위호환성이라는 단어가 나오면서 이해하기 시작하려니 하위 호환성에 대한 의미가 애매하게 받아들여 졌다.

아래는 [backward-vs-forward-compatibility](https://stevenheidel.medium.com/backward-vs-forward-compatibility-9c03c3db15c9) 발췌한 그림이다.

아래 장표를 설명하기 전 장표에 나오는 등장 인물을 소개해보면 다음과 같다.

**Schema(v1, v2..)**

 스키마는 Json 의 형태등의 데이터 유형의 정의 및 데이터를 이해하는 데 필요한 컨텍스트을 의미합니다.

**Reader**

 Reader 는 데이터를 구문 분석하는 서비스입니다. 클라이언트-서버 애플리케이션의 경우에 서버가 데이터를 보낼 때마다 클라이언트는 Reader가 됩니다. 그러나 클라이언트가 서버에 보내는 데이터(예: 함수에 대한 입력 인수)에 대해 이야기할 때는 역할이 반대로 됩니다. 즉, 클라이언트가 Writer가 됩니다.

**Writer**

Writer 는 데이터를 생성하는 서비스입니다. 클라이언트-서버 애플리케이션의 경우에 서버가 됩니다. 그러나 위에서 설명한 것과 같이 반대가 될 수도 있습니다.


![507ED655-6B4A-4066-8D97-9EAD18D45996](https://raw.githubusercontent.com/LenKIM/images/master/2023-06-03/507ED655-6B4A-4066-8D97-9EAD18D45996.jpg)

위 장표에 대한 설명으로 하위호환성과 상위호환성을 설명합니다.

- 하위 호환성(이전 버전과의 호환성)은 최신 스키마를 사용하는 Reader가 이전 스키마를 사용하는 Writer의 데이터를 올바르게 구문 분석할 수 있다는 의미입니다.
- 상위 호환성(상위 버전과의 호환성)은 이전 스키마를 사용하는 Reader가 최신 스키마를 사용하는 Writer의 데이터를 올바르게 구문 분석할 수 있음을 의미합니다.



Protobuf 는 상위 호환성과 하위 호환성을 지원해주는 도구라고 알려져 있습니다. Protobuf 는 어떻게 상위 호환성과 하위 호환성을 지킬 수 있을까요?











[참고]

- https://stevenheidel.medium.com/backward-vs-forward-compatibility-9c03c3db15c9
- https://protobuf.dev/overview/#updating-defs
