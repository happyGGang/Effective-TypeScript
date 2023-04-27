# 6주차 : 타입 추론

## item 23 : 한꺼번에 객체 생성하기

객채를 생성할때, 단계를 나누어 생성하면 쓸데없이 복잡해 질 수 있음  
```ts
1.
interface b {x:number; y:number};
const a = {}; // error

2.
interface b {x:number; y:number};
const a = {} as b
a.x=3;
a.y=4;
```
될 수 있으면 한꺼번에 생성하기  

## item 24 : 일관성 있는 별칭 사용하기  

```ts  
interface Coordinate {
    x:number;
    y:number;
}
interface BoundingBox {
    x: [number, number];
    y: [number, number];
}
interface Polygon {
exterior: Coordinate[];
holes: Coordinate[][];
bbox?: BoundingBox;
}

function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
    const box = polygon.bbox;
    if(polygon.bbox) {
        if(pt.x < box.x[0] || pt.x > box.x[1] || pt.y<box.y[0] || pt.y > box.y[1]) {
            return false;
        }                                                                                                           
    }
}
위 예시에서 box.x를 참조할 때 에러가 나는건 이해가 됨
그런데 polygon.bbox.x는 왜 에러가 안나는지 모르겠음
똑같이 Polygon 인터페이스의 bbox는 선택적 항목인데.
```  

## item 25 : 비동기 코드에는 콜백 대신 async 함수 사용하기

비동기 코드 예시  
```js
const page1Promise = fetch(url1);
page1Promise.then(res1 => {
    return fetch(url2);
}).then(res2 =>{
    return fetch(url3);
}).catch(err => {

})

aysnc function fetchPages() {
    const res1 = await fetch(url1);
    const res1 = await fetch(url2);
    const res1 = await fetch(url3);
}
```

가끔 프로미스를 직접 생성해야 할 때 빼고는 async/await를 사용하길 권장  
일반적으로 더 간결/직관적이며  
프로미스를 자동반환하도록 강제됨  


