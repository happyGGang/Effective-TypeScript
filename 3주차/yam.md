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



## item14 : 타입 연산과 제너릭 사용으로 반복 줄이기