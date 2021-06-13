### Modules

모듈은 `@Module()` 데코레이터로 주석이 달린 클래스 입니다. `@Module()` 데코레이터는 **Nest**가 어플리케이션 구조를 구성하는데 사용하는 메타데이터를 제공합니다.

<figure><img src="/assets/Modules_1.png" /></figure>

어플리케이션은 적어도 **루트 모듈** 한개의 모듈을 가지고 있습니다. 루트 모듈은 Nest가 모듈과 provider들의 관계와 의존성 정의를 구성하기 위해 사용하는 내부 데이터 구조인 **application graph**를 구성하기 위해 Nest가 사용하는 시작점 입니다. 규모가 작은 어플리케이션의 경우 이론적으로 루트 모듈만을 가질수 있지만 이는 일반적인 경우가 아닙니다. 우리는 구성요소들을 효율적으로 조직하기 위한 방법으로 모듈을 **강력히** 추천하고 싶습니다. 그러므로 대부분의 어플리케이션들에서 결과 구조는 인접한 **기능**들로 캡슐화된 여러개의 모듈을 이용할것입니다.

`@Module()` 데코레이터는 모듈을 나타내는 속성을 가진 단일객체를 취합니다.

<table>
  <tr>
    <td><code>providers</code></td>
    <td>provider들은 Nest injector에 의해 인스턴스화 되고 적어도 모듈내부에서 공유 됩니다.</td>
  </tr>
  <tr>
    <td><code>controllers</code></td>
    <td>인스턴스화 되어야하는 모듈에 정의된 컨트롤러들</td>
  </tr>
  <tr>
    <td><code>imports</code></td>
    <td>이 모듈애 필요한 provider들을 내보내는 모듈들의 목록</td>
  </tr>
  <tr>
    <td><code>exports</code></td>
    <td>이 모듈에서 제공되고 이 모듈을 import하는 다른 모듈에서 이 모듈을 사용할수 있게하는 <code>providers</code>들의 부분 집합</td>
  </tr>
</table>

모듈은 provider들을 기본적으로 **캡슐화**합니다. 이것은 현재 모듈의 부분이 아니거나 가져온 모듈에서 내보내지 않은 provider들의 주입이 불가능하다는 것을 의미합니다. 그러므로 모듈로 부터 내보내진 provider들을 모듈의 공용 인터페이스나 API로 간주해야 합니다.

#### Feature modules

