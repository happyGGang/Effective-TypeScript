# 1주차 학습

## item1 : 타입스크립트와 자바스크립트의 관계 이해하기

1. 타입스크립트는 자비스크립트의 **superset**
   - 모든 자바스크립트 프로그램이 타입스크립트 O, 반대는 X <br>
     아래코드 처럼 `:string` 과 같은 타입구문을 사용하는 순간부터 자바스크립 트는 타입스크립트 영역으로 들어감
     `typescript
function greet(who: string) {
    console.log('hello', who)
}
`
2. 타입스크립트는 런타임 오류를 발생시키는 코드를 찾아냄. but 100%는 아님

   - 가령 아래와 같은 코드가 있으면

     ```typescript
     let city = "new yorl city";
     console.log(city.toUppercase());
     ```

     타입구문을 따로 적지 않았지만 타입체커가 초깃값으로 부터 **타입을 추론**하고

     > 님 toUppercase 속성은 string 형식에 없는데 ㄹㅇ 쓸꺼임?

     라고 묻는다. 쏘 유용하다.

   - 또한 오류가 발생하지는 않지만 우리의 의도와 다르게 동작하는 코드가 있다면 그것 또한 찾아내 준다. 아래와 같은 코드에서

     ```typescript
     const states = [
       { name: "윤경", home: "왜관" },
       { name: "민우", home: "대구" },
     ];

     for (const state of states) {
       console.log(state.hame);
     }
     ```

     우리는 분명 `hame`을 의도하지 않았다. 하지만 오류는 발생하지 않고 `undefined`를 콘솔로 뱉낼거임. 이럴때 타입체커는

     > 님 hame은 없어,,home 맞지? 그거 쓸래?

     하고 오류도 잡아줄 뿐만 아니라 적절한 해결책까지 제시해준다.

   - 이렇게 타입스크립트는 **정적 타입시스템**이기 때문에 런타임에 오류를 발생시킬 코드를 미리 찾아낸다. 그리고 위의 예시 코드처럼 타입 구문을 추가하지 않아도 어느정도 오류를 찾아낼 수 있다. 그러나 여기서 주의할 것은 말그대로 어느정도의 오류를 찾아내는거지 **100% 모든 오류를 찾아내지 못한다.** 예시를 보자.

     ```typescript
     const states = [
       { name: "윤경", hame: "왜관" },
       { name: "민우", hame: "대구" },
     ];

     for (const state of states) {
       console.log(state.home);
     }
     ```

     위 코드에선 타입체커가 `hame`이 오타인지, `home`이 오타인지 구분하지 못한다. 오류의 원인은 알아내겠지만 오타를 구분할 수 없기때문에 제대로된 해결책을 제시해줄 수 없다.
     그래서 최대한 많은 오류를 발견해내기 위해서는 타입 구문을 추가해 확률을 높혀줘야한다.

     ```typescript
        interface State = [
         name: string,
         home: string
        ]

         const states: State[] = [
           {name: '윤경', hame: '왜관'},
           {name: '민우', hame: '대구'}
         ]

         for (const state of states) {
           console.log(state.home)
         }
     ```

## item2 : 타입스크립트 설정 이해하기

1.  타입스크립트 설정 방법

    - 타입스크립트 설정 방법에는 2가지가 있음. <br>

      1. 하나는 커맨드라인을 사용하는 방법
      2. tsconfig.json 사용하는 방법(설정 파일)

      이 두가지 중 가급적 설정파일을 사용하는게 더 좋다. (협업할때 좋음)

2.  꼭 알아야할 타입스크립트 설정 2가지 - noImplicitAny <br>
    변수들이 미리 정의된 타입을 가져야하는지에 여부에 대한 설정이다. noImplicitAny 해제하면 아래 코드처럼 암시적 any 타입으로 간주될 수 있다. 이때 any 타입은 유용하지만 **타입쉴드를 벗기는 행위**이기 때문에 조심해야한다. 그러므로 의도적으로 `: any` 라고 선언해주거나 noImplicitAny를 설정해 분명한 타입을 사용하는 것이 바람직하다. - strictNullChecks <br>
    null과 undefined가 모든 타입에 허용되는지 확인하는 설정이다.<br>
    null과 undefined에 관련된 오류를 찾아내는것에 도움이 많이 되지만 만약 해당 오류가 발생했을지 null 또는 undefined에rk 어디서부터 왔는지 찾아야해 코드 작성이 힘들어진다.

        -  번외로 이 모든 체크를 설정하려면 strict 모드를 사용하면 된다.

## item3 : 코드 생성과 타입이 관계없음을 이해하기

## item4 : 구조적 타이핑에 익숙해지기

## item5 : any 타입 지양하기
