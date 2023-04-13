# 4주차

## item 15 : 동적 데이터에 인덱스 시그니처 사용하기

js의 객체는 문자열 키를 타입의 값에 관계없이 매핑함  
타입스트립트에서는 '인덱스 시그니처' 라는걸 명시하여 유연하게 매핑할 수 있다 함  
```ts
type Rocket = {[property: string]: string};
const rocket: Rocket = {
    name: 'Falcon 9',
    variant: 'v1.0',
    thrust: '4,940 kN',
}; //정상
```
여기서 [property: string]: string이 '인덱스 시그니처' 이며  
당므 세자기 의미를 담고 있음  
 - 키의 이름 : 키의 위치만 표시하는 용도, 타입체커에서는 사용하지 않는 무시할 수 있는 참고정보  
 - 키의 타입 : string이나 number 또는 symbol의 조합이어야 하지만, 보통은 sring을 사용함  
 - 값의 타입 : 어떤 것이든 될 수 있음  

그런데 이렇게 타입체크를 수행하면 단점이 있다고 책에서 소개함  
나는 이 패턴이 이 책에서 너무 싫음  
질질끌리는 기분이 들어서 짜증이 난다  

아무튼 단점으로는  
 - 모든 키를 허용함(잘못된 키 까지)  
 - 특정 키가 필요하지 않음({}도 유효한 Rocket 타입임)  
 - 키마다 다른 타입을 가질 수 없음  
등 이 있음  

?? 인덱스 시그니처는 부정확하니 다른걸 찾아야 한대요  
책의 주제가 인덱스 시그니처인데?  
번역이 살짝 이상하게 됐나 봄  
짜증나네요  

다음 예시로는 인터페이스가 나왔음  
```ts
interface Rocket {
    name: string;
    variant: string;
    thrust_kN: number;
}
const falconHeavy: Rocket = {
    name: 'Falcon Heavy',
    variant: 'v1',
    thrust_kN: 15_200
};
```  

두둥 다음 문단에서 인덱스 시그니처가 나옴  
위 같은 예시에서 쓰지 말고,  
다른상황에 쓰라 함  
예는 CSV 파일처럼 헤더 행(row)에 열(column) 이름이 있고,  
데이터 행을 열 이름과 값으로 매핑하는 객체로 나타내고 싶은 경우입니다.  
```ts
function parseCSV(input: string): {[columnName: strin]: string}[] {
    const lines = input.split('\n');
    const [header, ...rows] = lines;
    const headerColumns = header.split(',');
    return rows.map(rowStr => {
        const row: {[columnName: string]: string} = {};
        rowStr.split(',').forEach((cell, i) => {
            row[headerColumns[i]] = cell;
        });
        return row;
    });
}
```
위 예시에어 열 이름이 무엇인지 미리 알 방법이 없음.  
이럴때 인덱스 시그니처를 사용하라 함  

뭐 이 파트의 주제는 값을 제한하기 어려운,  
유연하게 매핑해야 하는 곳에서는 '인덱스 시그니쳐' 를 생각해 보라~~ 입니다.  
