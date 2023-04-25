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
