원문: [FirstSteps](https://github.com/nestjs/docs.nestjs.com/blob/master/content/first_steps.md)

역자: [woobottle](https://github.com/woobottle)

### 첫단계

이 글에서는 Nest의 **core fundamentals** 에 대해 배울것 입니다. Nest application들의 필수 조각들과 익숙해지기 위해 입문수준에서 많은 부분을 소화할수 있는 기본적인 CRUD applicaiton을 구축할것 입니다.

#### 언어

[TypeScript](https://www.typescriptlang.org/)와 [Node.js](https://nodejs.org/en/)를 선호합니다. 이것이 Nest가 Typescript와 **pure Javascript**를 사용할수 잇는 이유입니다.
Nest는 최신 언어 긴능을 활용하므로 vanilla Javascript와 함께 사용하려면 [Babel](https://babeljs.io/) 컴파일러를 필요로 합니다.

제공된 예시들은 대개 TypeScript를 사용하고 있습니다. 그러나 항상 vanilla Javascript로 제공된 코드도 확인할수 있습니다(예시의 우상단에 언어 변환 버튼을 클릭해주세요)

#### 전제조건

[Node.js](https://nodejs.org/) (>= 10.13.0, except for v13) 가 운영환경에 설치되어 있어야 합니다.

#### 설정

[Nest CLI](/cli/overview)를 이용하면 기본 프로젝트 세팅은 굉장히 간단합니다. 설치되어있는 [npm](https://www.npmjs.com/)를 이용하여 OS 터미널 에서 아래의 명령어로 Nest 프로젝트를 생성할수 있습니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

`project` 디렉터리가 생성되고 node module과 몇몇 boiler plate 파일들이 설치될것입니다. 몇몇 핵심 file들로 채워진 `src` 디렉터리가 생성될것입니다.

<div class="file-tree">
  <div class="item">src</div>
  <div class="children">
    <div class="item">app.controller.spec.ts</div>
    <div class="item">app.controller.ts</div>
    <div class="item">app.module.ts</div>
    <div class="item">app.service.ts</div>
    <div class="item">main.ts</div>
  </div>
</div>

아래는 핵심 파일들의 간단한 개요입니다.

|                          |                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| `app.controller.ts`      | 단일 경로를 가진 기본 컨트롤러 입니다.                                                                                       |
| `app.controller.spec.ts` | 컨트롤러의 단위 테스트 입니다.                                                                                             |
| `app.module.ts`          | application의 기본 모듈 입니다.                                                                                         |
| `app.service.ts`         | 단일 method의 기본 service 입니다.                                                                                      |
| `main.ts`                | Nest applicaiton 인스턴스 생성을 하기 위한 `NestFactory`의 핵심 기능을 이용하는 application의 entry 파일 입니다.                   |

`main.ts`에는 **bootstap** appication을 가진 비동기 함수를 포함합니다. 

```typescript
@@filename(main)

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap(){
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
@@switch
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap(){
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

Nest applicaiton 인스턴스를 생성하기 위해 `NestFactory` 클래스를 이용합니다. `NestFactory`는 application 인스턴스를 생성하는 몇개의 static method들을 보여줍니다.
`create()` method는 `INestApplication` interface를 수행하는 application 객체를 반환합니다. 이 객체는 이후 챕터에서 확인할수 있는 몇가지 metho들을 제공합니다. `main.ts` 예제에서 
http 요청에 응답 대기하는 http listener를 간단하게 시작할 것입니다.

Nest ClI로 생성한 초기의 프로젝트 구조는 개발자로 하여금 각 모듈을 전용 directory에 포함하는 convention을 따르기를 권장합니다

<app-banner-courses></app-banner-courses>

#### 플랫폼

Nest는 플랫폼에 구애받지 않는 framework를 지향합니다. 플랫폼 독립은 개발자가 여려 유형의 applicaiton에 걸쳐 활용되는 재사용 가능한 논리적 부분들을 만들수 있게 합니다. 기술적으로 Nest는 adapter가 생성된 어떠한 Node HTTP 프레임워크들과 동작합니다.
[express](https://expressjs.com/) 와 [fastify](https://www.fastify.io)가 있습니다. 필요에 맞는 것을 선택하시면 됩니다.

|                    |                                                                                                                                                                                                                                                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `platform-express` | [Express](https://expressjs.com/)는 잘알려진 소규모의 node 웹 프레임워크 입니다. 이것은 전투적으로 테스트된, 많은 커뮤니티에서 실행된 수많은 리소스를 가진 production-ready 라이브러리 입니다. `@nestjs/platform-express` package는 default로 사용됩니다. 많은 사용자들이 express를 잘 이용하고 있기 때문에 실행시키기 위한 특별한 action은 필요 없습니다. |
| `platform-fastify` | [Fastify](https://www.fastify.io/)는 최고의 효율성과 속도를 제공하는데 중점을 둔 높은 성능과 적은 overhead를 가진 프레임워크 입니다. [here](/techniques/performance) 를 사용하기 위해 확인해주세요.                                                                                                                                  |

어느 플랫폼을 사용하든 자체 interface를 표시합니다. `NestExpressApplication` 과 `NestFastifyApplication` 으로 각각 확인할수 있습니다. 

아래의 예시에서와 같이 `NestFactory.create()` method로 type을 제공할때 `app` 객체는 특정 플랫폼에 대해서만 사용할수 있는 method가 있습니다. 
그러나 기본 platform API에 접근하려는 경우가 아니면 type을 지정할 필요가 없습니다.

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
```

#### 실행
설치가 전부 끝난후 아래의 명령어를 명령 프롬프트에서 실행시키면 http요청을 수신하는 어플리케이션이 실행됩니다. 

```bash
$ npm run start
```

위 명령어는 `src/main.ts` 파일에 정의 되어있는 포트를 수신하는 http 서버를 실행 시킵니다. 
application이 실행되면 브라우저를 열고 `http://localhost:3000` 으로 이동하세요. 
`Hello World!` 명령어를 확인할수 있습니다.
