# 1주차 학습

## item1 : 타입스크립트와 자바스크립트의 관계 이해하기

1. 타입스크립트는 자비스크립트의 **superset**
   - 모든 자바스크립트 프로그램이 타입스크립트 O, 반대는 X <br>
     아래코드 처럼 `:string` 과 같은 타입구문을 사용하는 순간부터 자바스크립 트는 타입스크립트 영역으로 들어감
     `typescript
function greet(who: string) {
    console.log('hello', who)
}
`
2. 타입스크립트는 런타임 오류를 발생시키는 코드를 찾아냄. but 100%는 아님

   - 가령 아래와 같은 코드가 있으면

     ```typescript
     let city = "new yorl city";
     console.log(city.toUppercase());
     ```

     타입구문을 따로 적지 않았지만 타입체커가 초깃값으로 부터 **타입을 추론**하고

     > 님 toUppercase 속성은 string 형식에 없는데 ㄹㅇ 쓸꺼임?

     라고 묻는다. 쏘 유용하다.

   - 또한 오류가 발생하지는 않지만 우리의 의도와 다르게 동작하는 코드가 있다면 그것 또한 찾아내 준다. 아래와 같은 코드에서

     ```typescript
     const states = [
       { name: "윤경", home: "왜관" },
       { name: "민우", home: "대구" },
     ];

     for (const state of states) {
       console.log(state.hame);
     }
     ```

     우리는 분명 `hame`을 의도하지 않았다. 하지만 오류는 발생하지 않고 `undefined`를 콘솔로 뱉낼거임. 이럴때 타입체커는

     > 님 hame은 없어,,home 맞지? 그거 쓸래?

     하고 오류도 잡아줄 뿐만 아니라 적절한 해결책까지 제시해준다.

   - 이렇게 타입스크립트는 **정적 타입시스템**이기 때문에 런타임에 오류를 발생시킬 코드를 미리 찾아낸다. 그리고 위의 예시 코드처럼 타입 구문을 추가하지 않아도 어느정도 오류를 찾아낼 수 있다. 그러나 여기서 주의할 것은 말그대로 어느정도의 오류를 찾아내는거지 **100% 모든 오류를 찾아내지 못한다.** 예시를 보자.

     ```typescript
     const states = [
       { name: "윤경", hame: "왜관" },
       { name: "민우", hame: "대구" },
     ];

     for (const state of states) {
       console.log(state.home);
     }
     ```

     위 코드에선 타입체커가 `hame`이 오타인지, `home`이 오타인지 구분하지 못한다. 오류의 원인은 알아내겠지만 오타를 구분할 수 없기때문에 제대로된 해결책을 제시해줄 수 없다.
     그래서 최대한 많은 오류를 발견해내기 위해서는 타입 구문을 추가해 확률을 높혀줘야한다.

     ```typescript
        interface State = [
         name: string,
         home: string
        ]

         const states: State[] = [
           {name: '윤경', hame: '왜관'},
           {name: '민우', hame: '대구'}
         ]

         for (const state of states) {
           console.log(state.home)
         }
     ```

## item2 : 타입스크립트 설정 이해하기

1.  타입스크립트 설정 방법

    - 타입스크립트 설정 방법에는 2가지가 있음. <br>

      1. 하나는 커맨드라인을 사용하는 방법
      2. tsconfig.json 사용하는 방법(설정 파일)

      이 두가지 중 가급적 설정파일을 사용하는게 더 좋다. (협업할때 좋음)

2.  꼭 알아야할 타입스크립트 설정 2가지 - noImplicitAny <br>

    - 변수들이 미리 정의된 타입을 가져야하는지에 여부에 대한 설정이다. <br>
    - noImplicitAny <br>
      noImplicitAny 해제하면 아래 코드처럼 암시적 any 타입으로 간주될 수 있다. 이때 any 타입은 유용하지만 **타입쉴드를 벗기는 행위**이기 때문에 조심해야한다. 그러므로 의도적으로 `: any` 라고 선언해주거나 noImplicitAny를 설정해 분명한 타입을 사용하는 것이 바람직하다.
    - strictNullChecks <br>
      null과 undefined가 모든 타입에 허용되는지 확인하는 설정이다.<br>
      null과 undefined에 관련된 오류를 찾아내는것에 도움이 많이 되지만 만약 해당 오류가 발생했을지 null 또는 undefined에rk 어디서부터 왔는지 찾아야해 코드 작성이 힘들어진다.

    - 모든 체크를 설정하려면 strict 모드를 사용하면 된다.

