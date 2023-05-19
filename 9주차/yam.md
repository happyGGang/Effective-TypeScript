# any 다루기
any의 장점은 살리며 단점을 줄이자.  
## item 38 : any 타입은 가능한 좁은 범위에서만 사용하기  
```ts
function processBar(b: Bar) {

}
function f() {
    const x = expressionReturningFoo();
    processBar(x); //error 'Foo' 형식의 인수는 'Bar' 형식의 매개변수에 할당될 수 없습니다.
}
```
x라는 변수가 Foo타입이지만 형태자체만 보면 Bar타입에도 할당이 가능하다면, 오류를 제거하는 방법은 두 가지임  
```ts
function f1() {
    const x: any = expressionReturningFoo();
    processBar(x);
}
function f2() {
    const x = expressionReturningFoo();
    processBar(x as any);
}
```
책에서는 f1보다 f2가 낫다고 말하고 있음.  
x를 any로 박아버린 순간 다른곳에서도 x를사용하고 있다면  
또다른 예외를 불러일으킴  
하지만 f2처럼 processBar를 호출하는 순간에만 any로 취급한다면 예외의 범위가 그만큼 한정됨  
이처럼 any를 사용할때 any가 적용되는 범위를 줄여야 함  

## item 39 : any를 구체적으로 변형해서 사용하기
어쩌피 any를 쓴다고 "any" 적어버리지 말고,  
최대한 성의껏 any를 작성해라  
```ts
let a: any[] = [1,2,3];
```
이렇게하면 사용자가 보기에도 조금 더 타입을 확정지을 수 있고,  
위의 예시에서 a.length를 반환한다 했을시  
배열any타입으로 선언되어있기 때문에 예외가 없다  

## item 40 : 함수 안으로 타입 단언문 감추기
함수를 작성하다 보면,  
외부로 드러난 타입정의는 간단하지만 내루 로직이 복잡해서 안전한 타입으로 구현하기 어려운 경우가 있음  
함수의 모든 부분을 안전한 타입으로 구현하는게 이상적이지만,  
**함수 내부에는 타입 단언을 사용하여 감추고** 함수 외부로 드러나는 타입 정의를 정확히 명시하는 정도로 끝내는게 괜찮음  
나는 타입 단언문 자쳬를 예외라 생각하는데 이 예외를 함수 내부로 진입시켜 예외범위를 줄인다 생각하니 이해됐음.  

## item 41 : any의 진화를 이해하기  
TS에서 일반적으로 변수의 타입은 변수 선언때 결정되지만  
any 타입은 예외인 경우가 존재함  

```ts
function range(start: number, limit: number) {
    const out = []; // any[]로 선언
    for (let i = start; i < limit; i++) {
        out.push(i);
    };
}
return out; // 반환 타입이 number[]로 추론됨
```
왜 반환 타입이 number[]로 추론되었을까?  
any[] > number[] 로 타입 좁히기가 된 것을 타입 진화라고 부르며,  
이는 noImplicitAny 가 설정된 상태에서 변수의 타입이 암시적으로 any일 경우, any[] 타입일때만 가능함 
any를 명시적으로 선언해주면 일어나지 않는 일.  
명시적으로 타입을 선언해서 안전하게 타입을 유지,좁히자.  

## item 42 : 모르는 타입의 값에는 any 대신 unknown 사용하기
any와 unknown의 차이를 안다면 제목만 봐도 이해가 가능함  
any는 아무 값이나 와도 괜찮겠지~ 하고 실행해버린다면  
unknown 타입은 모든 값이 온다고 가정하고 코드를 검증해준다.  
배열의 기본 메서드를 unknown타입이 사용한다고 가정하면,  
만일 unknown타입에 string이나 number의 값이 올 수 있고,  
에러를 일으킬 수 있기 때문에 사용이 불가하다.  
그래서 조금 더 안정적이라고 생각함  

책에서는 추가로 말하기를 예전에 unknown 대신 {}를 사용했다하는데  
null과 undefined가 불가능하다고 판단되는 경우에만 unknown대신 {}를 사용하라 함  
```ts
let aaaa: {};
aaaa=3; // not error
```
다만 {}를 사용하고 3과같은 값을 넣었을때 직관적으로 어울리지 않다고 개인적으로 생각되어  
사용을 기피할 듯 함
## item 43 : 몽키 패치보다는 안전한 타입을 사용하기  
자바스크립트의 유명한 특징 중 하나는,  
객체에 임의의 속성을 추가할 수 있을 만큼 유연하다는 것  
```js
window.a = 3;
```

다만 이 특징을 남발하다보면 부작용시 생길 수 있다.  
타입스크립트의 예시에선.  
돔의 속성에 접근하여 임의로 추가를 해버리면,  
HTMLElement의 타입체커가 에러를 뿜어낼 수 있음  

```ts
document.monkey = 'Tamarin'; // error 'Document' 유형에 'monkey' 속성이 없습니다.
(document as any).monkey = 'Tamarin'; // not error
```

any타입으로 대신하여 해결할 수 있지만,  
그럼 타입스크립트의 의미가 훼손되어짐  

이것의 해결책은 document로부터 데이터를 분리하라 함  
interface를 이용하기  
```ts
interface Document {
    monkey: string;
}
document.monkey = 'Tamarin';
```

## item 44 : 타입 커버리지를 추적하여 타입 안전성 사용하기  
noImplicitAny를 설정 후, 모든 암시적 any 대신  
명시적 타입 구문을 추가해도 any 타입과 관련된 문제들로부터 완벽히 안전할 수 없음  
any타입이 여전히 존재할 수 있는 두 가지 경우가 있음  
 - 명시적 any 타입  
 - 서드파티 타입 선언  

any의 개수를 추적할 수 있는 type-cover-age 패키지 라는 것이 있음  
뭐 이런 도구를 이용해 any 타입을 관리하라 함.  