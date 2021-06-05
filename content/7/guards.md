### Guards

Guard는 `@Injectable()` 데코레이터로 주석을단 클래스 입니다. Guards는 `CanActivate` 인터페이스를 구현해야 합니다.

<figure><img src="/assets/Guards_1.png" /></figure>

Guards는 **single responsibility**를 가지고 있습니다. 실행환경에서 발생하는 특정 환경(허가, 역할, 접근제어목록 등등)에 따라 route handler에서 전달받은 요청을 처리할지 말지를 결정합니다. 종종 **authorization**으로 언급되어 집니다. 인증은 일반적으로 기존 Expreess application들에서 미들웨어에 의해 처리되었습니다. 토큰 검증이나 `request` 객체에 속성을 부여하는것은 특정한 route context와 강하게 연결되어 있지 않으므로 미들웨어를 인증에 이용하는것은 좋은 선택입니다.

그러나 미들웨어는 본질적으로 멍청합니다. 미들웨어는 `next()` 함수 호출후에 어떤 handler가 수행될지를 알수 없습니다. 반면 **Guards**는 `ExecutionContext` 인스턴스에 접근할 수 있습니다. 그러므로 다음에 무엇이 수행될지 정확히 알고 있습니다. 
Guards는 exception filters, pipes, interceptors 들과 같이 요청/응답 주기에서 정확한 포인트에 로직을 선언적으로 수행할수 있도록 설계되었습니다. 이것은 코드는 DRY 하고 declarative하게 유지할수 있습니다.

> info **Hint** Guards는 middleware **다음에**, interceptor 혹은 pipe **이전에** 실행됩니다.

#### Authorization guard

언급했듯이, 특정 routes들은 호출자(대게 특정 권한을 가진 사용자)에게 충분한 권한이 있을때만 가능해야 하기 때문에 autorization은 Guard의 훌륭한 사용예시 입니다. 이제 빌드할 `AuthGuard` 인증된 사용자를 가정합니다(따라서 토큰이 요청 헤더에 포함됩니다).
토큰을 추출하고 검증합니다, 그리고 추출된 정보를 가지고 요청이 진행 가능할지 아닐지를 결정합니다.

```typescript
@@filename(auth.guard)
import { Injectable, CanActivate, ExecutionContext} from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ) : boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class AuthGuard {
  async canActivate(context) {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

> info **Hint** 만약 어플리케이션에서 인증이 실제로 어떻게 시행되는지 보고 싶다면 [this chapter](/security/authentication)를 확인하세요. 마찬가지로 좀더 복잡한 예시를 확인하고 싶다면 [this page](/security/authorization)를 확인하세요.

```validateReuqest()```의 내부 로직은 필요에 따라 단순해지거나 복잡해 질 수 있습니다. 이 예시의 중점은 Guards가 요청/응답 cycle에 어떻게 동작하는지 보여주는 것입니다.

모든 guard는 무조건 ```canActivate()```를 구현해야 합니다. 이 함수는 boolean을 응답 할 수 있고, 현재 요청이 허가될지 안될지를 나타낼수 있습니다. 응답을 동기적으로 혹은 비동기적으로 전달할수 있습니다.(```Promise``` 혹은 ```Obervable```를 통해서). Nest는 응답 값을 다음 행동을 제어하기 위해 사용합니다.

- `true`를 반환하면, 다음 요청은 진행될 것입니다.
- `false`를 반환하면, Nest는 요청을 부정합니다

#### Execution Context

```canActivate()```는 ```ExecutionContext``` instance를 단일 인자로 사용합니다. ```ExecutionContext``` 는 ```ArgumentsHost```를 상속합니다. 우리는 ```ArgumentsHost```를 이전에 exception filters 챕터에서 본적이 있습니다.
아래의 예시에서, ```Request```객체로 부터 참조를 가져오기 위해 우리가 이전에 사용했었던 ```ArgumentsHost``` 에서 정의된 같은 helper methods들을 사용할것입니다. 좀 더 많은 정보를 접하려면 **Arguments Host** 섹션의 [exception filters](https://docs.nestjs.com/exception-filters#arguments-host)를 참고하면 됩니다.

```ArgumentsHost```를 확장하여, ```ExecutionContext```에 현재 실행중인 프로세스에 대하여 추가적인 정보를 제공하는 새로운 helper methods들을 추가하였습니다. 이러한 세부사항들은 execution contexts, methods, controllers 에서 폭넓게 동작할수 있는 좀 더 일반적인 guards들을 만들수 있게 도와줍니다.  `ExecutionContext`에 대하여 [here](/fundamentals/execution-context)에서 좀 더 배우세요.


#### Role-based authentication

사용자의 특정한 역할에만 허가를 해주는 좀 더 기능적인 guards를 만들어 봅니다. guard의 기본 템플릿에서 시작하고, 다음 섹션에서 build 할 것 입니다. 현재는, 모든 요청을 허가 합니다
```typescript
@@filename(roles.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs;

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ) : boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class RolesGuard {
  canActivate(context) {
    return true;
  }
}
```

#### Binding guards

pipes와 exception filters와 같이 guards는 **controller-범위**, method-범위 혹은 global-범위에서 가능합니다.
아래에, ```@UseGuards()``` 데코레이터를 사용하는 controller-범위의 guard를 준비해두었습니다. 이 데코레이터는 단일 인자를 가지거나 콤마로 분리된 리스트를 인자로 가집니다. 이렇게 하면 하나의 선언으로 적절한 가드 집합을 쉽게 적용할 수 있습니다.

```typescript
@@filename()
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```
> info **Hint** ```@UseGuards()``` 데코레이터는 ```@nestjs/common``` 패키지에서 import 합니다

위에서 우리는 인스턴스 대신에 ```@UseGuards()``` 타입을 전달하여 프레임워크에 인스턴스화와 의존성 주입을 가능하게 하였습니다. pipe와 exception filter들과 같이 내부 인스턴스를 전달할 수 있습니다.

```typescript
@@filename()
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

