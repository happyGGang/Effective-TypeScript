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

## 아이템 13 : 타입과 인터페이스의 차이점 알기

## 아이템 14 : 타입 연산과 제너릭 사용으로 반복 줄이기
