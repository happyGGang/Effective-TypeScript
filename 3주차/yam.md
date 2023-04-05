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