위의 구조는 컨트롤러에 선언된 모든 handler에 guard를 부착시킵니다. 만약 단일 메소드에 guard를 적용하고자 하면, ```@UseGuards()``` 데코레이터를 메소드 레벨에서 사용하면 됩니다.

글로벌 guard로 설정하기 위해서, ```@useGlobalGuards()```메소드를 Nest 어플리케이션 인스턴스에서 사용하십시오
```typescript
@@filename()
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

> warning **Notice** 하이브리드 앱에서 ```useGlobalGuards()``` 메소드는 기본적으로 마이크로 서비스와 게이트웨이를 위한 guard를 세팅하지 않습니다. 일반적인 경우의(non-hybrid) 마이크로서비스 앱에선, ```useGlobalGuards()``` 는 guards를 전역적으로 설정합니다.

전역 guard는 전체 어플리케이션에서 모든 컨트롤러와 모든 route 핸들러를 위해 사용되어 집니다. 의존성 주입의 측면에서 모듈의 외부에서 선언된 전역 guard는(```useGlobalGuards()```와 함께있는 위의 예시) 의존성 주입을 할수가 없습니다. 이 이슈를 해결하기 위해 아래의 구조를 따라서 
모든 모듈에 직접 guard를 세팅할수 있습니다

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  provider: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

> info **Hint** guard의 의존성 주입을 하기위해 이러한 접근을 사용할때 이 guard는 이것의 구조가 사용되는 모듈에 상관없이 전역으로 설정된다는것을 인지하고 있어야 합니다. 어디에서 실행되어야 할까요? guard (위의 예시의```RolesGuard```)가 정의된 모듈을 선택하세요. 또한 ```useClass```는 커스텀 provider 등록을 다루기 위한 유일한 방법이 아닙니다. [여기](/fundamentals/custom-providers) 에서 확인 해보세요.

#### Settings roles per handler

```RolesGuard```는 이제 동작하지만 아직 스마트 하지 않습니다. 우리는 아직 guard 특성중에 가장 중요한 [execution context](/fundamentals/execution-context)의 이점을 취하지 않았습니다. 이것은 아직 역할들 혹은 어떤 역할들이 각각의 핸들러에서 허가되는지 알지 못합니다. ```CatsController```의 예시를 보면 다른 경로에 다른 허가 체계를 가지를 있습니다. 일부는 관리자에게만 가능할수 있고 다른것은 모두에게 개방되어 질수 있습니다. 어떻게 유연하고 재사용할수 있도록 경로에 역할을 match할수 있을까요?

**커스텀 메타데이터**가 역할을 부여받는 곳입니다 ([여기](https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata)에서 좀더 배울수 있습니다). Nest는 ```@SetMetadata()``` 데코레이터를 통해 경로 핸들러 에게 커스텀 **메타데이터**를 부착할수 있게 제공합니다. 이 메타데이터는 결정을 내리는데 필요한 guard인 역할 데이터를 제공합니다. ```@SetMetadata()```를 살펴봅시다

```typescript
@@filename(cats.controller)
@Post()
@SetMetadata('roles', ['admin'])
async create(@Body() createCatDat: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@Switch
@Post()
@SetMetadata('roles', ['admin'])
@Bind(Body())
async create(createCatDto) {
  this.catsService.create(createCatDto);
}
```
> info **Hint** ```@SetMetadata()```는 ```nestjs/common``` 패키지로 부터 import 됩니다

위의 구조에서 ```create()```메소드에 ```roles``` 메타데이터(```roles```는 키, ```['admin']```은 특정 값입니다.)를 부착하였습니다 이것이 동작하는 중에 당신의 경로에 ```@SetMetadata)```를 직접적으로 사용하는것은 좋은 예시가 아닙니다. 대신 아래의 예시와 같이 알맞은 각각의 데코레이터를 생성하세요. 

```typescript
@@filename(roles.decorator)
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
@@switch
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles) => SetMetadata('roles', roles)
```

이 접근은 타입화 되어있고 훨씬 깔끔하고 가독성이 좋습니다. 이제 우리는 커스텀 ```@Roles()``` 데코레이터가 생겼고 이것을 ```create()```에 사용할수 있습니다.

```typescript
@@filename(cats.controller)
@Post()
@Roles('admin')
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
@Roles('admin')
@Bind(Body())
async create(createCatDto) {
  this.catsService.create(createCatDto);
}
```

#### Putting it all together

이제 돌아가서 ```RolesGuard```와 함께 이것들을 묶어 봅시다. 현재 이것은 모든 경우에 ```true```를 반환하여 모든 요청의 진행을 허락합니다.
우리는 현재 경로를 진행하기위해 필요한 실제의 역할과 **현재 유저에 할당된 역할**의 비교에 근거한 값을 반환하길 원합니다. 경로들의 역할에 접근하기 위해서 ```@nestjs/core```패키지에서 제공하고 프레임워크의 밖에서 제공하는  ```Reflector``` 헬퍼 클래스를 사용할것입니다. 

```typescript
@@filename(roles.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if(!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
@@switch
import { Injectable, Dependencies } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
@Dependencies(Reflector)
export class RolesGuard {
  constructor(reflector) {
    this.reflector = reflector;
  }

  canActivate(context) {
    const roles = this.reflector.get('roles', context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
```

> info **Hint** node.js에서 ```request``` 객체에 권한이 있는 사용자를 부착하는것은 흔한 경우입니다. 그러므로 위의 우리 예시에서 우리는 ```request.user```가 user 인스턴스와 허가된 역할을 가지고 있다고 가정하였습니다. 너의 앱에서 너는 아마 커스텀 **authentication guard**(혹은 미들웨어)를 만들것입니다. 좀 더 많은 정보를 위해[this chapter](/security/authentication)를 확인하세요

> warning **Warning** ```matchRoles()``` 기능의 내부 로직은 필요에 의해 복잡해지거나 단순해질수 있습니다. 이 예시의 중점은 어떻게 guard가 요청/응답 사이클에 동작하는지를 보여주는 것입니다.

```Reflector```를 더 자세히 다루고 활용하기 위해 <a href="https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata">Reflection and metadata</a> 섹션의 **Execution context** 챔터를 참고하세요

요청에 불충분한 권한을 가지고 있는 사용자가 요청을 하면, Nest는 아래의 응답을 반환합니다

```typescript
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

뒤에서 guard가 ```false```를 반환할때 프레임워크는 ```ForbiddenException```을 반환합니다. 만약 다른 에러 응답을 반환하길 원한다면, 고유한 예외를 처리해야 합니다 예를들면: 
```typescript
throw new UnauthorizedException();
```
guard에 의해 반환된 모든 예외는 [exceptions layer](/exception-filters)(전역 exceptions filter와 모든 exceptions filter가 적용되는 현재 context)에서 다룹니다. 

> info **Hint** 만약 인증을 어떻게 수행하는지 실제 예시를 보고싶다면 [this chapter](/security/authorization).를 확인하세요.