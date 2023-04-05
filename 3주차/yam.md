# 3주차 학습

## item12 : 함수 표현식에 타입 적용하기  

반복되는 함수 시그니처가 있다면 하나의 타입으로 통합할 수 있음  
```ts
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a,b) => a+b;
const sub: BinaryFn = (a,b) => a-b;
const mul: BinaryFn = (a,b) => a*b;
const div: BinaryFn = (a,b) => a/b;

const add2: number = (a:number, b:number) => a+b; 
```
이처럼 재활용성을 위해 함수표현식 자체에 타입을 부여하는게 아닌,  
함수시그니처 타입을 구현하여 재활용하길 권장하고 있음  

허나 편하다고 남용하면 안되고 같은 주제로 묶여져있을때만 사용하면 좋을 듯  
전달인자의 타입이 number, 함수의 타입이 number인 다른 함수가 있더라도  
BinaryFn이라는 주제와 동떨어진 함수라면 사용하기 꺼려짐  

## item13 : 타입과 인터페이스의 차이점 알기  

인터페이스로 타입을 확장할 수 있고,  
타입으로 인터페이스를 확장가능  

유니온 인터페이스는 작성불가하지만  
타입,인터페이스를 사용하여 유니온 타입을 생성가능  

튜플구조는 T,I 둘 다 가능하지만  
타입으로 작성 권장  
이유는 아직 이해 못함  
보기에도 type이 더 편해보이긴 하다  

인터페이스는 중복된 이름으로 재 선언시  
인터페이스가 확장(+)되지만 type은 error를 반화난다.  


## item14 : 타입 연산과 제너릭 사용으로 반복 줄이기  

**타입 중복을 줄여라**  
책의 예시로는 중복된 인자는 새로 타입을 만들어 재사용하며 타입부여  
혹은 인터페이스의 확장을 이용하여 불필요한 타입생성 막기  

타입확장시  
일부부분만 표현하는 타입과,  
전체부분을 표현하는 타입이 있음  
그래서 일부부분과 전체부분의 중복된 부분을 줄이기 위해  
타입확장을 하려 함  
그런데 일부부분에서 전체부분으로 확장을 하자니 모양새가 이상함  
a에서 a_1, a_2로 확장해야 하는데  
a_1에서 a로 확장하는 느낌?  

```ts
interface State {
    userId: string;
    pageTitle: string;
    recentFiles: string[];
    pageContents: string;
}
interface TopNavState {
    userId: string;
    pageTitle: string;
    recentFiles: string[];
}
```

TopNavState를 타입으로 변환 후, State를 인덱싱하여  
중복제거함  
```ts
type TopNavState = {
    userId: State['userId'];
    pageTitle: State['pageTitle'];
    recentFiles: State['recentFiles'];
}
```

2차 중복제거  
```ts
type TopNavState = {
    [k in 'userId'|'pageTitle'|'recentFiles']: State[k]
}
```

값의 형태가 미리 있고,  
그 값의 형태에 따라 type을 선언하고 싶을때  
```ts
const INIT_OPTONS = {
    width:640,
    height:480
}
type Options = typeof INIT_OPTIONS;
```
타입스크립트에선 typeof가 더 정확히 연산되어 타입을 표현해주기 떄문에 이 같은 방법이 가능  

80장과 82장이후는 모르겠
