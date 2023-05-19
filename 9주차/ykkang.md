# any 다루기

## item 38 : any 타입은 가능한 좁은 범위에서만 사용하기

1. 함수와 관련된 any

   ```ts
   interface Foo {
     _brand: "foo";
   }

   interface Bar {
     _brand: "bar";
   }

   function processBar(b: Bar) {
     console.log(b);
   }

   function expressionReturningFoo(): Foo {
     /**  */
     return { _brand: "foo" };
   }

   function f1() {
     const x: any = expressionReturningFoo();
     // 쓰지말자
     processBar(x);
   }

   function f2() {
     const x = expressionReturningFoo();
     processBar(x as any); // 매개변수에만 사용
     // any가 함수 호출에만 영향을 주기때문에 이렇게 쓰자
   }
   ```

2. 객체와 관련된 any

   ```ts
   interface Config {
     a: number;
     b: number;
     c: {
       key: string;
     };
   }

   const value = 1;

   const config: Config = {
     a: 1,
     b: 2,
     c: {
       key: value,
     },
   } as any; // 쓰지말자
   // a와 b에도 any가 적용되어버림

   //추천 하는 방식
   const config2: Config = {
     a: 1,
     b: 2,
     c: {
       key: value as any, // 최소한의 범위로만 사용
     },
   };
   ```

3. 강제로 타입을 제거하려면 any를 쓰지말고 차라리 @ts-ignore를 사용하자

## item 39 : any를 구체적으로 변형해서 사용하기

1. any는 범위가 너무 넓은 타입
2. any보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높음
3. any 타입의 값을 그대로 정규식이나 함수에 넣는 것은 비추
   ```ts
   function getLenBad(array: any) {
     // no
     return array.length;
   }
   function getLen(array: any[]) {
     // more better
     return array.length;
   }
   ```
4. 정확하지 않은 객체를 파라미터로 받을때도 아래처럼 쓰자

   ```ts
   function getSomeObjKeysBad(obj: any) {
     // no
     return Object.keys(obj);
   }

   function getSomeObjKeys(obj: { [key: string]: any }) {
     // more better
     return Object.keys(obj);
   }
   ```

## item 40 : 함수 안으로 타입 단언문 감추기

1. 함수 모든 부분을 안전한 타입으로 설계하는 것이 이상적임
2. 하지만 불필요한 예외 상황까지 생각해가며 타입을 구성할 필요는 없음
3. 함수 내부에 타입 단언을 쓰고 함수 외부로 들어다는 타입정의를 정확히 명시하는 정도로 타협하자.
4. 프로젝트 전반에 위험한 타입 단언문이 노출되는것보다, 제대로 타입이 정의된 함수 안으로 숨기는것이 더 좋은 설계임

   ```ts
   declare function shallowEqual(a: any, b: any): boolean;

   function cacheList<T extends Function>(fn: T): T {
     let lastArgs: any[] | null = null;
     let lastResult: any;
     return function (...args: any[]) {
       if (!lastArgs || !shallowEqual(lastArgs, args)) {
         lastResult = fn(...args);
         lastArgs = args;
       }
       return lastResult;
     } as unknown as T;
   }
   ```

## item 41 : any의 진화를 이해하기

1. 일반적으로 타입스크립트에서 변수 타입은 변수 선언할때 결정
2. 그 이후에 정제될 수 있지만 새로운 값이 추가되도록 확장 불가능
3. 이때 any는 예외
   ```ts
   function range(start: number, limit: number) {
     const out = []; // any[]
     for (let i = start; i < limit; i++) {
       out.push(i); // number[]로 넘버 들어가는 순간 진화함
     }
     return out; // 반환 타입이 number[]로 추론됨.
   }
   ```
4. 이상한거 넣으면 그대로 계속 확장됨
   ```ts
   const result = []; // any[]
   result.push("a"); // string[]
   result.push(1); // (string | number) []
   ```
5. 분기에 따라서도 변함

   ```ts
   let val; // any
   if (Math.random() < 0.5) {
     val = /hello/;
     val; //타입이 RegEx
   } else {
     val = 12;
     val; //타입이 number
   }
   console.log(val); // 타입이 RegEx | number
   ```

