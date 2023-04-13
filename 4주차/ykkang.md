# 4주차

## item 15: 동적 데이터에 인덱스 시그니처 사용하기

1. 런타임때까지 객체의 속성타입을 모를때만 인덱스 시그니처를 사용하자
2. 키가 얼마나 많이 있는지 모른다면 선택적 필드 혹은 유니온 타입으로 사용하고 인덱스 시그니처는 피하자
3. 가능하다면 인터페이스, 레코드, 매핑된 타입을 쓰자.

## item 16: number인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

1. 숫자는 key로 사용 불가능
2. 배열의 타입은 object

   ```ts
   const arr = [1, 2, 3];
   console.log(typeof arr); // object\

   // 배열의 타입은 객체, 그래서 숫자 인덱스를 사용하는것이 맞지만
   // 인덱스 요소들은 숫자로 반환되어 문자열 키로 접근해도 배열 요소에 접근 가능
   console.log(arr["0"]); // 1
   ```

3. 타입스크립트는 숫자형 키를 허용

   ```ts
   //lib.es%.d.ts

   interface Array<T> {
     [n: number]: T;
   }
   ```

4. 런타임에서는 문자열 키로 인식하기 때문에 작동하지는 않음

   ```ts
   const xs = [1, 2, 3];
   const x0 = x[0]; // ok
   const x1 = x["1"]; // 타입에러가 나지만 실제로 동작하지는 않음
   ```

5. 인덱스 시그니처에 number를 사용하기보다 Array나 ArrayLike 타입을 쓰자

## item 17: 변경 관련된 오류 방지를 위해 readOnly 사용하기

1. readonly

   - 아래 코드를 실행하면 오류가 발생하는데

   ```ts
   function arraySum(arr: readonly number[]) {
     // ~~ readonly numner[] 형식에 pop 속성이 없음
     let sum = 0,
       num;
     while ((num = arr.pop()) !== undefined) {
       sum += num;
     }
     return sum;
   }

   function printTriangles(n: number) {
     const nums = [];
     for (let i = 0; i < n; i++) {
       nums.push(i);
       console.log(arraySun(nums));
     }
   }
   ```

   - readonly를 붙이면 배열의 요소를 읽을 순 있지만 쓸수없고, length를 읽을 수 있지만 배열 변경은 불가해서
     배열을 변경하는 메서드를 사용할 수 없다. 결국 number[]는 readonly number[]보다 기능이 많아서 readonly number[]의 서브타입임.

   - 고로 변경가능한 배열을 readonly에 할당할 수 있지만 반대는 불가능

2. 파라미터를 readonly로 선언하면 생기는일

   - ts가 함수내 매개변수가 변경이 일어나는지 확인
   - 호출하는 쪽에서 함수가 매개 변수를 변경하지 않는다는 보장을 받음
   - 호출쪽에서 함수에 readonly 넣을 수도 있음

3. readonly를 쓸려면 고로 배열을 수정안하면 된다

   ````ts
   function arraySum(arr: readonly number[]) {
      // ~~ readonly numner[] 형식에 pop 속성이 없음
      let sum = 0;
      for(const num of arr) {
         sum += num
      }
      return sum
   }


   function printTriangles(n: number) {
      const nums = []
      for(let i = 0; i < n;  i++) {
         nums.push(i)
         console.log(arraySun(nums))
      }
   }
   ```
   ````

4. 매개변수를 변경하지 않는다면 readonly를 쓰자
5. readonly는 얕게 동작한다

## item 18: 매핑된 타입을 사용하게 값을 동기화하기

1. 새로운 속성이 추가될때 매핑된 타입을 사용해서 관련된 값과 타입을 동기화 시켜야함.
2. 인터페이스에 새로운 속성을 추가할 때 선택을 강제하도록 매핑된 타입을 고려해야함.
