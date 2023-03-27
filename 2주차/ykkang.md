# 2주차 학습 : 타입스크립트의 타입 시스템

## item6 : 편집기를 사용하여 타입 시스템 탐색하기

1. 타입스크립트를 설치하면 실행할 수 있는 것

   - 타입스크립트 컴파일러(주된 목적)
   - 타입스크립트 서버(언어 서비스 제공)

2. 타입스크립트 언어 서비스
   - 코드 자동완성
   - 명세 검사
   - 검색
   - 리팩터링

## item7 : 타입이 값들의 집합이라고 생각하기

1. never type
   - 아무것도 포함하지 않는 공집함.
     ```ts
     const x: never = "12"; //(X)
     // never 타입이기때문에 아무값도 할당할 수 없음
     ```
2. unit type(literal type)
   - 한가지 값만 포함하는 타입
     ```ts
     type A = "A";
     type B = "B";
     type Twelve = 12;
     ```
3. | 연산자, union type
   - 세가지 이상의 타입이 묶인 값 집합들의 합집함으로 | 연산자로 연결시켜줌
     ```ts
     type AB = A | B;
     type AB12 = A | B | 12;
     ```
4. & 연산자, 인터섹션

   - 두 타입의 인터섹션, 즉 교집합을 &로 계산 <br>
     아래 코드는 `Person`과 `LifeSpan`이 공통으로 가지는게 없어서 `PersonSpan`이 never type 같지만 인터섹션은 결국 여러타입을 만족하는 하나의 타입을 의미한다. (여러 타입정의를 하나로 합치는 행위)

     ```ts
     interface Person {
       name: string;
     }

     interface LifeSpan {
       birth: Date;
       death: Date;
     }

     type PersonSpan = Person & Lifespan;
     ```

     그래서 아래 코드는 오류가 발생하지 않는다.

     ```ts
     const ps: PersonSpan +
     name: '아무개',
     birth: new Date('1912/06/23'),
     death: new Date('1954/06/07')
     ```

5. union type 주의점

   - 아래 K는 never 타입이다. 왜냐하면 union type에 어떠한 값도 속하지 않기 때문이다.

     ```ts
     type K = keyof (Person | LifeSpan);
     ```

     일반적으로 인터페이스를 합칠려면 extends를 쓰면 되는대 ~ 할당 가능하느 ~ 에 부분집합 이라는 의미로 해석하면 된다.

     ```ts
     interface Peson {
       name: string;
     }

     interface PersonSapn extends Person {
       birth: Date;
       death: Date;
     }
     ```

6. 서브타입
   - 어떤 집합이 다른 집합의 부분집합 이라는 뜻.
     Vector3D는 Vector2D의 서브타입, Vector2D는 Vector1D의 서브타입
     ```
     interface Vector1D {x: number};
     interface Vector2D {x: number, y: number};
     interface Vector3D {x: number, y: number, z: number};
     ```
7. extends 키워드
   - 제네릭 타입의 한정자로도 사용되는대
     ```ts
     function getKey<K extends string>(val: any, key: K);
     ```
     위 코드에서 extends를 집합의 관점에서 해석하면 string의 부분 집합 범위를 가지는 어떠한 타입이 된다. 그러므로 string 리터럴 타입, string 리터럴 타입의 유니온, string 자기 자신 모두 포함된다. 그래서 아래 코드중 마지막 12 를 제외한 모든 코드가 정상동작 하게 된다.
     ```ts
     getKey({}, "x");
     getKey({}, Math.random() < 0.5 ? "a" : "b");
     getKey({}, document.Title);
     getKey({}, 12);
     ```

## item8 : 타입 공간과 값 공간의 심벌 구분하기

## item9 : 타입 단언보다는 타입 선언을 사용하기

## item10 : 객체 래퍼 타입 피하기

## item11 : 잉여 속성 체크의 한계 인지하기
