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





