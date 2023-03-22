### 01 타입스크립트 알아보기

타입스크립트는 자바스크립트의 상위집합으로써,  
자바스크립트에 더해  
인터페이스, 타입 부여 등을 통해 코드의 예상 범위를 줄여 에러를 방지함  

### 02 타입스크립트 설정 이해하기  

 - noImplicitAny  
타입이 설정되지 않았을 때 암시적으로 any 타입을 부여함

 - strictNullChecks  
모든 타입에서 null과 undefined를 허용하지 않음  

이외에도 공식문서 등 다른 자료들을 확인하여 옵션을 설정  
### 03 코드 생성과 타입이 관계없음을 이해하기  

 - 런타임에는 타입 체크가 불가능함  
```ts
interface Square {
    width: number;
}
interface Rectangle extends Square {
    height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
    if (shape instanceof Rectangle) { 
        // instanceof는 런타임시에 실행되기 때문에 Rectangle이 아무런 역할을 하지 못함
        return shape.width * shape.height;
    } else {
        return shape.width* shape.width
    }
}
```

 - 타입 연산은 런타임에 영향을 주지 않음  
```ts
function asNumber(val: number | string): number {
    return val as number;
}
```
위 코드를 js로 컴파일시에
```js
function asNumber(val) {
    return val;
}
```
이렇게 변해버림  

as 는 타입 연산이므로 런타임과정에서 아무런 영항을 주지 못하게 됨  
이는 자바스크립트의 연산을 통해 변환을 수행해야 한다  

 - 런타임 타입은 선언된 타입과 다를 수 있음  
```ts
function setLightSwitch(value: boolean) {
    switch (value) {
        case true:
            console.log(1);
            break;
        case false:
            console.log(2);
            break;
        default:
            console.log('실행되지않을까 걱정');
    }
}
```
컴파일 과정에서 setLightSwitch 의 인자에 boolean 값이 아닌 다른것을 전송했다면  
에러를 검출해주거나 default문을 실행하지 않을 것이라 생각했으나,  
아닌 경우도 있음  
ex) 서버 호출 후 받아온 값을 인자로 보냈을 시  
ex)
```ts
let a:any = 3;
function setLightSwitch(value: boolean) {
    switch (value) {
        case true:
            console.log(1);
            break;
        case false:
            console.log(2);
            break;
        default:
            console.log('실행되지않을까 걱정');
    }
}
setLightSwitch(a)
```
any타입을 보내면 boolean일 수도 있다고 생각을 해 그냥 컴파일을 해줌  
설정값에 따라 다를 수 있지만  
타입스크립트의 컨셉은  
실제 값과 타입의 비교가 아닌  
타입과 타입의 비교인듯  

위 코드로 풀어말하자면  
a의 값은 숫자 타입이라 setLightSwitch의 인자로 사용시 에러가 날 것 같지만  
실제로는 a의 값인 숫자에 주목하는게 아닌 부여받은 타입인 any에 관심이 치중되어 있어  
any > boolean의 개념이 적용된 듯  





