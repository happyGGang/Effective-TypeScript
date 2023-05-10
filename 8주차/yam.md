# 8주차 : 타입 설계

## item 29 : 사용할 때는 너그럽게, 생성할 때는 엄격하게  
TCP 구현체의 일반적 원칙은  
나의 작업은 엄격하게 하며, 다른사람의 작업은 너그럽게 수용해야 한다 << 임  
예를 들면 함수 매개변수 타입의 범위는 넓어도 되지만,  
반환할 때는 최대한 구체적이어야 한다.  
(협업, 인수인계시 위 마인드가 도움이 될 듯)  
물론 여러 타입의 매개변수를 처리하느라 코드는 길어진다는 단점이 있긴 함  

예시코드  
```ts
declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;

예제에선 viewportForBounds의 결과를 setCamera로 바로 전달하고 싶은듯

cameraOptions와 LngLat의 타입정의는 아래와 같음  
interface CameraOptions {
    center?: LngLat;
    zoom?: number;
    bearing?: number;
    pitch?: number;
}
type LngLat = 
    {lng: number; lat: number}|
    {lon:number; lat: number;}|
    [number,number];

CameraOptions의 옵션을 변경할때는 일부만 변경할 수 있게  
선택적 옵션을 사용하였으며  
lnglat의 값도 유니온형식으로 정해 setCamera에 CameraOptions의 값을 넣을때 비교적 수월함  

viewportForBounds의 매개변수의 타입인 LngLatBounds도 자유롭게 타입을정함
type LngLatBounds = 
    {northeast: LngLat, southwest: LngLat} |
    {LngLat, LanLat} |
    [number, number, number, number];

뭐 대충 이렇게 범용성있게 허용함으로 가능한 형태가 19가지 이상이랍니다.  
여기에 새 함수를 작성한다 함  

function focusOnFeature(f: Feature) {
    const bounds = calculateBoundingBox(f);
    const camera = viewportForBounds(bounds);
    setCamera(camera);
    const {center: {lat,lng}, zoom} = camera;
    // ... 형식에 'lat' 속성이 없습니다.
    // ... 형식에 'lng' 속성이 없습니다.
    zoom; // 타입이 number | undefinnd
    window.location.search = `${lat}${lng}${zoom}`
}
위 에러가 왜 일어나나면, camera를 생성할때 viewportForBounds에서 너무 자유롭게 반환값이 도출되기때문에 에러가 남
범용성을 늘리기 위해선, 매개변수의 타입을 넓히는 것 뿐만 아니라,  
반환값의 타입을 좁히는것 도 중요하다 가 주제인듯  
```

## item 30 : 문서에 타입 정보를 쓰지 않기  
다음코드에 잘못된 부분이 있다 함
```ts
/**
 * 전경색 문자열을 반환합니다
 * 0개 또는 1개의 매개변수를 받습니다.
 * 매개변수가 없을 때는 표준 전경색을 반환합니다.
 * 매개변수가 있을때는 특정 페이지의 전경색을 반환합니다.
 * 
 */
function getForegroundColor(page?: string) {
    return page === 'login' ? {r: 127, g:127, b:127} : {r:0,g:0,b:0}
}
```
 - 함수가 전경색문자열을 반환하지만 실제론 rgb객체를 반환함  
 - 주석에는 0,1개의 매개변수를 받는다 하지만 굳이 필요없는 설명임  
 - 함수선언보다 주석이 쓸데없이 더 김  

타입스크립트의 타입구문시스템은 간결하며, 쉽게 읽을 수 되어있으므로  
위 같은 간단한 코드에선 주석이 굳이 필요없음

// 애플리 케이션 또는 특정 페이지의 전경색을 가져옵니다.
정도가 적당할 듯  
이렇게 바꾸면 반환값의 타입이 바뀌워도 주석자체를 바꿀 이유가 사라짐  

## item 31 : 타입 주변에 null 값 배치하기  
strictNullChecks 설정을 키면,  
null/undefined 값 관련된 오류들이 갑자기 나타날 수 있음.  
값이 모두 null이거나 null이 아닌경우로 **분명히 구분** 된다면,  
값을 다르기 쉬움.  

아래는 숫자들의 최솟값과 최댓값을 계산하는 extent 함수임  
```ts
function extent(nums: number[]) {
    let min, max;
    for (const num of nums) {
        if (!min) {
            min = num;
            max = num;
        } else {
            min = Math.min(min, num);
            max = Math.max(max, num);
        }
    }
    return [min, max];
}
```  
이 코드에는 아래의 결함이 있음  
 - 최솟값이나 최댓값이 0인 경우, 값이 덧씌워져 버림  
 - nums배열이 비어있다면 함수는 [undefined, undefined]를 반환해버림  

개선안  
```ts
function extent(nums: number[]) {
    let result: [number, number] | null = null;
    for (const num of nums) {
        if (!result) {
            result = [num, num];
        } else {
            result = [Math.min(num,result[0]), Math.max(num, result[1])];
        }
    }
    return [min, max];
}
```
이제 반환타입이 [number,number] | null이 되어서 사용하기가 수월해지며 버그도 수정됨  
이 챕터의 주제를 잘 설명하진 못하겠지만  
타입스크립트랑 크게 관련있는 주제같지 않다고 생각됨  
그냥 단순한 버그 수정으로 보임  

