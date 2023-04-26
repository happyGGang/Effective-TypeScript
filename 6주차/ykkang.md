# 6주차 : 타입 추론

## item 23 : 한꺼번에 객체 생성하기

1. 타입스크립트에서 객체를 만들때 객체 속성을 한번에 명세하는 것이 좋음

   ```ts
   const point = {}; // point type => {}
   point.x = 5; // error, point에 x 없음
   point.y = 10; // error, point에 y 없음

   const pointFix = {
     x: 5,
     y: 10,
   };
   pointFix.x = 10; // ok
   pointFix.y = 15; // ok
   ```

2. 전개연산자를 사용하자

   ```ts
   const person = {
     age: 0,
     name: "jeff",
   };

   const birth = {
     birth: new Date(),
   };

   const person_with_brithday = {}; // person_with_brithday type => {}
   Object.assign(person_with_brithday, birth, person);
   console.log(person_with_brithday.name); // error, 'name' 속성 존재 X.

   const person_with_brithday_fix = {
     ...person,
     ...birth,
   };

   //type is
   // {
   //     birth: Date;
   //     age: number;
   //     name: string;
   // }

   console.log(person_with_brithday_fix.name); // ok
   ```

3. 조건부 추가하기

- 전재연산자 사용시 조건부 적용 가능

  ```ts
  declare let hasMiddle: boolean;
  const firstLast = {
    first: "Harry",
    last: "Truman",
  };

  // 조건부 객체 추가
  const president = {
    ...firstLast,
    ...(hasMiddle ? { middle: "S" } : {}),
  };
  // {
  //     middle?: string | undefined;
  //     first: string;
  //     last: string;
  // }
  ```

## item 24 : 일관성 있는 별칭 사용하기

1. 별칭 난발 금지!

- 타입 별칭을 만들어 값을 수정하면 원본 객체도 수정되기 때문에 코드 제어 및 흐름 파악이 힘들다.

  ```ts
  const borough = {
    name: "Brooklyn",
    location: [40.688, -73.979],
  };
  const loc = borough.location;
  //별칭의 값을 변경 => 원래 속성값도 변경

  loc[0] = 0;
  console.log(borough);
  // {
  //   "name": "Brooklyn",
  //   "location": [
  //     0,
  //     -73.979
  //   ]
  // }
  ```

2. 그래서 변수 별칭을 사용할땐 디스트럭쳐링을 사용해 일관성있게 쓰자
3. 속성이 선택적일 경우 속성체크가 따로 필요하다

## item 25 : 비동기 코드에는 콜백 대신 asycn 함수 사용하기

1.  콜백지옥이란?
    실행 순서도 코드 순서와 반대, 보기도 빡심

    ````js
    fetchURL(url1, (res1) =>{
    fetchURL(url2, (res2) =>{
    fetchURL(url3, (res3) =>{
    console.log(1);
    });
    console.log(2);
    });
    console.log(3);
    });
    console.log(4);

        //로그
        //4
        //3
        //2
        //1
        ```

    ````

2.  프로미스(미래에 가능해질 어떤 것을 나타냄)

- 콜백지옥을 해결하기 위해 도입. 진행 순서도 코드 순서와 같아졌고 가독성도 굿

  ```js
  const pagePromise = fetch(url1);

  pagePromise
    .then((res1) => {
      return fetch(url2);
    })
    .then((res2) => {
      return fetch(url3);
    })
    .then((res3) => {
      return fetch(url4);
    })
    .catch((err) => {
      console.log(err);
    });
  ```

- Promise.all 써서 병렬로 표현도 가능함

  ```js
  const promise1 = Promise.resolve(3);
  const promise2 = 42;
  const promise3 = new Promise((resolve, reject) => {
    setTimeout(resolve, 100, "foo");
  });

  Promise.all([promise1, promise2, promise3]).then((values) => {
    console.log(values);
  });
  // Expected output: Array [3, 42, "foo"]
  ```

- promise.race

  - 콜백보다 코드작성, 타입추론 면에서 프로미스가 더 좋음!
  - 먼저 완료되는 것을 이행
  - 타입 구문 없어도 반환 타입 Promise<Response>로 추론

    ```ts
    async function parallelFetch() {
      // 모든 promise가 resolve 되면
      // promise all의 then이 호출

      // 각각의 fetch는 병렬로 처리
      const [res1, res2, res3] = await Promise.all([
        fetch(
          // 이때 ts는 각 res의 타입을 response로 추론
          url1
        ),
        fetch(url2),
        fetch(url3),
      ]);
    }
    ```

3. async await

- 프로미스보다는 코드가 더 간결하고 async는 항상 프로미스를 반환하도록 강제하기때문에 프로미스를 생성하기 보다는 async await를 쓰자

```js
async function asyncFetchExceptHandle() {
  try {
    const promise1 = await fetch(url1);
    // fetch가 완료될때 까지 기다린다.
    const promise2 = await fetch(url2);
    // fetch가 완료될때 까지 기다린다.
    const promise3 = await fetch(url3);
    // fetch가 완료될때 까지 기다린다.
  } catch (err) {
    console.log(err);
    // try내 fetch들 에서 reject 발생시 catch 내에서 에러 핸들링
  }
}
```
