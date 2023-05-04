# 7주차 : 타입추론, 타입 설계

## item 26 : 타입 추론에 문맥이 어떻게 사용되는지 이해하기
```ts
type Language = 'JavaScript' | 'TypeScript' | 'Python';
functuion setLanguage(language: Language) {}
setLanguage('JavaScript'); // success
let language = 'JavaScript';
setLanguage(language) // error string타입불가
```
이젠 위 예시가 당연하게 느껴짐  
setLanguage('JavaScript')는 타입자체가 JavaScript로 고정되어있고
language는 string으로 추론될 것.

더 나아가 튜플 타입에는 이런 유사한 문제도 있음  
```ts
function panTo(where: [number, number])
panTo([10,20]) //success
const loc = [10,20];
panTo(loc) // error [number, number] 타입이 아닌 number[] 타입으로 추론됨  
```

이는 아래와 같이 해결 가능
```ts
1.
const loc = [10,20] as const;
panTo(loc); // error ( as const는 readonly타입이라 [number,number]타입과 맞지 않는다.)

2.
function panTo(where: readonly [number,number]) {}
const loc = [10,20] as const;
panTo(loc); // success
```

콜백활용
```ts
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
    fn(Math.random(), Math.random());
}

callWithRandomNumbers((a,b) => {
    a; // 타입 = number
    b; // 타입 = number
    console.log(a+b);
});
```
callWithRandomNumbers의 타입선언으로 인해 a와 b타입이 number로 추론된다 함.

//아래부터 모르겠음
하지만 콜백함수를 상수로 뽑아내면 noImpolicitAny 오류 발생 
```ts
const fn = (a,b) => {
    console.log(a+b);
}
```
## item 27 : 함수형 기법과 라이브러리로 타입 흐름 유지하기

타입스크립트에서 외부 라이브러리를 사용하면  
타입정보를 파악하면서 사용할 수 있기 때문에 js에서보다 비교적 권장한다.  
추가로 불필요한 타입구문 명시를 줄일 수 있다.  
## item 28 : 유효한 상태만 표현하는 타입 지향하기
