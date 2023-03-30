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
TS에서 변수에 값을 할당하고 타입을 부여하는 방법은 두가지임
```ts
interface Person {name:string}
cosnt alice: Person = {name: 'Alice'}; // 타입 선언
const bob = {name: 'Bob'} as Person;   // 타입 단언
```
이 두가지 방법은 결과가 같아보이지만 다름  
```ts
const alice: Person = {}; //error
const bob = {} as Person
```
타입 선언은 할당되는 값이 인터페이스를 만족하는지 검사하지만  
타입 단언은 이를 무시하고 강제 타입지정해버림  
```ts
interface Person {name:string}
const alice: Person = {
    name:'Alice',
    occupation: 'dasda' // error
}
const bob = {
    name:'bob',
    occupation: 'adasda'
} as Person // not error
```
꼭 필요한 경우가 아니면 타입 단언대신 타입선언을 사용해야 강력하게 체크할 수 있다.  
다만 alice에 리터럴로 값을 부여하는게 아닌,  
변수에 값을 미리 넣어놓고, 해당 변수를 부여하는 것이라면 에러가 나지않음(리터럴이 아니라면 상위집합 허용);  

```ts
const p = {name:'Alice',occupation:'dasda'};
const alice2: Person = p; // not error
```

타입 단언을 사용해야 하는 경우는 아래와 같다.
```ts
document.querySelector('#myBtn').addEventListener('click',e => {
    e.currentTarget // 타입은 EventTarget
    const button = e.currentTarget as HTMLButtonElement;
    buttomn         // 타입은 HTMLButtonElement
})
```
타입스크립트에서는 dom에 접근불가하므로 myBtn이 버튼인지 알지 못함.
하지만 우리는 myBtn이 버튼인지 확신하고있으므로 타입단언(as)를 사용하여 타입을 부여한다. 

## item10 : 객체 래퍼 타입 피하기
10번은 래퍼객체에 대한 설명이 주를 이룸  
타입스크립트에서는 string을 예시로 들었을때,  
string 기본형과 래퍼객체를 별도로 모델링 한다 소개함  

이말인 즉슨, string으로 타입을 설정시 length와 같은 String 래퍼객체의 메서드가 없다 하는 것  
그래서 String와 string을 잘 구분지어서 사용해야 함  

```ts
const f = function (a:String, b:string) {
    b=a; // error (String 형식의 인수는 string형식의 매개변수에 할당될 수 없음)
}
f('ddd')
```
그래서 래퍼객체 대신 string 기본 형식을 이용하라 함  

## item11 : 잉여 속성 체크의 한계 인지하기
item9에서 봤듯이 타입이 명시된 변수에 객체 리터럴를 할당하면,  
해당 타입의 속성이 있는지, 또 그 외의 필요없는 속성이 있는지 검출해냄  

이는 구조적 타이핑 관점에서 봤을때 "되야하는게 아닌가?" 싶었음  
책에서도 그렇게 소개하고 있고,  
실제로 객체 리터럴 대신 객체변수를 대입하면 잘 들어감  
왜 리터럴을 안되고 변수로 넣을땐 잘 들어가는가?  

<!-- 이는 '잉여 속성 체크' 라는 과정이 수행되었기 때문이라 함   -->
이는 타입스크립트는 단순히 런타임에 예외를 던지는 코드에 오류를 찾는것 뿐 아니라,  
의도와 다르게 작성된 부분까지 찾아주려고 하기 때문  
```ts
interface Options {
    title: string;
    darkMode?: boolean;
}
function createWindow(options: Options) {
    if(options.darkMode) {
        setDarkMode();
    }
}
createWindow({
    title: 'Spider Solitaire',
    darkmode: true // 'error' ('darkMode'를 쓰려고 했습니까?)
})
```

여기서 Options는 허용범위가 매우 넓음  
document, HTMLAnchorElement도 title객체를 가지고 있고, darkMode는 필수값이 아니기 떄문  
순수한 구조적 타입 체커는 이런 종류를 찾아내지 못한다 함  

이 때 '잉여속성체크' 라는 것이 있어 필요없는 속성이 추가로 했을때 검사해주는 것인데..  
document, HTMLAnchorElement와 같은 객체변수나 타입단언(as)에서는 '잉여속성체크'가 실행되지않음  

흠.. 잉여속성체크가 뭔지, 일반속성체크(구조적 타입체커)의 차이점은 이해했는데  
document, HTMLAnchorElement를 대입할때 잉여속성체크를 하는 방법은 없나 모르겠음  
