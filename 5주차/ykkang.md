# 5주차 : 타입 추론

## item 19 : 추론 가능한 타입을 사용해 장황한 코드 방지하기

1.  추론 가능한 타입은 타입구문 쓰지말자
    ```ts
    // 이렇게 해도 되지만 추론 가능하다면
    const num: number = 1; // (하수)
    // 이렇게 쓰자
    const num = 1; // (고수)
    ```
    타입체커는 우리가 생각하는 것 보다 더 구체적으로 타입을 추론함
    ```ts
    const axis1: string = "x"; //type 'string'
    const axis2 = "y"; // type 'y' -> const로 선언했기 때문에 타입이 string X
    const test: typeof axis2 = "x"; // error
    ```
2.  타입이 변경될수도 있다면(함수 파라미터로 object 할당받을때) 디스트럭쳐링 쓰자.

    ```ts
    interface Product {
      id: number;
      name: string;
      price: number;
    }

    // 이거보단
    function logProduct(product: Product) {
      const id: number = product.id;
      const name: string = product.name;
      const price: number = product.price;
      console.log(id, name, price);
    }

    // 이렇게 디스트럭처링
    function logProductFix(product: Product) {
      const { id, name, price } = product;
      //typeof id = number;
      //typeof name = string;
      //typeof price = number;
      console.log(id, name, price);
    }
    ```

3.  추론 가능한 경우라도 함수 반환이나 객체 리터럴에는 타입 명시를 고려하자

    - 결국 이상적인 타입스크립트 = 함수 or 메서드 구현부에 타입구문 X, 함수 내 로컬 변수 할당 시 타입 구문 하나하나 명시 X
    - 함수를 읽는 사람이 로직에 집중할 수 있도록 하기 위함!
    - 이미 타입 정보가 명시된 라이브러리를 쓸 경우 파라미터 타입 정의 하나하나 쓸 필요 없음
    - 그러나 타입체커가 추론할수있는 상황에도 타입을 명시해야할때가 있음
      - 객체 리터럴 할당할때(잉여속성체크 하고싶을때)
      - 오류 방지를 위해 함수 리턴 타입명시

4.  no-inferrable-types 쓰면 타입구문이 필요한지 안한지 확인 가능

## item 20 : 다른 타입에는 다른 변수 사용하기

1. 변수값은 바뀔 수 있지만 그 타입은 변하지 않음
2. 가려지는 변수 사용 지양
3. let 대신 const로 변수를 할당하기에 더 유리, const로 변수를 할당하면 더욱 strict한 타입 체킹을 유도

## item 21 : 타입 넓히기

### 타입 체커는 지정된 단일값을 가지고 할당 가능한 값들의 집합을 유추(타입 넓히기)

1. 타입스크립트가 아무리 영리해도 타입넓히기를 통해 추론한 타입이 언제나 옳을 수 없음.
2. 일반적 규칙으로 변수가 선언된 후로는 타입이 바뀌지 않아야함.
3. 만능은 아니지만 const 로 타입 넓히기 제어하기

```ts
interface Vector2 {
  x: number;
  y: number;
}

const getComponent = (vector: Vector2, axis: "x" | "y") => {
  return vector[axis];
};

const vector: Vector2 = {
  x: 3,
  y: 4,
};

let x = "x";
// 타입 추론에 의해 x 의 타입은 string;
console.log(getComponent(vector, x));
// 'x' | 'y' 파라미터로 허용하는 함수에 string 타입과 함께 호출시 에러가 발생.

//넓히기 과정을 제어할 수 있는 한가지 방법으로 const 로 변수를 선언하는 것.
const y = "y"; // 타입 추론에 의해 y 의 타입은 'y';
console.log(getComponent(vector, y));
//정상

// const 도 만능은 아님
const v = {
  x: 1, // v.x type number
};
v.x = 3;
v.x = "3"; // error
```

### 타입추론 강도 제어

4. 명시적 타입 구문으로 기본동작 재정의하기
   ```ts
   const v: { x: 1 | 3 | 5 } = {
     x: 1,
   };
   ```
5. as const로 기본동작 재정의

   ```ts
   const v1 = {
     x: 1,
     y: 2,
   };
   // 타입은 {x : number; y: number;}

   const v2 = {
     x: 1 as const,
     y: 2,
   };
   // 타입은 {x: 1; y: number;}

   const v3 = {
     x: 1,
     y: 2,
   } as const;
   // 타입은 { readonly x: 1; readonly y: 1;}
   //as const  = 타입스크립트는 최대한 좁은 타입으로 타입을 추론
   const a1 = [1, 2, 3];
   // 타입이 number[];
   const a2 = [1, 2, 3] as const;
   // 타입이 readonly number [1, 2, 3];
   ```

## item 22 : 타입 좁히기

넓은 타입에서 > 좁은 타입으로 진행하는 과정

1. 타입 좁히는 방법

   - null 체크
     ```ts
     const el = document.getElementById("test");
     // type HTMLElement | null
     if (el) {
       el.innerHTML = "test element!";
       // el type HTMLElement
       //const el: HTMLElement
     } else {
       //const el: null
       // el type null
       alert("cannot found test element");
     }
     ```
   - 분기문 예외처리, 함수 반환

     ```ts
     const el = document.getElementById("test");
     // type HTMLElement | null

     if (el === null) {
       throw new Error("cannot found test element");
     }

     el.innerHTML = "test element!";
     // el type HTMLElement
     ```

   - interfaceof
     ```ts
     function contains(test: string, search: string | RegExp) {
       if (search instanceof RegExp) {
         return search.test(test);
         // search type RegExp
       }
       return test.includes(search);
       // search type string
     }
     ```
   - (object)속성 체크
     ```ts
     function pickAB(ab: A | B) {
       if ("a" in ab) {
         return ab["a"]; // ab type 'A'
       } else {
         return ab["b"]; // ab type 'B'
       }
     }
     ```
   - Array.isArray 같은 일부 내장함수
     ```ts
     function contains_array(text: string, terms: string | string[]) {
       const term_list = Array.isArray(terms) ? terms : [terms];
       console.log(term_list);
       // term_list type string[];
     }
     ```
     ### 초기값 오류나 예외처리 실수도 타입 좁히기가 제대로 동작하지 않을수 있음. 그때 사용하는 타입 좁히기 방법
   - 명시적 태그(태그된 유니온, 구별된 유니온이라고 불림)

     ```ts
     interface UpLinkSpeed {
       type: "uplink";
       bit_per_sec: number;
       res_data: string;
     }

     interface DownLinkSpeed {
       type: "downlink";
       bit_per_sec: number;
       req_data: string;
     }

     type EthernetSpeed = UpLinkSpeed | DownLinkSpeed;

     function checkEthernetSpeed(speed_data: EthernetSpeed) {
       if (speed_data.type === "uplink") {
         console.log(speed_data.res_data);
         // speed_data type => 'UpLinkSpeed'
       } else {
         console.log(speed_data.req_data);
         // speed_data type => 'DownLinkSpeed'
       }
     }
     ```

   - 타입 가드(타입을 좁히기 위해 별도 함수를 정의하는 방법)

     ```ts
     // is HTMLInputElement룰 반환하는 함수 = 타입가드
     function isInputElement(el: HTMLElement): el is HTMLInputElement {
       return "value" in el;
     }

     function getElementContents(el: HTMLElement) {
       if (isInputElement(el)) {
         return el.value;
       }
       return el.textContent;
     }
     ```

2. 타입 가드
   - 참고 : https://chanhuiseok.github.io/posts/ts-2/
