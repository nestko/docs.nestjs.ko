### Providers

Provider는 Nest의 기본 개념입니다. 대부분의 기본 Nest 클래스들이 provider로 간주 됩니다 - services, repositories, factories, helpers, 등등. provider의 주요한 개념은 의존성 **주입**을 할수 있다는 것입니다. 이것은 객체들이 그들 간에 다양한 관계를 만들어 낼수 있다는 것이고 객체 인스턴스들의 "wiring up" 기능은 Nest 실행 환경에 위임 될수 있습니다.

<figure><img src="/assets/Components_1.png" /></figure>

이전 챕터에서 우리는 단순한 `CatsController`를 만들었습니다. 컨트롤러는 HTTP 요청을 다룰수 있고 **providers**에게 더 복잡한 임무를 부여할수 있습니다. [module](/modules)안에서 Providers는 순수 자바스크립트 클래스로 `providers` 로 선언되어 집니다.

> info **Hint** Nest는 객체지향 방법에서 의존성을 조직하고 디자인하는게 가능합니다. [SOLID](https://en.wikipedia.org/wiki/SOLID) 원칙에 따르는 것을 추천합니다.

#### Services

단순한 `CatsService`를 만드는것으로 시작합시다. 이 서비스는 데이터의 저장과 검색을 담당할 것이며 `CatsController`에 의해 사용되는것으로 디자인 되었습니다. 그래서 이것은 provider로 정의될수 있는 좋은 예시 입니다.

```typescript
@@filename(cats.service)
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  constructor() {
    this.cats = [];
  }

  create(cat) {
    this.cats.push(cat);
  }

  findAll() {
    return this.cats;
  }
}
```

> info **Hint** `$ nest g service cats` 명령어를 사용하면 CLI에서 서비스를 생성할수 있습니다.

`CatsService` 하나의 속성과 두개의 메소드를 가진 기본 클래스 입니다. 유일한 새로운 특성은 `@Injectable()` 데코레이터를 사용한다는 것입니다.`@Injectable()` 데코레이터는 메타데이터를 부착하여, `CatsService`가 Nest IoC container에 의해 관리되는 클래스라고 알려줍니다. 이 예제는 또한 아래의 예시에서 볼수 있는 `Cat` 인터페이스를 사용합니다:

```typescript
@@filename(interfaces/cat.interface)
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

cats를 반환하는 서비스 클래스가 생겼으니, `CatsController` 안에서 사용해 봅시다:

```typescript
@@filename(cats.controller)
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
@@switch
import { Controller, Get, Post, Body, Bind, Dependencies } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
@Dependencies(CatsService)
export class CatsController {
  constructor(catsService) {
    this.catsService = catsService;
  }

  @Post()
  @Bind(Body())
  async create(createCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll() {
    return this.catsService.findAll();
  }
}
```

`CatsService`는 클래스 constructor를 통해 **주입** 됩니다. `private` 문법에 주목하세요. 이 단축명렁어는 `catsService` 멤버를 즉시 실행하고 선언하는것을 같은위치에서 할수 있도록 해줍니다.

#### Dependency injection

Nest **의존성 주입**으로 흔히 알려진 디자인 패턴을 중심으로 구축되었습니다. [Angular](https://angular.io/guide/dependency-injection) 문서에서 개념에 관하여 읽어보기를 추천합니다.

Nest에서 TypeScript의 기능 덕분에, 의존성 관리가 매우 쉬워졌습니다. 아래의 예시에서, Nest는 `CatsService` 인스턴스를 생성하고 반환함으로써 `catsService`를 생성 합니다(singleton의 일반적인 경우 다른곳에서 이미 요청되어졌던 인스턴스가 존재한다면 이것을 반환합니다). 
이 의존성은 생성되어지고 controller의 contstructor를 통해 전달됩니다(혹은 지정된 속성에 할당됩니다):

```typescript
constructor(private catsService: CatsService) {}
```

#### Scopes

Providers는 일반적으로 어플리케이션의 lifecycle과 동기화된 수명("범위")를 가집니다. 어플리케이션 부트스트랩 될때 모든 의존성은 모든 종속성이 부여되어야 합니다, 그래서 모든 provider는 인스턴스화 되어집니다. 유사하게, 어플리케이션 꺼질때 모든 provider는 종료됩니다. 그러나 
provider의 수명을 **요청 범위**로 만들수 있는 방법이 있습니다. [여기](/fundamentals/injection-scopes)에서 이러한 기술에 대해 좀 더 읽어 보세요.

<app-banner-courses></app-banner-courses>

#### Custom providers

Nest는 providers간에 관계를 생성할수 있는 내장된 제어반전 컨테이너를 가지고 있습니다. 이러한 특성은 위의 예시에 나타난 의존성 주입의 밑바탕에 깔려있습니다. 허나 이것은 위의 예시에서 제공된것보다 훨씬 유용합니다. provider는 여러뜻으로 정의 됩니다: 순수 값, 클래스 비동기 혹은 동기적 factories로 정의 될수 있습니다. 더 많은 예시는 [여기](/fundamentals/dependency-injection)에서 확인하세요.

#### Optional providers

가끔, 필수가 아닌 의존성을 생성해야할 수도 있습니다. 예를 들어 클래스는 **configuration 객체**에 의존할수 있지만 아무 값도 전달되지 않은 경우 기본값이 사용되어져야 합니다. 이러한 경우 configuration provider자의 부재가 에러를 야기시키지 않으므로 의존성은 선택사항이 될수 있습니다.

provider가 선택사항이라는것을 명시하기 위해 `@Optional()` 데코레이터를 constructor 안에 사용하세요

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

위의 예시에서 우리는 커스텀 provider를 사용하고 있는데, 이것이 우리가 `HTTP_OPTIONS` 커스텀 **토큰**을 포함하는 이유입니다.
이전의 예시에서 constructor 클래스를 통해 constructor 기반의 의존성 주입을 확인하였습니다
[여기](/fundamentals/custom-providers)에서 커스텀 provider와 제공되어지는 token에 관해 더 읽어보세요.

#### Property-based injection

이전에 사용되어져 왔던 기술은 constructor 메소드를 통해 providers가 주입되어 졌으므로 constructor 기반 주입이라고 합니다.
특별한 경우 **속성 기반 주입**이 유용할 것입니다. 예를들면 최상위 클래스가 하나 혹은 복수의 provider에 의존하고 있다면 모든 하위 클래스에서 `super()`를 호출함으로서 constructor를 제공하는 것은 굉장히 지루할 수 있습니다. 이것을 피하기 위해 `@Inject()` 데코레이터를 속성 레벨에서 사용할 수 있습니다.

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

> warning **Warning** 클래스가 다른 provider를 확장하지 않는경우, **constructor 기반** 주입을 사용하는것이 좋습니다..

#### Provider registration

(`CatsService`) provider를 정의하였습니다 그리고 (`CatsController`) 서비스를 사용하고 있습니다, Nest가 주입을 수행할수 있도록 서비스를 등록하는것이 필요합니다. (`app.module.ts`)을 수정하고 서비스를 `@Module()` 데코레이터의 `providers` 배열에 추가하면 됩니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

Nest는 `CatsController` 클래스에 의존성을 부여할수 있을 것입니다.

디렉토리 구조는 다음과 같습니다.

<div class="file-tree">
<div class="item">src</div>
<div class="children">
<div class="item">cats</div>
<div class="children">
<div class="item">dto</div>
<div class="children">
<div class="item">create-cat.dto.ts</div>
</div>
<div class="item">interfaces</div>
<div class="children">
<div class="item">cat.interface.ts</div>
</div>
<div class="item">cats.controller.ts</div>
<div class="item">cats.service.ts</div>
</div>
<div class="item">app.module.ts</div>
<div class="item">main.ts</div>
</div>
</div>

#### Manual instantiation

지금까지, 우리는 Nest가 의존성 해결의 대부분의 세부사항을 자동으로 처리하는 방법에 대해 논의했습니다. 특정 상황에서 내장 의존성 주입을 사용하지 않고 수동으로 provider를 제공하거나 인스턴스화 해야할수 있습니다. 아래에서 두개의 주제에 관해 간략히 논의 해보도록 하겠습니다.

현재 존재하는 인스턴스들을 가져오거나 provider를 동적으로 인스턴스화 하기 위해 [Module reference](https://docs.nestjs.com/fundamentals/module-ref)를 사용할수 있습니다

`bootstrap()` 함수(예를 들어 컨트롤러가 없는 standalone 어플리케이션 이거나 부트스트래핑 중에 configuration 서비스를 활용하는 경우) 내부에서 provider를 가져오기 위해 [Standalone applications](https://docs.nestjs.com/standalone-applications)를 확인하세요
