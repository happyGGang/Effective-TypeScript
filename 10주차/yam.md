# 타입 선언과 @types

## item 45 : devDependencies에 typescript와 @types 추가하기  
npm은 세가지 종류의 의존성을 구분해서 관리하며,  
각각의 의존성은 package.sjon 파일내의 별도영역으로 구분된다.  

 - dependencise : 현재 프로젝트를 실행하는데 필수적인 라이브러리  
                  런타임에 사용되는 라이브러리로 구성  
                  프로젝트를 다른 사용자가 설치한다면, dependencies에 들어 있는 라이브러리도 함께 설치될 것  
 - devDependencise : 현재 프로젝트를 개발하고 테스트하는 데 사용되지만, 런타임에는 필요없음  
                     예를 들어 프로젝트에서 사용 중인 테스트 프레임워크
 - peerDependencise : 런타임에 필요하긴 하지만,  의존성을 직접 관리하지 않는 라이브러리들이 포함  
                      예로는 플러그인이 있음  

타입스크립트는 당연히 devDependencise에 포함됨  

## item 46 : 타입 선언과 관련된 세 가지 버전 이해하기  

타입스크립트는 의존성 관리가 귀찮다.  

- 라이브러리의 버전
- 타입 선언(@types)의 버전
- 타입스크립트의 버전  
세가지 버전을 모두 맞춰야 함  
뒷 내용은 버전이 맞지않았을때의 상황 등등을 소개하는듯  

## item 47 : 공개 API에 등장하는 모든 타입을 익스포트하기  
라이브러리 제작자들은 export하기 전, 사용자들을 위해  
타입들을 모두 제대로 명시해 둘 것.  

## item 48 : API 주석에 TSDoc 사용하기  
사용자를 위해 주석을 JSDoc 스타일로 만들기  
```js
// 내용
/** 내용 */
```

jsdoc예시  
```js
/**
 * 인사말을 생성합니다.
 * @param name 설명
 * @param title 설명
 * @returns 설명
 * /
```

tsdoc의 타입정의 예시  
```ts
/** 특정 시간과 장소에서 수행된 측정 */
interface Measurement {
    /** 어디에서 측정되었나? */
    position: Vector3D;
    /** 언제 측정되었나? */
    time: number;
    /** 측정된 운동량 */
    momentum: Vector3D;
}
```
**타입스크립트에서는 타입정보가 이미 보기이때문에 주석에 추가로 쓸 필요없음**  

## item 49 : 콜백에서 this에 대한 타입 제공하기  
자바스크립트의 this는  
let, const 가 렉시컬 스코프인것과 다르게  
다이나믹 스코프임  

그래서 타입스크립트에서 사용할 때 주의해서 사용해야 함  
예제는 일단 대충봐서 이해가 안되니 패스  

## item 50 : 오버로딩 타입보다는 조건부 타입을 사용하기  
```ts
function double(x: number|string): number| string {
  return x + x;  
};
```
위 코드가 틀린것은 아니지만  
string을 넣으면 string이 반환되고,  
number를 넣으면 number를 반환된다는 정보는 어디에도 없다;  

제너릭을 사용하면 위 아쉬운 점을 해결 가능
```ts  
function double<T extends nyumber|string>(x: T): T;
```

이게 거창하다면 그냥 타입을 구분지어 함수를 생성
```ts
function double(x: number): number;
function double(x: string): string;
```

하지만 위 방법도 아쉬운 점이 있음  
원래대로라면 double함수는
string|number가 타입인 인수에 할당가능해야하지만 위처럼 타입을 구분지어 버리면 할당이 불가함  
```ts
function f(x: number|string) {
    return double(x); // error      string|number 형식의 인수는 string형식의 매개변수에 할당불가
}

```

조건부 타입 추가  
```ts
function double<T extends number | string>(
    x: T
): T extends string ? string : number;
function double(x: any) {return x + x;}
function f(x: number|string) {
    return double(x);
}
```




