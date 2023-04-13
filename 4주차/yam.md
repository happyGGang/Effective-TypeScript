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

## item 16 : number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

JS에서 객체는 보통 키/값의 쌍 구조이며 키는 문자열이다  
그러므로 배열에서도 숫자인 STRING을 키로 가진다  
{"0":0,"1",1,"2",2}  

타입스크립트는 이러한 혼란을 잡기위해 숫자 키를 허용함  
다만 책 뒷 부분에서는, 이 숫자 키는 실제로 동작하지 않으며  
ArrayLike와 같은 다른 방법을 사용하라 함  

여기서도 런타임 시와, 빌드시에 차이점으로 인한 문제를 말하는 듯한데..  
이 부분은 내가 타입스크립트를 자주 써봐야 감이 잡힐 듯 하다  

## item 17 : 변경 관련된 오류 방지를 위해 readonly 사용하기

삼각수(1,1+2,1+2+3,...)를 출력하는 코드임  
```ts
function print삼각수(n: number) {
    const nums = [];
    for (let i = 0; i < n; i++>) {
        nums.push(i);
        console.log(arraySum(nums));
    }
}
function arraySum(arr: number[]) {
    let sum = 0, num;
    while((num= arr.pop()) !== undefined) {
        sum += num;
    }
    return sum;
}
print삼각수(5)
```

위 코드를 실행하면 예상과 달리  
0,1,2,3,4 순서대로 나온다 함  
arraySum이 nums를 변경하지 않는다고 간주해서 문제가 발생했다고 하는데...  
왜 for문 안에서 배열이 빈배열로 바뀌는지 모르겠음  
>>아 pop이 있네  
이 에러를 체크하려면 arraySum에 readonly 접근제어자를 사용하면 됨  
이로 인해 새로 배열을 쓰거나 pop 등의 메서드를 호출할 수 없게 됐음  

요소를 지우지 않을 것이라면 readonly로 방지하자

## item 18 : 매핑된 타입을 사용하여 값을 동기화하기

솔직히 문법을 잘 모르겠어서 이해 못함  




