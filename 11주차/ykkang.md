# 코드를 작성하고 실향하기

## item 53 : 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

타입 스크립트 초창기 시절에 도입된 몇개의 기능들을 사용하면 타입공간과 값공간의 경계를 혼란스럽게 만든다.

1. enum

```ts
enum Flavor {
  VANILLA = 0,
  CHOCOLATE = 1,
  STRAWBERRY = 2,
}

let flavor = Flavor.CHOCOLATE; // type Flavor

let flavor2 = Flavor[0]; // 값이 "VANILLA"; type string
```

여기서 아래처럼 없는 값이 할당되면 에러가 발생됨

```ts
let flavor3 = Flavor[4]; // 매우 위험함.
```

그리고 상수 열거형은 보통의 열거형과 달리 런타임에 완전히 제거됨. 그래서 const를 제거하면 컴파일시 코드가 생성됨.

```ts
const enum Flavor {
  VANILLA = 0,
  CHOCOLATE = 1,
  STRAWBERRY = 2,
}
```

문자열 열거형은 런타임의 타입 안정성과 투명성을 제공. 대신 타입스크립트의 다른 타입과 달리 구조적 타이핑이 아닌 명목적 타이핑을 사용함.

```ts
const enum Flavor {
  VANILLA = "vanilla",
  CHOCOLATE = "chocolate",
  STRAWBERRY = "strawberry",
}

let flavor = Flavor.CHOCOLATE; // type Flavor
flavor = "strawberry"; // error. 구조적 타이핑이 XXX
```

그래서 아래처럼 쓰는게 좋음

```ts
type FlavorFix = "vanilla" | "chocolate" | "strawberry";
```

2. 매개변수 속성
   일반적으로 클래스를 초기화할 때 속성을 할당하기 위해 생성자의 매개변수를 활용

```ts
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}
```

위와 같은 예제를 좀더 간단히 구현할 수 있도록 타입스크립트에서는 '매개변수 속성' 기능을 지원

```ts
// 생성자 파라미터의 'public name'은 '매개변수 속성'이라고 합니다.

// '매개변수 속성' 에는 몇가지 문제가 O.

// 매개변수 속성은 컴파일시 코드가 늘어나는 문법

// 매개변수 속성이 런타임에는 실제로 사용되지만, 타입스크립트 관점에서는 사용되지 않는 것처럼 보임

// 매개변수 속성과 일반 속성을 섞어서 사용하면 클래스의 설계가 혼란

class PersonFix {
  constructor(public name: string) {}
}
```

아래 코드를 보면 실제로 클래스에는 3개의 속성이 있지만 first, last 속성만 명시적으로 나열되고 있어 일관성이 없음.

```ts
class Person {
  first: string;
  last: string;
  constructor(public name: string) {
    [this.first, this.last] = name.split(" ");
  }
}
```

3. 네임스페이스 트리플 슬래시 임포트

타입스크립트에서 모듈 내보내기 및 가져오기 기능을 사용할 때 'import'와 'export'를 쓰셈.

4. 데코레이터
   데코레이터는 클래스, 메서드, 속성에 annotation을 추가하거나 기능을 추가하는 데 사용 가능. 예를 들어, 클래스의 매서드가 호출될 때마다 로그를 남기려면 logged annotation 정의.

```ts
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }

  @logged
  greet() {
    return "Hello, " + this.greeting;
  }
}

function logged(target: any, name: string, descriptor: PropertyDecorator) {
  const fn = target[name];
  descriptor.value = function () {
    console.log(`Calling ${name}`);
    return fn.apply(this, arguments);
  };
}

console.log(new Greeter("Dave").greet());
// [LOG]: "Calling greet"
// [LOG]: "Hello, Dave"
```

참고로 데코레이터는 비표준임. 아직 안쓰는게 좋음.

## item 54 : 객체를 순회하는 노하우

## item 55 : DOM 계층 구조 이해하기

## item 56 : 정보를 감추는 목적으로 private 사용하지 않기

## item 57 : 소스맵을 사용하여 타입스크립트 디버깅 하기
