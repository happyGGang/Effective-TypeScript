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
     위 코드에서 extends를 집합의 관점에서 해석하면 string의 부분 집합 범위를 가지는 어떠한 타입이 된다. 그러므로 string 리터럴 타입, string 리터럴 타입의 유니온, string 자기 자신 모두 포함된다. 그래서 아래 코드중 마지막 12를 제외한 모든 코드가 정상동작 하게 된다.
     ```ts
     getKey({}, "x");
     getKey({}, Math.random() < 0.5 ? "a" : "b");
     getKey({}, document.Title);
     getKey({}, 12);
     ```

## item8 : 타입 공간과 값 공간의 심벌 구분하기

1. 타입스크립트 심벌은 타입 공간이나 값 공간 중 한곳에 존재

   - 심벌은 이름이 같더라도 속하는 공간에 따라 다른 것을 나타냄.

     ```ts
     interface Cylinder {
       // 타입으로 쓰임
       radius: number;
       height: number;
     }

     const Cylinder = (radius: number, height: number) => ({ radius, height }); // 값으로 쓰임
     ```

     대신 `interface Cylinder`와 `Cylinder`는 서로 아무 관련도 없지만 이름이 같아서 가끔 오류가 발생함

     ```ts
     function calculateVolume(shape: unKnown) {
       if (shape instanceof Cylinder) {
         shape.radius;
       }
     }
     ```

     위 코드에서 instanceof를 이용해서 shape가 Cylinder 타입인지 확인하려고 했지만 런타임때는 인터페이스는 사라지기때문에 interface Cylinder가 아닌 calculateVolume함수를 참조하게 되어 오류가 발생함

2. 타입인지 값인지는 문맥을 통해 알아내야함
   - 아래 코드처럼 type이나 interface 뒤에 오는 심벌은 타입으로, const나 let 뒤에 오는 심벌은 값으로 사용되는것이 명확하다. 또는 : 뒤에는 타입이, = 뒤에는 값이 오는 것도 명확하다,
     ```ts
     type T1 = "string literal";
     const v1 = "string literal";
     ```
     그러나 class나 enum은 타입도 값도 모두 가능한 예약어라 문맥을 통해 값인지 타입인지 파악해야한다.
3. 값과 타입에서 각각 다르게 동작하는 typeof 연산자
   - 타입으로 사용될때는 값을 읽고 타입을 리턴, 타입 공간에서는 보다 큰 타입의 일부분으로 사용, type 구문으로 이름을 붙이는 용도로 사용
   - 값 관점에서는 값 공간의 typeof가 대상 심벌의 런타임 타입의 문자열을 반환
4. ## class 에서 typeof

## item9 : 타입 단언보다는 타입 선언을 사용하기

1. 타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법
   - 타입 선언 ( : )
   - 타입 단언 ( ~ )
2. 타입 선언
   - 그 값이 선언된 타입임을 명시
3. 타입 단언
   - 타입스크립트가 추론한 타입이 있더라도 단언된 타입으로 간주
4. 타입 단언이 꼭 필요한 경우가 아니라면 타입 선언 사용
   - 아래처럼 오류가 발생할 수 있기때문에 타입 단언보단 타입 선언을 사용하자
     ```ts
     interface Person {
       name: string;
     }
     const alice: Person = {};
     // Person 유형에 필요한 name이 {}에 없음!!!
     const bob = {} as Person;
     // 정상 동작
     ```
5. 화살표 함수의 타입 선언

   - 원하는 타입을 명시하고 할당문의 유효성을 검사하도록 선언

     ```ts
     interface Person {
       name: string;
     }

     const people: Person[] = ["alice", "bob", "jan"].map(
       (name): Person => ({ name })
     );
     ```

6. 타입스크립트보다 타입 정보를 더 잘 알때느 타입 단언문과 null 아님 단언문 사용
   - 타입스크립트는 DOM 엘리먼트에 접근할 수 없다. 그래서 currentTarget이 버튼인지도 알수없음. 이럴때 우리는 타입 단언문을 써야 한다.
     ```ts
     document.querySeletor("#myButton").addEventListener("click", (e) => {
       e.currentTarget;
       const button = e.currentTarget as HTMLButtonElement;
       button;
     });
     ```
   - 변수에 접두사로 쓰이는 !를 사용해서 null이 아님을 단언할 수 있음. 대신 null이 아니라고 확신할때만 사용해야함.
     ```ts
     const elNull = document.getElementById("foo");
     // HTMLElement | null
     const el = document.getElementById("foo")!;
     // HTMLElement
     ```
   - unKnown은 모든 타입의 서브타입이기 떄문에 임의의 타입 간에 변환을 가능하게 하지만 위험한 동작이기 때문에 자제하자!

## item10 : 객체 래퍼 타입 피하기