## item 32 : 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기  
유니온 타입의 속성을 가지는 인터페이스를 작성하고 있다면,  
반대로 인터페이스를 유니온 타입으로 적용시키는게 더 낫지 않을까 생각해봐야 함  

```ts
interface Layer {
    layout: Fill | Line | Point;
    paint: Fill | Line | Point;
}
```
위 코드의 잘못된 점은  
애초에 layout타입이 예를들어 fill이면 paint도 같은 fill 타입이 되어야함  
그런데 소스로만 분석하자면 layout과 point가 서로 다른 타입을 가질 수 있게되버림  

굳이 중복되게 생성할 필요없이 아래처럼 생성한다.  
```ts
interface FillLayer {
    layout: Fill;
    paint: Fill;
}
interface LineLayer {
    layout: Line;
    paint: Line;
}
interface PointLayer {
    layout: Point;
    paint: Point;
}
type Layer = FillLayer|LineLayer|PointLayer;  
```  

뒤의 내용들은 굳이 기록하지 않아도 될 내용으로 보여 읽고 넘김  
기억하지 못할까봐 기록함.  

## item 33 : string 타입보다 더 구체적인 타입 사용하기  
string 타입의 범위는 매우 넓음  
아무 글자나 다 받아들이기 때문에,  
가능하다면 더 구체적으로 string 타입을 좁히자.  
위 주제만 봐도 충분히 이해가 가서 예제는 작성안함  

keyof를 이용할때도 위의 주제를 따라 string의 타입을 좁혀 선언했다면 편함.  
```ts
type A = 'a' | 'b' | 'c';
어떠한 객체의 타입을 type A 를 이용하여 정한것이 아닌, 단순 string으로 했다면
typeof 더 잘 사용 불가하다.
```

## item 34 : 부정확한 타입보다는 미완성 타입을 사용하기  
일반적으로 타입을 더 자세히 작성하였을때, 버그를 더 많이 잡을 수 있음  
그러나 타입선언의 정밀도를 높이는 과정에서 실수가 발생하기 쉽고,  
잘못된 타입은 차라리 타입이 없는 것 보다 못할 수 있음(당연)  
결제기능만들다가 잘못만들어서 카드번호,비밀번호등을 뿌려버리는 버그가 있으면 당연히  
결제기능없는게 더 좋음  

위의 예시코드임  
```ts
적을려고 했는데 예시를 읽어보니 딱히 안적어도 될듯.  
정확하게 타입이 모델링되지 않았다면,  
어쩔수 없이 덜 좁게 타입을 사용해야 함  
```

## item 35 : 데이터가 아닌, API와 명세를 보고 타입 만들기  
예시데이터가 아닌 API 명세서를 보고 만들어야 하는건 당연해 보인다.  
예시코드가 필요하진 않을듯  
데이터의 예외적인 부분까지 API 명세서에 작성하는 것이니  
예시 한,두개 제공받는다고 모든 예외를 다 알진 못한다.  

190장의 예시에 coordinates가 있다고 가정한게 문제라는데,  
애초에 예시데이터도 안보여주네요.  
아마 예시데이터엔 coordinates가 있는데  
실제론 있을수도 있고 없을 수도 있겠죠 뭐  

## item 36 : 해당 분야의 용어로 타입 이름 짓기  
데이터를 명확하게 작성하지 않으면 인수자,  
인수자가 아니더라도 미래의 내가 코드파악하는데 어려움이 있을 수 있음  
어떠한 식물의 객체를 만든다 했을 때,  
이름이라는 속성이 있음  
그 속성이 단순 명칭일지, 어떤 언어를 이용하여 표현해야 하는지에 따라 속성명이 name이 아닌 더 자세하게 표현될 수 있음  
적당히 자세하게 기술하여 모든 소스코드 사용자를 편하게 하자  
다만, 약어문서를 만들어 잘 공유한다면 소스코드내에서 자세히 표현하지 않더라도 충분하다고 생각함  

## item 37 : 공식 명칭에는 상표를 붙이기  
벡터같은 예시를 왜드는지 모르겠다 좀 쉽게 주면 안되나?  
나는 문관데?  
사실 귀찮아서 그냥 소스를 안봤음  
```ts
interface Vector2D {
    x: number;
    y: number;
}
function calculateNorm(p: Vector2D) {
    return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({x:3, y:4}); //정상 결과 5
const vec3D = {x:3, y:4, z:1};
calculateNorm(vec3D); //정상! 결과는 동일하게 5
```
이 코드에 버그가 없긴하지만 구조적 타이핑때문에  
3차원벡터의 객체를 calculateNorm에게 넘겨도 잘 작동됨.  
3차원벡터 객체의 허용자체를 없애려면 공식명칭을 사용하라 함  
```ts
interface Vector2D {
    _brand: '2d';
    x:number;
    y:number;
}
function vec2D(x: number, y:number); Vector2D {
    retrun {x, y, _brand: '2d'}
}
function calculateNorm(p: Vector2D) {
    return Math.sqrt(p.x * p.x + p.y * p.y);
}
calculateNorm({x:3, y:4}); //정상 결과 5
const vec3D = {x:3, y:4, z:1};
calculateNorm(vec3D); // error
```
이해 완료  
vec3D를 vec2D를 사용하여 Vector2D로 바꾼다음 사용도 가능하다.  
구조적 타이핑때문에 값 체크가 잘 되지않는다면 타입을 나타내는 속성을 추가하여 제어해보자.  