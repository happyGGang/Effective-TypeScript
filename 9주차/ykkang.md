# any 다루기

## item 38 : any 타입은 가능한 좁은 범위에서만 사용하기

## item 39 : any를 구체적으로 변형해서 사용하기

## item 40 : 함수 안으로 타입 단언문 감추기

## item 41 : any의 진화를 이해하기

## item 42 : 모르는 타입의 값에는 any 대신 unknown 사용하기

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
