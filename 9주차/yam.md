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

## item 41 : any의 진화를 이해하기

## item 42 : 모르는 타입의 값에는 any 대신 unknown 사용하기

## item 43 : 몽키 패치보다는 안전한 타입을 사용하기

## item 44 : 타입 커버리지를 추적하여 타입 안전성 사용하기