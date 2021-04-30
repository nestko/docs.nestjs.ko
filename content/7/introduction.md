원문: [Introduction](https://github.com/nestjs/docs.nestjs.com/blob/master/content/introduction.md)

역자: [dogemad](https://github.com/dogemad)

### 소개

Nest (NestJS)는 [Node.js](https://nodejs.org/) 환경에서 효율적이고 확장성이 뛰어난 서버 사이드 어플리케이션을 구축하기 위한 프레임 워크입니다. 진보한 자바스크립트(역: 아마도 ECMA Script를 의미하는)를 사용하고 [TypeScript](http://typescriptlang.org/) (하지만 순수 JavaScript로도 코딩할 수 있음)로 구축되었으며 전적으로 TypeScript를 지원합니다. 또한 OOP (Object Oriented Programming), FP (Functional Programming), FRP (Functional Reactive Programming) 의 요소도 결합되어 있습니다.

내부적으로 Nest는 [Express](https://expressjs.com/) (기본값)와 같은 강력한 HTTP 서버 프레임 워크를 사용하며 선택적으로 [Fastify](https://github.com/fastify/fastify/)를 사용하도록 구성 할 수도 있습니다!

Nest는 일반적인 Node.js 프레임워크(Express/Fastify) 이상의 추상화 수준을 제공하지만, 개발자에게 API를 직접 제공합니다. 이를 통해 개발자는 기본 플랫폼에 사용 가능한 third-party 모듈을 자유롭게 사용할 수 있습니다.

#### 철학

최근 몇 년 동안 Node.js 덕분에 JavaScript는 프론트엔드, 백엔드 어플리케이션 양쪽에 있어 웹의 "lingua franca"(역: 만국 공통어의 의미)가 되었습니다. 이로 인해 [Angular](https://angular.io/), [React](https://github.com/facebook/react), [Vue](https://github.com/vuejs/vue) 와 같은 멋진 프로젝트들이 생겨나 개발 생산성이 높고, 빠르고, 테스트가 가능하며 확장 가능한 프론트엔드 어플리케이션을 만들 수 있었습니다. 그러나 Node (및 서버 사이드 JavaScript)를 위한 훌륭한 라이브러리나 헬퍼와 같은 도구들이 많이 존재 하지만 이 중 어느 것도 **아키텍쳐** 상의 주요한 문제들을 효과적으로 해결하지 못합니다.

Nest는 개발자와 팀이 고도로 테스트 가능하고, 확장 가능하며, 느슨하게 결합되고, 쉽게 유지 관리할 수 있는 어플리케이션을 즉석에서 만들 수 있는 아키텍쳐를 제공합니다. 해당 아키텍쳐는 Angular에서 큰 영감을 받았습니다.

#### 설치

시작하려면 [Nest CLI]()를 사용하여 프로젝트를 스캐폴딩 하거나 입문용 프로젝트를 복제할 수 있습니다(둘 다 동일한 결과를 생성함).

Nest CLI로 프로젝트를 스캐폴드 하려면 다음 명령어를 실행하세요. 이렇게 하면 새 프로젝트 디렉토리가 생성되고 초기 코어 Nest 파일 및 지원 모듈로 디렉토리가 채워져 프로젝트의 기본 구조가 생성 됩니다. 초심자는 **Nest CLI**를 사용해 새 프로젝트를 만들 것을 권장합니다. 이런 접근 방식은 [First Steps]()에서 마저 진행하겠습니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

#### 대안

**Git**을 사용해 입문용 TypeScript 프로젝트를 복사할 수도 있습니다:

```bash
$ git clone https://github.com/nestjs/typescript-starter.git project
$ cd project
$ npm install
$ npm run start
```

브라우저를 열고 [`http://localhost:3000/`](http://localhost:3000/) 로 이동하세요.

입문용 JavaScript 버전을 설치하려면 위의 명령어 시퀀스에서 `javascript-starter.git` 를 사용하세요.

**npm** (또는 **yarn**)으로 코어나 서포트 파일을 설치한 뒤 처음부터 수동으로 새 프로젝트를 만들 수도 있습니다. 물론 이 경우에는 스스로 프로젝트의 보일러 플레이트를 만들어야 합니다.

```bash
$ npm i --save @nestjs/core @nestjs/common rxjs reflect-metadata
```

