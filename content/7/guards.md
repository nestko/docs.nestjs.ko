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

상단에서, 