# 타입 선언과 @types

## item 45 : devDependencies에 typescript와 @types 추가하기

1. 타입스크립트는 런타임시 존재하지 않기 때문에 일반적으로 devDependencies에 속함
2. 타입스크립트를 시스템 레벨로 설치하기 말자

- 팀원들이 같은 버전 설치한다는 보장 없음
- 프로젝트 셋업 시 별도의 단계 추가

3. @types 의존성은 devDependencies에 포함

## item 46 : 타입 선언과 관련된 세 가지 버전 이해하기

1. 타입스크립트는 의존성 문제를 복잡하게 만듬

- 라이브러리의 버전
- 타입 선언(@types)의 버전
- 타입스크립트의 버전

2. 위 세가지 버전이 하나라도 맞지않으면 오류남
3. 그래서 라이브러리를 업데이트 하는 경우 해당 @types도 업데이트 해줘야함.
4. 만약 업데이트 버전이 준비되지 않았다면 보강기법 사용
5. 라이브러리보다 타입 선언 버전이 더 높아면 타입 선언을 다운그레이드하고나 라이브러리 업데이트 해야함.

## item 47 : 공개 API에 등장하는 모든 타입을 익스포트하기

1. 공개 메서드에 등장한 타입은 전부 익스포트하자

## item 48 : API 주석에 TSDoc 사용하기

1. 익스포트된 함수나 클래스 타입에 주석을 달때 JSDoc이나 TSDoc 형태로 작성해야 편집기가 주석 정보 표시해줌
2. @params, @returns 구문과 문서 서식을 위해 마크다운 사용 가능
3. 주석에 타입정보 쓰지말자
4. 주석은 간단하게만 쓰자.

## item 49 : 콜백에서 this에 대한 타입 제공하기

1. 콜백함수에서 this를 사용해야한다면 타입정보를 명시하자.

```ts
//함수의 두 번째 파라미터 fn의 시그니처에서 첫 번째 라파미터 this는 특별하게 처리됩니다.
function addKeyListener(
  el: HTMLElement,
  fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
  el.addEventListener("keydown", (e) => {
    fn.call(el, e); // 정상
    fn(el, e); // 오류, 콜백 함수의 매개별수에 this를 추가하면 this 바인딩이 체크되기 때문에 실수를 방지 할 수 있음.
  });
}
```

## item 50 : 오버로딩 타입보다는 조건부 타입 사용하기

1. 여러 종류의 파라미터를 받는 함수 시그니처를 명세할 때에는 함수 오버로딩 X 조건부 타입을 활용한 함수 시그니처 명세가 더 정교한 타입 추론을 가능

```ts
// 타입 오버로딩
function double(x: number): number;
function double(x: string): string;
function double(x: any) {
  return x + x;
}

const x = double(10);
const y = double("yy");
const z: string | number = "11";

function zz(z: number | string) {
  double(z); // error
}

// 조건부 타입
function double<T extends string | number>(
  x: T
): T extends string ? string : number;
function double(x: any) {
  return x + x;
}

function zz2(z: number | string) {
  double(z); // 정상
}
```

## item 51 : 의존성 분리를 위해 미러타입을 사용하기

## item 52 : 테스팅 타입의 함정에 주의하기