## item3 : 코드 생성과 타입이 관계없음을 이해하기

1. 타입스크립트 컴파일러의 두가지 역할

   - 최신 타입스크립트와 자바스크립트가 브라우저에서 동작하도록 구버전으로 **트랜스파일**
   - 타입 오류 체크
   - 위 두가지 역할은 서로 **독립적**이여서 타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향 X, 자바스크립트 런타입 시점에도 영향 X

2. 타입 오류가 있는 코드도 컴파일 가능
   - 타입체크와 컴파일은 독립적으로 동작하기 때문에 타입 오류가 있어도 컴파일을 멈추지 않는다.
3. 런타임에는 타입 체크가 불가능

   - 아래 코드에서 `instanceof`는 런타입시 체크한다. 하지만 `Rectangle`은 타입이기 때문에 런타임때는 아무 역할도 하지 못한다. 즉 타입스크립트는 **erasable** 하고 실제 컴바일 과정에서 인터페이스, 타입, 타입구문은 모두 제거된다.

     ```typescript
     interface Square {
       width: number;
     }
     interface Rectangle extends Square {
       height: number;
     }
     type Shape = Square | Rectangle;

     function calculateArea(shape: Shape) {
       if (shape instanceof Rectangle) {
         return shape.width * shape.height;
       } else {
         return shape.width * shape.width;
       }
     }
     ```

4. 타입 연산은 런타임에 영향 X

   - string 또는 number 타입을 항상 Number로 정제해야하는 상황을 가정해보자.

     ```typescript
     function asNumber(val: number | string): number {
       return val as Number;
     }
     ```

     위 코드가 제대로 정제를 정제해줄까? 아니다. 런타임에는 결국 아래와 같은 코드만 남기때문에 정제되지 않는다.

     ```javascript
     function asNumber(val) {
       return val;
     }
     ```

     다시 말해 타입 연산이 런타임에 영향을 미치지 못하기 때문에 아래 처럼 수정해야한다.

     ```typescript
     function asNumber(val: number | string): number {
       return typeof val === "string" ? Number(val) : val;
     }
     ```

5. 런타임 타입은 선언된 타입과 다를 수 있음
   - 아래 value는 boolean이라고 작성은 했지만 실제로 무조건 boolean값만 함수에 전달될것이라는 보장은 없다. api response가 변경될 수도 있기 때문이다.
     ```typescript
     function setLightSwitch(value: boolean) {
       switch (value) {
         case true:
           console.log(1);
           break;
         case false:
           console.log(2);
           break;
         default:
           console.log("실행되지않을까 걱정");
       }
     }
     ```
     실제로 boolean 값이 아닌 다른 타입의 값을 받아오더라도 런타임때는 아래와 같은 코드가 되기때문에 오류가 나지않고 마지막 `console.log()`를 실행한다.
     ```javascript
     function setLightSwitch(value) {
       switch (value) {
         case true:
           console.log(1);
           break;
         case false:
           console.log(2);
           break;
         default:
           console.log("실행되지않을까 걱정");
       }
     }
     ```
6. 타입스크립트 타입으로는 함수를 오버로드 불가능

   - C++ 같은 언어는 동일한 이름에 매개변수만 다른 여러 버전의 함수를 만들 수 있지만 타입스크립트에서는 불가능하다.

     ```typescript
     function add(a: number, b: number) {
       return a + b;
     }

     function add(a: string, b: string) {
       return a + b;
     }
     ```

     이렇게 작성하더라도 결국 타입 수준에서만 오버로딩기능이 가능한거지 컴파일 이후에는 아래와 같은 코드가 남기때문에 하나의 함수를 여러번 선언하는거에 불과하고 결론적으로 구현체는 하나가 된다.

     ```javascript
     function add(a, b) {
       return a + b;
     }

     function add(a, b) {
       return a + b;
     }

     const two = add(1, 1);
     const three = add("1", "2");
     ```

