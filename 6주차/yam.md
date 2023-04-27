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