1. 기본형과 객체형
   - string 기본형에는 메서드가 없기 때문에 아래와 같은 코드를 동작시키려면 기본형을 String 객체형으로 래핑하고 > 메서드 호출 > 래핑객체를 버리는 순으로 동작하는데 여기서 기본형 string은 객체형 String을 받을 수 X, 그렇기 때문에 객체형을 직접적으로 사용하거나 인스턴스를 생성하는것은 피해야한다.

## item11 : 잉여 속성 체크의 한계 인지하기

1. 타입스크립의 잉여 속성 체크

   - 타입이 명시된 변수에 객체 리터럴을 할당할때 타입스크립트는 해당 타입의 속성이 있는지, 그 외에 속성이 없는지 확인한다.

     ```ts
     interface Room {
       numDoors: number
       ceilingHeightFt: number
     }

     const r: Room {
       numDoors: 1,
       ceilingHeightFt: 10,
       elephant: 'present'
     }
     // Room에 elephant 없삼!!
     ```

     구조적 관점에서는 오류가 나지 않아야하지만, 구조적 타입 시스템에서 발생 할 수 있는 오류를 잡을 수 있도록 잉여 속성 체크라는 과정이 수행되서 오류가 발생했다.
2. 잉여속성체크와 할당가능검사는 별도의 과정!
    - 아래 코드를 보면 첫번째 코드에서는 잉여속성이 체크되었지만 두번째 코드에선 잉여속성 체크가 되지 않았다. 잉여속성체크와 할당가능검사는 별도의 과정이라 두 코드 전부 할당가능검사에서는 통과가 된 상태고 아래 코드는 객체리터럴을 할당하지 않아서 잉여속성체크가 일어나지 않은거임
      ```ts
        const obj {
          numDoors: 1,
          ceilingHeightFt: 10,
          elephant: 'present'
        }
        
        const r:Room = obj
        // 정상
        ```
3. 엄격한 객체 리터럴 체크 
    - 잉여속성체크는 말그대로 필요한 속성 이외의 속성까지 체크하기 때문에 엄격한 객체 리터럴 체크라고 표현한다.
4. 잉여속성체크를 피하는 방법
    - 타입 단언은 잉여속성체크를 무시하게 만든다.
        ```ts
        const obj {
          numDoors: 1,
          ceilingHeightFt: 10,
          elephant: 'present'
        } as Room

        // 통과
        ```     
    - *인덱스 시그니처를 사용하면 잉여속성을 피할 수 있음. 
      > 인덱스 시그니처는 객체가 <key, vlaue>형식이며 key와 value의 타입을 정확하게 명시해야하는 경우 사용할 수 있는대, 객체의 특정 value에 접근할 때 그 value의 key를 문자열로 인덱싱해 참조할 수 있다.
      ```ts
      const dog = {
        breed: "retriever",
        name: "elice",
        bark: () => console.log("woof woof"),
      };
 
      dog["breed"]; 
      // value의 key인 breed 문자열로 객체의 value 접근 => 'retriever'
      ```
    - 아래 코드들 처럼 없는 잉여속성이 있을때도 인덱스 시그니처를 사용하면 잉여속성 체크를 피할 수 있는대 두번째 코드처럼 타입을 선언해준 객체가 빈 객체일때 인덱스 시그니처로만 이루어져있다면 오류를 발생시키지 않는 문제가 생길 수 있다.
      ```ts
        interface Options {
          darkMode: boolean
          [otherOptions: string] : unKnown;
        }

        const o: Oprions = {darkmode: true}
        // 정상 동작
        ```

        ```ts
        type ArrStr = {
          [key: string]: string | number;
          [index: number]: string;
        };
    
        const a: ArrStr = {}; 
    
        a["str"]; // 오류 x
        ```
5. 약한 타입
    - 선택적 속성만 가지는 약한 타입에서는 잉여속성체크와 비슷한 체크가 동작하게 되는데 
      ```ts
      interface LineChartOptions {
        logscale?: boolean;
        invertedYAxis?: boolean;
        areaChart?: boolean;
      }

      const opts = { logScale: true };
      const o: LineChartOptions = opts;
      // 오류
      ```
      위 코드는 구조적 관점에서는 모든 속성이 일단 선택적이기 때문에 모든 객체를 포함할 수 있을 것 같지만 저렇게 약한 타입일땐 타입스크립트가 별도의 공통 속성체크라는 검사를 한다. 즉 값 타입과 선언 타입에 공통된 속성이 있는지 체크를 수행하게 되어 위 코드에서 오류를 발생시킨다. 잉여 속성 체크와 같이 오타 검사에도 탁월하지만 잉여속성체크는 객체리터럴 할당에만 동작하는 반면 공통 속성체크는 약한 타입과 관련된 할당문 마다 모두 수행한다.