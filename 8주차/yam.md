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