7. 타입스크립트 타입은 런타임 성능에 영향 X

   - 앞서 계속 말했듯 타입, 인터페이스, 타입연산자들은 자바스크립트로 변환되는 시점에 제거되기 때문에 런타임 성능에 영향을 주지 않는다.

   - 런타임 *오버헤드가 없는대신 빌드타임 오버헤드가 있는대 일반적으로 상당히 빠른편이고 *증분 빌드를 한다면 더욱 체감된다.
     > **1. 오버헤드** <br>
     > 특정 기능을 수행하는데 드는 간접적인 시간으로 10초 걸리는 기능이 간접적 이유로 20초가 걸렸다면 오버헤드는 10초다.<br> **2. 증분 빌드**<br>
     > 변경된 부분만 찾아서 빌드하는 방법

## item4 : 구조적 타이핑에 익숙해지기

1. 덕 타이핑과 구조적 타이핑

   - 자바스크립트는 본질적으로 덕 타이핑 기반이다. 어떤 함수의 매개변수 값이 모두 제대로만 주워진다면 그 값이 어떻게 만들어졌는지는 신경 쓰지 않고 사용한다. 이런 자바스크립트의 성질때문에 타입스크립트는 구조적 타이핑, 즉 코드 관점에서 타입이 서로 호환되는지 여부를 판단하고 괜찮다면 모델링을 한다.

     ```typescript
     interface Vector2D {
       x: number;
       y: number;
     }

     function calculateLength(v: Vector2D) {
       return Math.sqrt(v.x * v.x + v.y * v.y);
     }

     interface NameVector {
       name: string;
       x: number;
       y: number;
     }

     const v: NameVector = { x: 3, y: 4, name: "Zee" };
     calculateLength(v);
     ```

     위 함수 실행결과는 정상적으로 5가 도출된다. Vector2D와 NameVector의 관계에 대해 전혀 선언하지 않았지만 타입스크립트는 변수가 가지고 있는 모양에 집중하기 때문에 오류가 발생하지 않고 정상적으로 동작한다.

   - 구조적 타이핑때문에 문제가 발생하기도 하는데 방금 코드에 3D 백터를 추가해보면

     ```typescript
     interface Vector2D {
       x: number;
       y: number;
     }

     function calculateLength(v: Vector2D) {
       return Math.sqrt(v.x * v.x + v.y * v.y);
     }

     interface NameVector {
       name: string;
       x: number;
       y: number;
     }

     const v: NameVector = { x: 3, y: 4, name: "Zee" };
     calculateLength(v);

     interface Vector3D {
       x: number;
       y: number;
       z: number;
     }

     function nomalize(v: Vector3D) {
       const length = calculateLength(v);
       return {
         x: v.x / length
         y: v.y / length
         z: v.z / length
       }
     }

     nomalize({x: 3, y: 4, z: 5})
     ```

     예상대로라면 위 함수는 1을 출력해야하지만 1.41을 출력할 것이다. `calculateLength()`가 `Vector2D` 를 받도록 선언되었지만 구조적 타이핑 관점에서 `Vector3D`도 x와 y가 있어서 `Vector2D`와 호환되기 때문이다. 우리가 흔히 함수를 작성할때 호출에 사용되는 매개변수 속성들이 매개변수 타입에 선언된 속성만 가질 거라고 생각한다. 이러한 타입을 **봉인된 타입** 이라고 하는데 타입스크립트는 **열린 타입** 이기 때문에 위와 같은 결과가 발생한다.

## item5 : any 타입 지양하기

1. any 타입에는 안전성이 없음
   - 어떤 타입도 들어올 수 있기때문에 예상하지 못한 오류들이 발생할 여지가 많다
     any 타입은 때때로 유용하지만 특별한 경우를 제외하고는 쓰지 않는게 좋다.
2. any 는 함수 시그니처를 무시함

   - 아래 코드처럼 자바스크립트는 때때로 암시적 타입변환이 이뤄지기 때문에 함수 시그니처가 무시된다.

   ```typescript
   function calculateAge(birthDate: Date): number {
     ...
   }

   let birthDate: any = '1990-01-01'
   calculateAge(birthDate)
   ```

3. any 타입에는 언어 서비스가 적용되지 않음
   - 타입스크립트 언어 서비스는 각 타입별로 자동완성 기능과 적절한 도움말을 제공하지만 any 타입에는 해당 언어 서비스를 제공하지 않음