6. 변수 초기값이 null인 경우에도 변함
   ```ts
   let val2 = null;
   try {
     someDangerousWork();
     val2 = 12; // number
     val2; // number
   } catch (e) {
     console.warn(e);
     val2; // number | null
   }
   val2; // number | null
   ```
7. 변수 선언 시 타입을 any로 명시하면 any의 진화 안함.
   ```ts
   let val: any; // any
   val = 12; // any
   ```
8. 암시적 any 상태인 변수에 어떠한 할당도 하지 않고 사용하려고 하면 암시적 any 오류 발생함

   ```ts
   function range(start: number, limit: number) {
     const out = [];
     // 오류 Variable 'out' implicitly has type 'any[]' in some locations where its type cannot be determined.(7034)

     if (start === limit) {
       return out; // 오류 Variable 'out' implicitly has an 'any[]' type.(7005)
     }

     for (let i = start; i < limit; i++) {
       out.push(i);
     }
     return out;
   }
   ```

9. 암시적 any타입은 함수 호출을 거쳐도 진화하지 않음.

   ```ts
   function makeSquares(start: number, limit: number) {
     const out = [];
     //Variable 'out' implicitly has type 'any[]' in some locations where its type cannot be determined.

     range(start, limit).forEach((i) => {
       out.push(i * i);
     });
     return out;
     //Variable 'out' implicitly has an 'any[]' type.(7005)
   }
   ```

## item 42 : 모르는 타입의 값에는 any 대신 unknown 사용하기

1. any 타입에 문제가 발생하는 이유 두가지

- 어떤 타입이든 할당 가능
- 어떤 타입으로도 할당 가능
- 예시로 YAML 파서인 parseYAML 함수를 보면

  ```ts
  // 함수의 반환시 any 지양해야하는대 써버렸음
  function parseYAML(yaml: string): any {
    // ...
  }

  const book = parseYAML(`
      name: asdfsafd
      author: abbbb
  `);

  alert(book.title);
  // 타입체커는 오류 발견 못해도 런타임때 undefined 에러 있을 수도
  book();
  // 타입체커는 오류 발견 못해도 런타임에 함수가 아니라고 오류가 발생할 수도
  ```

2. 그래서 any 보다는 unknown이 좋음

   ```ts
   // 함수의 반환값 unknown
   function parseYAMLFix(yaml: string): unknown {
     return parseYAML(yaml);
   }

   const book = parseYAMLFix(`
       name: asdfsafd
       author: abbbb
   `);

   alert(book.title); // 타입 채킹에서 오류 발생. Object is of type 'unknown'.
   book(); // 타입 채킹에서 오류 발생. Object is of type 'unknown'.
   ```

3. unknown은 무조건 타입을 좁혀서 사용해야 하는 의무가 있는 반면,
   any는 타입을 좁혀서 사용하지 않아도 되서 자유롭다는 차이점이 있기때문임.

4. 그래서 unknown을 사용하려면 적절한 타입으로 변환하도록 강제해야함

   ```ts
   function parseYAMLFix(yaml: string): unknown {
     return parseYAML(yaml);
   }

   interface Book {
     name: string;
     author: string;
   }

   const book = parseYAMLFix(`
       name: asdfsafd
       author: abbbb
   `) as Book; // unknown => Book 타입으로 단언

   alert(book.title); // 오류. Book 타입에는 title 속성은 없음.
   book(); // 오류. Book 타입은 함수가 아님.
   ```

5. 변수 선언과 관련된 unknown

- 위에 처럼 타입 단언만이 unknown을 원하는 타입으로 정제하는 유일한 방법이 아님
- instanceOf를 체크한 후 변경하는 방법도 있음

  ```ts
  interface Reature {
    id?: string | number;
    geometry: Geometry;
    properties: unknown;
  }
  function processValue(val: unknown) {
    if (val instanceof Date) {
      val.getDate(); // val이 Date 타입.
    }
  }
  ```

