# 5주차

## item 19 : 추론 가능한 타입을 사용해 장황한 코드 방지하기  

ts를 처음 접한 개발자가 하는 일은 변수마다 타입을 주는 것임  
하지만 이는 사실 불필요함  
코드의 모든 변수에 타입을 선언하는 것은 비생산적이며  
```ts
let x = 12;
```
정도만 해도 이미 number로 추론되어 있음  

이는 객체와 배열에도 마찬가지이며,  
명시적으로 타입을 넣을 시, 불필요하게 타입체커를 통과하지 못할 수 있다.  
그래서 타입스크립트에서 타입을 추론할 수 있다면, 굳이 타입 구문을 작성하지 않는 게 좋다.  
다만 함수의 변환 타입을 명시해주면,  
 - 함수에 대해 더 명확하게 파악이 가능하고,  
 - 명명된 타입을 사용할 수 있음
명명된 타입을 씀으로써 오는 장점으로는 아래와 같음
```ts
interface Vector2D {x:number; y:number;}
function add(a: Vector2D, b:Vector2D) {
    return { x: a.x + b.x, y: a.y + b.y};
}
```
Vector2D 타입과 같은 값을 add함수에서 반환하지만,  
타입스크립트에서는 Vector2D라고 표현을 하지 않고,  {x: number; y: number;} 타입의 함수라고 결정지어버린다.  
틀렸다고 볼 순 없지만,  
내가 지정한 타입 그대로 타입스크립트에서 알려주는게 더 깔끔해보임  

## item 20 : 다른 타입에는 다른 변수 사용하기  

js에서는 한 변수의 타입을 변경해 가며 사용 가능하지만,  
타입스크립트에서는 초기화 된 변수라면 타입이 이미 지정되어있어  
타입을 변경해가며 사용할 수 없다.  

이 점이 싫다면, 유니온 타입을 사용하여 가능한 타입을 넓게 열어두는 방법이 있지만,  
무분별하게 유니온 타입을 사용시, 타입체커와 사람 모두에게 혼란을 줄 수 있어서  
단일 타입 지정보단 아쉬운 면이 있음  
책에서는 차라리 별도의 변수를 도입하길 권장함  

이로 인해 생기는 장점은  
 - 변수명을 더 구체적으로 지을 수 있음  
 - 타입 구문이 불필요해짐(타입스크립트의 타입추론)
 - 타입이 더 간결해짐
 - let 대신 const로 변수를 생성? << 이건 왜지 타입이 안바뀌고 값만 바꾸려고 할 수도 있는데

## item 21 : 타입 넓히기  

타입스크립트가 작성된 코드를 체크하는 정적인 시점에,  
변수는 가능한 값들의 집합인 타입을 가집니다.  
상수를 사용해서 변수를 초기화할 때 타입을 명시하지 않으면  
타입 체커는 변수의 타입을 결정해야 합니다.  
이 말은 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야 한다는 뜻이며,  
이를 타입스트립트에서는 '넓히기' 라고 부름

```ts
interface Vector3 {x: number; y:number; z:number;}
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
    return vector[axis];
}

let x = 'x';
let vec = {x:10, y:20, z:30};
getComponent(vec,x);
```
위 예시는 실제 코드는 오류 없이 실행되지만 편집기에서는 
x의 타입이 string으로 추론되어 할당할 수 없다함  

아니 'x'|'y'|'z' 이니 string으로 추론되면 되는게 아닌가 생각했지만,  
```ts
getComponent(vec,'x');
getComponent(vec,'a');
```
잘생각해보면 string이 아닌 더 소집합인 'x'|'y'|'z'의 타입만을 받아들이기 때문에  
string으로 타입추론이 날시 경고를 내주는 거였음   

이처럼 타입스크립트는 string, number와 같은 단계로 타입을 추론해주는데,  
이를 방지하기 위해서는 let 대신 const를 이용한다.  
const이용시 값이 변동되지 않기때문에 타입넓히기가 제한된다.  
```ts
const a = 3;
a는 3에서 불변한 값이기 떄문에 타입도 3으로 제한되어버린다.
```

## item 22 : 타입 좁히기  
  
타입 좁히기는 타입 넓히기의 반대를 뜻하며  
가장 대표적인 예시는 null 체크임  
```ts
const el = document.getElementById('foo'); // 타입이 HTMLElement | null
if(el) {
    el // 타입이 HTMLElement
    el.innerHTML = 'party time'.blink();
} else {
    el // 타입이 null
    alert('no element')
}
```
위 예시에서 if문 내에 도달하려면 el이 null이 아니여야 하기때문에  
타입이 HTMLElement로 좁혀진다.  
속성체크로도 타입을 좁힐 수 있음  
```ts
interface A {a: number}
interface B {b: number}
function pickAB{ab: A | B} {
    if('a' in ab) {
        ab // 타입이 A
    } else {
        ab // 타입이 B
    } 
    ab // 타입이 A | B
}
```

타입스크립트는 이처럼 조건문에서 타입좁히기가 매우 쉬운데,  
타입을 섣불리 판단하는 실수를 저지를 수 있어 꼼꼼히 확인해야 함.  
예를 들어  
```ts
const el = document.getElementById('foo') // 타입이 HTMLElement/null
if(typeof el === 'object') {
    el; // 타입이 HTMLElement | null
    //js에서는 typeof null이 object이기때문에 타입좁히기가 수행되지 않았다.
}
```

타입을 좁히는 다른방법  
 - 태그된 유니온 | 구별된 유니온
 타입 좁히기를 수행할 객체의 예상 타입들을 인터페이스들로 정리 후,  
 분기문을 이용하여 타입을 좁힌다.  
 - 사용자 정의 타입 가드  
 ```ts
 function isInputElement(el: HTMLElement): el is HTMLInputElement {
    return 'value' in el;
 }

 function getElementContent(el: HTMLElement) {
    if (isInputElement(el)) {
        el; // 타입이 HTMLInputElement
        return el.value;
    }
    el; // 타입이 HTMLElement
    return el.textContent;
 }
 ```
 타입좁히기용 함수를 만들어 조건문에 사용했음  
 