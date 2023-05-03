# 7주차 : 타입추론, 타입 설계

## item 26 : 타입 추론에 문맥이 어떻게 사용되는지 이해하기

1. 타입스크립트는 타입을 추론할때 단순히 값만 고려하는 것이 아닌 값이 존재하는 문맥까지 살핌. 이때 가끔 아래처럼 이상한 결과가 나올 수 있는데

   ```ts
   type Language = 'javascript' | 'typescript' | 'python';
   function setLanguage(language: Language){
      <!-- ... -->
   }

   setLanguage('javascript') // 정상

   //타입 추론은 변수의 할당 시점에 결정된다.
   // 그래서 이 값을 변수로 분리해 내면 할당 시점에서 타입을 추론
   let javascript = 'javascript'; // type string

   setLanguage(javascript);
   // 비정상) string은 javascript 타입에 할당 불가.
   ```

   타입 추론에 문맥이 어떻게 사용되는지 알면 대처 가능

   ```ts
   let javascript: Language = "javascript";
   //타입 선언에서 할당 가능한 값을 제한.
   setLanguage(javascript); // ok

   const typescript = "typescript";
   // const를 사용하면 보다 구체적으로 타입을 추론함 타입이 'typescript'
   setLanguage(typescript); //ok
   ```

2. 튜플 사용 조심하기

   ```ts
   function panTo(where: [number, number]) {
     <!-- ... -->
   }

   panTo([1, 2]) // ok

   const loc = [3, 4] // type number[]
   panTo(loc) // error
   ```

   위에 코드처럼 튜플 타입도 문맥과 값을 분리하면 타입 선언시점에 타입을 추론하기때문에 예상하지 못한 결과가 나올 수 있다. any로 빠른 수정도 가능하지만 아래처럼 고치거나

   ```ts
   function panTo(where: [number, number]) {
     <!-- ... -->
   }

   panTo([1, 2]) // ok

   const loc: [number, number] = [3, 4] // type number[]
   panTo(loc) // ok
   ```

   다음 코드처럼 고칠 수 있는대 const 는 단지 값이 가리키는 참조가 변하지 않는 다는 얕은 상수인 반면 as const는 내부까지 상수라는 것을 타입스크립트에 알려준다. 대신 해당 추론은 너무 과하게 정확함!
   그리고 만약 타입 정의에 실수가 있다면(튜플에 세번째 요소 추가하기 등등) 오류가 정의가 아닌 호출부에서 나기때문에 원인 파악이 어려움

   ```ts
   function panTo(where: [number, number]) {
     <!-- ... -->
   }

   panTo([1, 2]) // ok

   const loc = [3, 4] as const // type number[]
   panTo(loc) // type readonly [3, 4]
   ```

   오류를 any없이 잡으려면 아래처럼 리드온리 구문을 추가해야한다.

   ```ts
   function panTo(where: readonly[number, number]) {
    <!-- ... -->
   }

   panTo([1, 2]) // ok

   const loc = [3, 4] as const // type number[]
   panTo(loc) // type readonly [3, 4]
   ```

3. 객체 사용 시 주의할점

   ```ts
   type Language = 'javascript' | 'typescript' | 'python'

   interface GovernedLanguage {
    language : Language;
    organization: string;
   }

   function complain(language: GovernedLanguage) {
    <!-- ... -->
   }

   complain({language: 'javascript', organization: 'ms'}) //ok

   const ts = {
    language : 'typescript';
    organization: 'apple';
   }

   complain(ts) // error


   const ts: GovernedLanguage = {
      language : 'typescript';
      organization: 'apple';
    }

    complain(ts) // ok
   ```

   같은 맥락으로 문맥에서 값을 분리하는 문제는 객체에서 상수를 뽑아낼때도 발생해서 타입선언 추가 혹은 상수 단언으로 해결할 수 있다

4. 콜백 함수 사용 시 주의점

- 콜백을 다른 함수로 전달할 때, 타입스크립트는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용함.

  ```ts
  function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
    fn(Math.random(), Math.random());
  }

  callWithRandomNumbers((a, b) => {
    a; // type number
    b; // type number

    console.log(a + b);
  });
  ```

  여기서 콜백함수를 상수로 뽑아내면 문맥이 소실되어 에러가 발생한다.

  ```ts
  const fn = (a, b) => {
    a; // 암시적으로 any 할당
    b; // 암시적으로 any 할당
    console.log(a + b);
  };

  callWithRandomNumbers(fn);
  ```

  이런 경우 매개변수 타입을 추가해주거나 함수 표현식 전체에 타입 선언을 적용하면 된다.

  ```ts
  const fn = (a: number, b: number) => {
    a; // 암시적으로 any 할당
    b; // 암시적으로 any 할당
    console.log(a + b);
  };

  callWithRandomNumbers(fn);
  ```

## item 27 : 함수형 기법과 라이브러리로 타입 흐름 유지하기

1. 가독성, 명시적 타입구문 필요성을 줄일려면 직접 구현하는것보다 내장 메소드나, 서드파티 라이브러리 사용하자

## item 28 : 유효한 상태만 표현하는 타입 지향하기
