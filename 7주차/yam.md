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

타입스크립트에서는 타입만 잘 설계해도  
코드를 직관적으로 작성할 수 있다.  
```ts
interface State {
    pageText: string;
    isLoading: boolean;
    error?: string;
}

//페이지를 그리는 renderPage 함수
function renderPage(state: State) {
    if (state.error) {
        return `error ${currentPage}: ${state.error}`
    } else if(state.isLoading) {
        return `Loading ${currentPage}`;
    }
    return `<h1>${currentPage}</h1>${state.pageText}`;
}
```
책에서 말하기를 isLoading과 error가 동시에 있다면  
에러인지, 로딩중인지 알 수 없다 함.  

```ts
//페이지를 전환하는 changePage 함수
async function changePage(state: State, newPage: string) {
    state.isLoading = true;
    try {
        const res = await fetch (getUrlForPage(newPage));
        if (!res.ok) {
            throw new Error(`Unable to load ${newPage}: ${res.statusText}`);
        }
        const text = await res.text();
        state.isLoading = false;
        state.pageText = text;
    } catch (e) {
        state.error = '' + e;
    }
}
```
위 함수는 오류가 났을때 isLoading을 초기화 해주지 않고,  
state.error를 초기화하지 않아 로딩메시지대신 과거의 오류 메시지를 보여준다??  
보여주는 코드가 어디있지 >> renderPage  

이 문제들은 바로 state의 두 가지 속성이 정보가 부족하거나, 두 가지 속성이 충돌해서 일어난 일임  
State 타입은 isLoading이 true이면서 동시에 error 값이 설정되는 무효한 상태를 허용해버림  

```ts
interface RequestPending {
    state: 'pending';
}
interface RequestError {
    state: 'error';
    error: string;
}
interface RequestSuccess {
    state: 'ok;
    pageText: string;
}
type RequestState = RequestPending | RequestError | RequiestSuccess;

interface State {
    currentPage: string;
    requests: {[page: string]: RequestState}
}
```
무효한 생타를 허용하지 않도록 개선한 코드임  

```ts
function renderPage(state: State) {
    const {currentPage} = state;
    const requestState = state.requests[currentPage];
    switch (requestState.state) {
        case 'pending':
            return `Loading ${currentPage}`;
        case 'error':
                return `${currentPage}: ${requestState.error}`;
            case 'ok':
                    return `${currentPage}${requestState.pageText}`
    }
}
//isLoading과 error가 동시에 있는경우를 배제해버림
```