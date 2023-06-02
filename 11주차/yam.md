# 타입 선언과 @types

## item 53 : 타입스크립트 기능보다는 ECMAScript 기능을 사용하기  
ts 개발 당시 js에 없던 클래스 등의 기능도 포함하여 신규 제작 하였음  
이후, js의 발전과 함께 신규기능이 출시되었는데  
이 기능들이 ts와 호환이 잘 안되는 문제가 있어  
ts팀에서는 호환성을 포기하고 타입기능만을 발전시켰음  

- enum  
```ts
enum Flavor {
    VANILLA = 0,
    CHOCOLATE = 1,
    STRAWBERRY = 2,
}
let flavor = Flavor.CHOCOLATE;
Flaver // VANILLA, CHOCOLATE, STRAWBERRY
Flaver[0] // VANILLA
```

Flavor 를 매개변수로 받는 함수를 가정  
```ts 
function scoop(flavor: Flavor) {}
```
```js
scoop('vanilla'); // 자바스립트에서 호출
```
```ts
scoop('vanilla');// 'vanilla' 형식은 'flavor' 형식의 매개변수에 할당될 수 없습니다.

import {Flavor} from 'ice-cream';
scoop(Flaovr.VANILLA); //정상
```

이처럼 js와 ts간의 동작이 다름  

- 매개변수 속성  
일반적으로 클래스를 초기화할때 속성을 할당하기 위해 생성자의 매개변수를 사용함  
```ts
class Person {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
}
```
타입스크립트는 더 간결만 문법을 제공함  
```ts
class Person {
    constructor(public name: string) {}
}
```

위 두 문법을 섞어서 사용할 시 클래스 설계가 혼란스러워질 수 있고,  
ts문법은 보통 런타임시에 코드가 줄어들지만 매개변수 문법은 코드가 늘어난다는 특징이 있음  

문제점에대한 예  
```ts
class Person {
    first:string;
    last:string;
    constructor(public name: string) {
        [this.first, this.last] = name.split('');
    }
}
```
Person 클래스에는 세가지 속성이 있지만  
name만 따로 분리되어 있어 통일성이 없어 보임  
클래스에 매개변수속성만 존재한다면 클래스 대신 인터페이스로 만들고 객체 리터럴을 사용하는 것이 좋음  

책에서 강조하기로는,  
매개변수 문법을 사용하든 안하든 통일성을 주고  
코드양이 줄어들지만 다른 자바스크립트의 패턴들과 이질적인것을 인지하여  
선택하여 작성하길 권유함  

- 네임스페이스와 트리플 슬래시 임포트  
ES2015 이전에는 JS의 공식적인 모듈시스템이 없었음  
그래서 여러환경마다의 모듈시스템이 존재했고,  
타입스크립트는 
namespace foo
/// <reference path="other.ts"/>
대충 이런게 있음  
그냥 이거 이제 쓸 필요 없단소리  

- 데코레이터  
데코레이터는 클래스, 메서드, 속성에  
애너테이션을 붙이거나 기능을 추가하는 데 사용함  
다만 현재까지도 표준화가 안료되지 않아서,  
사용중인 데코레이터가 비표준으로 바뀌거나 호환성이 깨질 가능성이 있음  

## item 54 : 객체를 순회하는 노하우  
밑 예제는 정상적으로 실행하지만 편집기에서는 오류가 발생함  
```ts
const obj = {
    one: 'a',
    two: 'b',
    thre: 'c',
};
for (const k in obj) {
    const v = obj[k] // obj에 인덱스 시그니처가 없기 때문에 엘리먼트는 암시적으로 'any' 타입입니다.
}
```
이 오류는 k의 타입은 string인 반면, obj 객체에는 one,two,three 세 개의 키만 존재하기 때문에  
k와 obj의 타입이 서로 다르게 추론되어 오류를 발생시키는 것.  
k의 타입을 구체화 시키면 됨
```ts  
let k: keyof typeof obj; // "one"|"two"|"three" 타입
for (k in obj) {
    const v = obj[k] // 정상
}
```  
이 처럼 keyof typeof 를 이용하여 객체를 순회할 수 있고,  
Object.entries 도 직관적이진 않지만 하나의 방법임  

## item 56 : 정보를 감추는 목적으로 private 사용하지 않기  
자바스크립트는 클래스에 비공개 속성을 만들 수 없고,  
많이들 "_" 를 접두사로 붙이곤 했음  
```js
class Foo {
    _private = 'secret123';
}
```
다만 실제로 비공개 되진 않음
```js
const f = new Foo();
f._private // 'secret123';
```

타입스크립트에는 private, protected 등 접근 제어자가 있지만  
컴파일 후에는 사라지게 된다.  
즉 정보를 감추기 위해 private를 쓰면 안됨  
대신 클로저를 이용함  
