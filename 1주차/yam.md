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
이는 자바스크립트의 연산을 통해 변환을 수행해야 한다.  

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

### 04 구조적 타이핑에 익숙해지기  

아래 코드에서 calculateLength() 를 호출할때  
Vector2D 타입이아닌 NamedVector 타입의 변수로 호출하였지만 동작이 잘 됨  
이는 Vector2D가 NamedVector의 하위집합이므로 가능  
```ts
interface Vector2D {
    x: number,
    y: number,
}

function calculateLength(v: Vector2D) {
    return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NamedVector {
    name: string;
    x:number;
    y:number;
}

const v:NamedVector = {x:3, y:4, name: 'Zee'};
calculateLength(v);
```
이처럼 어떠한 (A)타입을 사용하도록 설계된 함수지만  
상위집합인 다른 (B)타입을 사용가능한 이유는
타입스크립트에서는 타입의 이름까지 체크하는 것이 아닌, 타입만을 체크하며
집합관계를 허용하는 구조적 타이핑을 사용하기 때문

명목적 타이핑에서는 타입들의 구조가 같더라도 사용하기로한 타입만을 사용해야 되는것으로 알고 있음

다만 이 때문에 문제가 발생할 때도 있음
```ts
interface Vector3D {
    x: number;
    y: number;
    z: number;
}

function normalize(v: Vector3D) {
    const length = calculateLength(v);
    return {
        x: v.x / length,
        y: v.y / length,
        z: v.z / length,
    }
}

console.log(normalize({x:3,y:4,z:5})) // 0.6, 0.8, 1.0
```
사실 이 부분은 뭐가 문젠지 모르겠음  
벡터를 몰라서 책의 내용이 이해가 안감   
0.6 0.8 1.0이 나오는게 정상처럼 보임  
그냥 다음으로 패스  

결론은 상위집합에 속하는 에러를 범할 수 있으니 주의하자 인 듯  

다음예제에서도 이와 같은 에러 예시를 보여줌  
```ts
function calculateLengthL1(v: Vector3D) {
    for (const axis of Object.keys(v)) {
        const coord = v[axis];
        length += Math.abs(coord);
    }
    return length;
}
calculateLengthL1({x:3,y:4,z:5})
```
이라고 했으나 컴파일이 잘됨  
내가 멍청한건지 타입스크립트가 패치된건지 도저히 모르겠음  
책에서 말해주는 에러 이유로는 calculateLengthL1의 인자로 Vector3D타입의 상위타입이 주어질 시,  
프로퍼티의 값이 숫자가 아닌 any값이 된다하여 에러를 뿜는다 함  
책만봤을땐 이해가 갔는데 아무튼 에러는 안났음  

### 05 any 타입 지양하기  

any를 쓸때는 조심하자는 주제임  
타입스크립트의 메인 기능인 타입부여로 인해 오는 장점이 사라짐  
에러 대비도 덜되고, 파악하기도 어려움  