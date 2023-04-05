# 3주차

## 아이템 12 : 함수 표현식에 타입 적용하기

1. 함수 문장보다는 함수 표현식 사용하기

- 자바스크립트(타입스크립트)에서는 함수 문장과 함수 표현식을 다르게 인식
- 타입스크립트에서는 함수 문장보다는 표현식을 사용하는게 재사용 관점에서 좋음

  ```ts
  type DiceRollFn = (sides: number) => number

  const rollDice: DiceRollFn = sides => {...}
  // 매개변수부터 반환된 값까지 타입으로 선언해 재사용 가능함
  ```

- 불필요한 코드 반복을 줄일 수 있음

  ```ts
  // 함수 문장
  function add(a: number, b: number) {
    return a + b;
  }
  function sub(a: number, b: number) {
    return a - b;
  }
  function mul(a: number, b: number) {
    return a * b;
  }
  function div(a: number, b: number) {
    return a / b;
  }

  // 함수 표현식
  type BinaryFn = (a: number, b: number) => number;
  const add: BinaryFn = (a, b) => a + b;
  const sub: BinaryFn = (a, b) => a - b;
  const mul: BinaryFn = (a, b) => a * b;
  const div: BinaryFn = (a, b) => a / b;
  ```

2. 다른 함수의 시그니처를 참조하려면 typeof fn 사용
   ```ts
   async function checkedFetch(input: RequestInfo, init?: RequestInit) {
     const response = await fetch(input, init);
     if (!resoponse.ok) {
       throw new Error("error");
     }
     return response;
   }
   ```
   이 코드에서 fetch는 함수 시그니처가 이미 있는 자바스크립트 모듈이기 떄문에 아래 코드처럼 간결하게 수정할 수 있다.
   ```ts
   const checkFetch: typeof fetch = async (input, init) => {
     const response = await fetch(input, init);
     if (!response.ok) {
       throw new Error("error");
     }
     return response;
   };
   ```

## 아이템 13 : 타입과 인터페이스의 차이점 알기

1. 타입스크립트에서 타입을 정의하는 방법

- type
  ```ts
  type TState = {
    name: string;
    capital: string;
  };
  ```
- interface
  ```ts
  interface IState = {
    name: string
    capital: string
  }
  ```

2. 타입과 인터페이스 비슷한 점

- 명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태는 차이가 없다.
- 인덱스시그니처는 인터페이스와 타입 모두 사용할 수 있다.
- 함수 타입은 인터페이스나 타입으로 정의 가능
- 둘다 제네릭이 가능하다.
- 서로 상호 확장가능

3. 차이점

- 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지 못해 &과 타입을 사용해야한다.
-

## 아이템 14 : 타입 연산과 제너릭 사용으로 반복 줄이기