`CatsController`과 `CatsService` 같은 어플리케이션 영역에 속합니다. 밀접하게 연관되어 있으므로 feature 모듈로 이동하는 것으로 이해할수 있습니다. feature 모듈은연관된 특정기능을 위해 단순히 구조화하고 코드를 조직화되고 명확한 경계를 가지도록 유지합니다.
이것은 어플리케이션 혹은 팀의 규모가 커짐에 따라 [SOLID](https://en.wikipedia.org/wiki/SOLID) 원칙으로 개발하고 복잡도를 다룰수 있게 도와줍니다.

이것을 증명하기 위해 우리는 `CatsModule`을 만들것입니다.

```typescript
@@filename(cats/cats.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

> info **Hint** CLI를 이용하여 모듈을 만들기 위해 `$ nest g module cats` 명령을 실행해주세요.

위에서 우리는 `cats.module.ts` 파일에 `CatsModule`을 정의 하였고 이 모듈관련한 모든 것들을 `cats` 디렉터리로 이동시켰습니다.
마지막으로 해야할것은 root 모듈에 이 모듈을 import 하는 것입니다(`AppModule`은 `app.module.ts`파일에 정의되어 있습니다).

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

디렉터리 구조는 다음과 같습니다.

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
      <div class="item">cats.module.ts</div>
      <div class="item">cats.service.ts</div>
    </div>
    <div class="item">app.module.ts</div>
    <div class="item">main.ts</div>
  </div>
</div>

#### Shared modules

Nest에서 모듈은 기본적으로 **싱글톤**이며 복잡한 모듈간에 간편하게 어떠한 provider라도 같은 인스턴스를 공유할수 있습니다.

<figure><img src="/assets/Shared_Module_1.png" /></figure>

모든 모듈은 자동으로 **공유 모듈**입니다. 한번 생성이 되어지면 어떠한 모듈에 의해서라도 재사용 될 수 있습니다. 다른 여러 모듈간에 `CatsService` 인스턴스를 공유하길 원한다고 가정해 봅시다. 이것을 행하기 위해서 아래의 예시와 같이 우리는 첫번째로 `CatsService` provider를 모듈의 `exports`배열에 추가함으로서 **내보내야 합니다**

```typescript
@@filename(cats.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

이제 `CatsModule`를 import한 어떠한 모듈이라도 `CatsService`를 사용할수 있고 이것을 import한 다른 모듈간에 같은 인스턴스를 공유할수 있습니다.

<app-banner-enterprise></app-banner-enterprise>

#### Module re-exporting

아래에서 볼수 있듯이 모듈들은 내부 provider들을 export할수 있습니다. 게다가 그들은 그들이 import한것을 다시 export할수 있습니다.
아래의 예시에서 `CoreModule`에서 `CommonModule`은 import **되고** export 함으로써 이 모듈을 import한 다른모듈이 사용할수 있습니다.

```typescript
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

#### Dependency injection

모듈 클래스는 provider를(예: 구성 목적으로) **주입**할수 있습니다.

```typescript
@@filename(cats.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
@@switch
import { Module, Dependencies } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
@Dependencies(CatsService)
export class CatsModule {
  constructor(catsService) {
    this.catsService = catsService;
  }
}
```

그러나 모듈 클래스들 간에는 [circular dependency](/fundamentals/circular-dependency) 때문에 provider로서 주입될수는 없습니다.

#### Global modules

만약 같은 모듈들의 집합을 모든곳에서 import해야 하는경우 이것은 지루할수 있습니다. Nest와 달리 [Angular](https://angular.io) `providers`는 전역 범위로 등록이 되어있습니다. 한번 정의되어지면 모든곳에서 사용 가능 합니다. 그러나 Nest는 provider들을 모듈 범위에서 캡슐화 합니다. 캡슐화 모듈을 import하지 않는다면 모듈의 provider들을 사용할수 없습니다.

박스의 바깥에서 (예: 헬퍼, 데이터베이스 연결 등) provider들의 집합을 제공하고 싶다면 모듈을 `@Global()` 데코레이터를 이용하여 **전역**으로 생성할수 있습니다.

```typescript
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

`@Global()` 데코레이터는 모듈을 전역범위로 만듭니다. 전역 모듈은 일반적으로 root나 주요 모듈에 **한 번만** 등록되어야 합니다.
위의 예시에서 `CatsService` provider는 어디에나 어디에나 있을수 있으며 서비스를 주입하고자 하는 모듈에서 `CatsModule`을 그들의 import 배열에 추가할 필요는 없습니다.

> info **Hint** 모든것을 글로벌하게 만드는 것은 좋은 선택이 아닙니다. 전역 모듈은 필요한 boilerplate의 양을 줄일수 있습니다. `imports` 배열은 모듈의 API를 사용자가 사용할수 있도록 하는데 일반적으로 선호되어 집니다.

#### Dynamic modules

Nest의 모듈 시스템은 **동적 모듈**로 불리는 강력한 기능을 포함하고 있습니다. 이 기능은 동적으로 provider들을 구성하고 등록하는 모듈을 쉽게 사용자 지정이 가능하도록 만들수 있습니다. 동적모듈은 [여기](/fundamentals/dynamic-modules)에서 다루고 있습니다. 이 챕터에서 모듈의 구성을 완성하는 간략한 개요를 제공합니다.

아래는 `DatabaseModule`을 위한 동적 모듈 정의의 예시입니다.

```typescript
@@filename()
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
@@switch
import { Module } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?) {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

> info **Hint** `forRoot()` 메소드는 동기적 혹은 비동기적으로 동적 모듈을 제공합니다.(즉, `Promise`를 통해).

이 모듈은 `Connection` provider를 기본으로 정의하지만(`@Module()` 데코레이터 메타데이터 안에서) 추가적으로 `forRoot()` 메소드 내부에 제공되는 `entities`과  `options` 객체에 따라 repository와 같은 provider들의 집합을 제공합니다. 
동적모듈을 통해 제공되는 속성들은 기본 모듈의 `@Module()` 데코레이터 내부에 정의되어 있는 메타데이터들을 **확장**(덮어씌움 보다는) 한다는것을 알고있어야 합니다. 이것이 정적으로 선언된 `Connection` provider와 동적으로 생성된 repository provider들을 모듈로부터 export하는 방법입니다. 

만약 동적모듈을 전역범위로 등록하고 싶다면 `global` 속성을 `true`로 설정하세요

```typescript
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

> warning **Warning** 위에서 언급되었듯이, 모든것을 전역으로 생성하는 것은 **좋은 선택이 아닙니다**.

`DatabaseModule`은 아래의 방법을 따라 구성되고 import될수 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

만약에 동적모듈을 다시 export하고 싶다면 `forRoot()` 메소드를 export 배열에서 호출하는것을 생략 할수 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class AppModule {}
```

[동적모듈](/fundamentals/dynamic-modules) 챕터는 좀 더 자세하게 이 주제를 다루고 [동작 예시](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules).를 포함하고 있습니다.