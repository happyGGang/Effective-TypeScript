# 2주차 학습 : 타입스크립트의 타입 시스템

## item6 : 편집기를 사용하여 타입 시스템 탐색하기
보통은 타입스크립트 컴파일러를 실행하지만,  
타입스크립트 서버에서는 '언어 서비스' 를 제공한다고 함  

여기서 '언어 서비스' 란  
코드 자동완성, 명세검사, 검색, 리팩터링이 포함됨  
명세검사를 통해 타입스크립트가 어떠한 지점에서 특정 객체의 타입을 무엇으로 추론하는지도 확인 가능하다.  

## item7 : 타입이 값들의 집합이라고 생각하기
1주차 학습에서 메모했던 것처럼 구조적 타이핑의 특징으로  
타입과 값 과의 관계가 **일치** 해야만 하는것이 아니라  
값(객체)이 타입에 포함되는 집합(상위집합)일 때도 허용함  
이 단원에서는 위 내용만 알면 되는 듯  

```ts
const list: number[] = [1,2];
const tuple: [number, number] = list;
// tuple에는 number[] 형식을 부여할 수 없음.  
// 실제론 1,2 두 가지 원소가 있지만 number[] 타입에는 공집합, 혹은 원소의 개수가 하나일 수 있기 때문에 경고를 줌  
```

```ts
const triple: [number, number, number] = [1,2,3];
const double: [number, number] = triple;
// triple을 double에 부여할 수 없음  
// 이는 자바스크립트의 배열은 배열의 모습을 한 객체로써,  
// length 프로퍼티를 지니고 있음  

// double = triple 이 에러가 난 이유는  
// 서로 보유하고 있는 length 프로퍼티의 값이 맞지 않아 에러가 발생한 것  
```

## item8 : 타입 공간과 값 공간의 심벌 구분하기
```ts
interface Cylinder {
    radius: number;
    height: number;
}

const Cylinder = (radius: number, height: number) => ({radius,height})
// 위의 두 Cylinder는 독립적인 타입이며, 값이다.  
// 그래서 서로 아무런 관련도 없음.  
// 위 문법은 Implicit return이라고 부르며 나중에 추가 공부할 예정  

// 아무튼 위 상황일때 Cylinder가 타입,값 두 가지 모두로 쓰일수 있는 특징떄문에 아래와 같은 오류를 야기할 수 있음  
function calculateVolume(shape: unknown) {
    if (shape instanceof Cylinder) {
        shape.radius
        // {} 형식에 'radius' 속성이 없습니다.
    }
}
// instanceof는 js의 런타임 연산때 수행하므로 Cylinder의 타입에 대해 접근할 수 없어서 함수를 참조함  
```

타입스크립트에서 class와 enum은 타입과 값 두가지 가능한 예약어임  
밑의 예제는 타입으로 쓰임  
```ts
class Cylinder {
    radius=1;
    height=1;
}
function calculateVolume(shape:unknown) {
    if(shape instanceof Cylinder) {
        shape        //정상 타입은 Cylinder
        shape.radius //정상 타입은 number
    }
}
```
클래스가 타입으로 쓰일때는 형태가 사용되는 반면, 값으로 쓰일때는 생성자가 사용됨  
그런데 instanceof는 js런타임연산때 수행되는데 타입으로 쓰일수있나? 라고생각했지만  
class는 js때도 유지되니 잘못생각한거였음  

어려워서 8장 패스  

## item9 : 타입 단언보다는 타입 선언을 사용하기

## item10 : 객체 래퍼 타입 피하기

## item11 : 잉여 속성 체크의 한계 인지하기