- 사용자 정의 타입가드도 방법중 하나임.

  ```ts
  function isBook(val: unknown): val is Book {
    return (
      typeof val === "object" &&
      val !== null &&
      "name" in val &&
      "author" in val
    );
  }

  function processValue2(val: unknown) {
    if (isBook(val)) {
      console.log(val.name); // 타입이 Book
    }
  }
  ```

6. 이중 단언문에 any 대신 unknown 쓰기

```ts
declare const foo: Foo;
let barAny = foo as any as Bar; // no
let barUnk = foo as unknown as Bar; // more better
```

7. unknown 과 비슷한 object 또는 {}

- object 또는 {}는 unknown만큼 넓지만 약간 좁음
- object 또는 {}는 unknown를 제외한 모든 값을 포함 가능
- unknown를 타입이 도입되기전까지 많이 사용됨. 요즘은 거의 안씀
- 만약에 null 이나 unknown이 정말 불가능하다고 생각할때 쓰자.

## item 43 : 몽키 패치보다는 안전한 타입을 사용하기

1. 몽키 패치

- 자바스크립트의 유연함 덕분에 개발시 윈도우나 도큐먼트 객체에 임의의 속성을 만들어 값을 할당, 전역변수처럼 사용하는대 이것을 몽키 패치라고함

  ```ts
  window.monkey = "Tamarin";
  document.monkey = "Howler";

  const el = document.getElementById("colobus");
  el.home = "tree";
  ```

2. 위의 방식은 좋은 설계가 아님. 타입체커는 도큐먼트와 HTMLelement의 내장 속성에 대해 알고있지만 임의로 추가한 속성에 대해서는 알지 못함.

```ts
document.monkey = "Tamarin";
// error) Property 'monky' does not exist on type 'Window & typeof globalThis'.(2339)

// 단언으로 해결할 수 있지만
// 타입 안정성을 상실하고 언어서비스 지원 X
(document as any).monkey = "Tamarin"; // ok
```

3. 그래서 도큐먼트와 돔에서부터 데이터를 분리하는것이 바람직함.

4. 하지만 꼭 내장타입에 데이터를 저장해야하는 경우 차선책 2가지가 있음

- 인터페이스 특수 기능중 하나인 보강 사용

  ```ts
  interface Document {
    monkey: string;
  }

  document.monkey = "monky patch with interface augmentation"; // ok

  // 오타나 잘못된 타입의 할당에 대한 오류 보여줌
  // 속성에 대한 주석 가능
  // 속성 자동완성 지원
  // 몽키패치가 적용된 부분 기록됨
  ```

  - 추가로 모듈관점에서 제대로 동작하게 하려면 global 선언 추가해야함

    ```ts
    export {};
    declare global {
      interface Document {
        /** 몽키 패치 */
        monkey: string;
      }
    }

    document.monkey = "Tamarin"; // ok
    ```

  - 보강은 전역적으로 사용되기때문에 코드의 다른 부분이나 라이브러리로부터 분리 불가능. 어플리케이션 실행동안 속성을 할당하면 보강 적용방법 없음.

- 구체적인 타입 단언문 사용
  ```ts
  interface MonkeyDocument extends Document {
    monkey: string;
  }
  (document as MonkeyDocument).monkey = "Macaque";
  ```

## item 44 : 타입 커버리지를 추적하여 타입 안전성 사용하기

1. noImplicitAny (변수들이 미리 정의된 타입을 가져야하는지)를 설정하고 모든 암시적 any 대신 타입 구문을 추가해도 아래와 같이 모든 위험으로부터 안전하다고 할 수 없다.

- 명시적 any 타입
  - any타입의 범위를 좁히고 구체적으로 만들어도 여전히 any임.
- 서드파티 타입 선언
  - @types 선언 파일로부터 any타입이 전파되기 때문에 결국 noImplicitAny를 설정하고 any를 사용하지 않아도 여전히 any타입은 영향을 미침

2. 위 문제들을 해결하기위해 프로젝트에 any 갯수를 추적하는것도 좋은 방법임

```
npx type-coverage
9985 / 1011 98.96%

npx type-coverage --detail
<!-- any 타입 쓴 경로 알려줌 -->
```
