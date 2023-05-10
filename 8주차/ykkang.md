# 타입 설계

## item 29: 사용할 때는 너그럽게, 생성할 때는 엄격하게

1. 함수 시그니처 구현시 매개변수는 타입 범위가 넓게, 반환값은 타입 범위를 구체적으로 만들어야한다.

   ```ts
   declare function setCamera(camera: CameraOptions): void;
   // 매개변수 타입의 범위를 넓게 해야한다.
   declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;
   // 매개변수 타입의 범위를 넓게 해야한다.

   interface CameraOptions {
     center?: LngLat;
     zoom?: number;
     bearing?: number;
     pitch?: number;
   }

   type LngLat =
     | { lng: number; lat: number }
     | { lon: number; lat: number }
     | [number, number];

   type LngLatBounds =
     | { northeast: LngLat; southwest: LngLat }
     | [LngLat, LngLat]
     | [number, number, number, number];

   // 총 19가지 받을 수 있음

   function focusOnFeature(f: Feature) {
     const bounds = clacBoundBox(f);
     const camera = viewportForBounds(bounds);
     setCamera(camera);
     const {
       center: { lat, lng },
       zoom,
     } = camera;
     // error => 반환 타입의 범위가 넓으면 반환 받을때 어려움을 겪는다.
   }
   ```

   위 코드보단 아래 코드가 바람직

   ```ts
       interface LngLatFix {
       lng: number;
       lat: number;
   };

   type LngLatLike = LngLatFix | { lon: number; lat: number} | [number, number] ;

   interface Camera {
       center: LngLatFix;
       zoom: number;
       bearing: number;
       pitch: number;
   }

   interface CameraOptionsFix extends Omit<Partial<Camera>, 'center'> {
       center?: LngLatLike;
   }

   type LngLatBoundsFix =
       { northeast: LngLatLike, southwest: LngLatLike} |
       [LngLatLike, LngLatLike] |
       [number, number, number, number];

   declare function setCameraFix(camera: CameraOptions): void;
   declare function viewportForBoundsFix(bounds: LngLatBoundsFix): Camera;

   function focusOnFeature2(f: Feature) {
       const bounds = clacBoundBoxFix(f);
       const camera = viewportForBoundsFix(bounds);
       setCamera(camera);
       const {center: {lat, lng}, zoom} = camera; // 정상) 반환 타입은 좁혀진 상태로 하는것이 유리하다.
   ```

2. 결국 옵션이나 유니온 속성은 반환값보다 매개변수 타입에 적용
3. 재사용을 위해 느슨한 구조와 스트릭한 구조를 항상 함께 도입

## item 30: 문서에 타입 정보를 쓰지 않기

1. 주석에 변수명과 타입정보 적기 금지
2. 타입이 명확하지 않은 경우 변수명에 단위 정보 포함

## item 31: 타입 주변에 null 값 배치하기

1. strictNullChecks 설정을 활성화 시키면 오류를 회피하기위해 null check하는 예외 처리 구문이 코드 전반에 추가해야함. 그러나, 값이 전부 null이거나 전부 null이 아닌 경우로 분명하게 구별될 수만 있다면 코드 작성이 훨씬 편함.

   ```ts
   // [0, 1, 2] => [1, 2] 로 변경됨
   // 만약에 nums가 빈배열일떄 [undefined, undefined] 배열이 반환
   // 객체에 undefined 가 포함되는 것은 좋지 않음
   // 배열에 0이 들어 잇으면 의도하지 않게 앞쪽 if(!min) 조건에 성립하게 된다.
   function extent(nums: number[]) {
     let min, max;
     for (const num of nums) {
       if (!min) {
         min = num;
         max = num;
       } else {
         min = Math.min(min, num);
         max = Math.max(max, num); // error, max가 undefined일수도 있으니까
       }
     }

     return [min, max];
   }

   // 수정
   function extentFix(nums: number[]) {
     // 타입 주변에 null 값 배치
     let result: [number, number] | null = null;

     for (const num of nums) {
       if (!result) {
         result = [num, num];
       } else {
         result = [Math.min(num, result[0]), Math.max(num, result[1])];
       }
     }
     return result;
   }
   ```

2. 클래스에서도 null과 null이 아님을 혼용해서 에러가 발생할 수 있음

   ```ts
   // init 함수 내 프로미스가 완료되지 않은 상태에 user와 Post 상태 = null
   // 이때 user와 posts는 둘다 null이거나 모두 null이 아니거나, 둘중 하나만 null인 두가지 상태.

   // 불확실해서 null 체크 남발하게 되어 코드 질이 떨어짐

   class UserPosts {
     user: UserInfo | null;
     posts: Post[] | null;

     constructor() {
       this.user = null;
       this.posts = null;
     }

     async init(userId: string) {
       return Promise.all([
         async () => (this.user = await fetchUser(userId)),
         async () => (this.posts = await fetchPostUser(userId)),
       ]);
     }

     getUser() {
       ///... ?
       return this.user;
     }
   }

   // 수정

   // 모든 값이 null이 아닐때 생성
   class UserPostsFix {
     user: UserInfo;
     posts: Post[];

     constructor(user: UserInfo, posts: Post[]) {
       this.user = user;
       this.posts = posts;
     }

     static async init(userId: string): Promise<UserPostsFix> {
       const [user, posts] = await Promise.all([
         fetchUser(userId),
         fetchPostsForUser(userId),
       ]);
       return new UserPostsFix(user, posts);
     }

     getUserName() {
       return this.user.name;
     }
   }
   ```

## item 32: 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

1. 유니온 타입의 속성을 가지는 인터페이스를 작성중이라면 인터페이스의 유니온 타입을 사용하는게 더 적절한지를 고려해야함.

   ```ts
   // 유니온 타입의 속성을 가지는 인터페이스보다는
   interface Layer {
     layout: FillLayout | LineLayout | PointLayout;
     point: FillPaint | LinePaint | PointPatin;
   }

   // 인터페이스의 유니온 타입 추천
   interface FillLayer {
     layout: FillLayout;
     paint: FillPaint;
   }

   interface LineLayer {
     layout: LineLaout;
     paint: LinePaint;
   }

   interface PointLayer {
     layout: PointLayout;
     paint: PointPaint;
   }
   ```

2. 여러 인터페이스를 유니온(태그된 유니온)으로 작성하는 것이 타입 좁히기에서 더 유리함

   ```ts
   interface FillLayer {
       type : 'fill';
       layout: FillLayout;
       paint: FillPaint;
   }
   interface LineLayer {
       type : 'line';
       layout: LineLayout;
       paint: LinePaint;
   }
   interface PointLayer {
       type : 'point';
       layout: PointLayout;
       paint: PointPaint;
   }

   type Layer = FillLayer | LineLayer | PointLayer;

   function drawLayer(layer: Layer) {
     if(layer.type === 'fill') {
       const {paint} = layer // type => FillPaint
       const {layout} = layer // type =? FillPatin
     }
     <!-- ... -->
   }
   ```

3. 태그된 유니온은 타입스크립트와 잘맞기 때문에(제어 흐름 분석에 좋음) 타입에 태그 넣는 것을 고려하자

   ```ts
   // 지양
   interface Person {
     name: string;
     placeOfBirth?: string;
     dateOfBirth?: Date;
   }

   // 지향
   interface Person {
     name: string;
     birth?: {
       place: string;
       date: Date;
     };
   }
   ```

   만약 api 결과값에 손댈 수 없는 상황에서는 인터페이스의 유니온을 사용해서 속성 사이 관계 모델링 가능

   ```ts
   interface Name {
     name: string;
   }

   interface PersonWithBirth extends Name {
     placeOfBirth: string;
     dateOfBirth: Date;
   }

   type Person = Name | PersonWithBirth;

   function eulogize(p: Person) {
     if ("placeOfBirth" in p) {
       p;
       const { dateOfBirth } = p;
     }
   }
   ```

## item 33: string 타입보다 더 구체적인 타입 사용하기

1. 스트링 타입은 범위가 너무 넓음. 고로 구체적인 타입으로 만들어 쓰자

   ```ts
   // 스트링 남발한것보다
   interface Album {
     artist: string;
     title: string;
     releaseDate: string; //YYYY-MM-DD
     recordingType: string; // 예를 들어 "live" "studio"
   }

   // 타입좁히기
   type RecordingType = "studio" | "live"; //
   interface AlbumFix {
     artist: string;
     title: string;
     releaseDate: Date;
     recordingType: RecordingType;
   }
   ```

2. 위와 같은 방식으로 작성하면 좋은점 3가지

- 타입을 명시적으로 정의해 다른 곳으로 값이 전달되어도 타입 정보 유지
  ```ts
    //함수 호출시 파라미터 타입이 string이 아니라 구체적 허용되는 값이 명시되어 있음.
    function getAlbumsOfType(recordingType: RecordingType): AlbumFix[] {
        <!-- ... -->
    }
  ```
- 해당 타입 의미를 주석으로 설명하기 쉬움
  ```ts
  /** 이 녹음은 어떤 환경에서 이루어졌는지?  */
  type RecordingType = "studio" | "live";
  ```
- keyof 연산자로 세밀하게 객체 속성 체크 가능

  ```ts
  var group: AlbumFix[] = [
    {
      artist: "Dreatm Threater",
      recordingType: "studio",
      releaseDate: new Date("1999-12-12"),
      title: "Dance of the eternity",
    },
    {
      artist: "Queen",
      recordingType: "studio",
      releaseDate: new Date("1990-10-12"),
      title: "Let me live",
    },
  ];

  pluck(group, "artist"); // ['Dreatm Threater', 'Queen']

  // 이렇게하면 any가 너무 넓은 반환값을 유도해 key의 값이 string으로 T에서 허용하지 않는 key를 허용하게 됨.
  function pluck(record: any[], key: string): any[];

  // 이렇게하면 key가 string이라 너무 넓음
  function pluck<T>(record: T[], key: string): any[];

  // 이렇게 작성하면 완벽
  type K = "artist" | "title" | "releaseDate" | "recordingType";

  function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
    return records.map((r) => r[key]);
  }
  ```

## item 34: 부정확한 타입보다는 미완성 타입을 사용하기

1. 타입이 구체적일수록 좋지만 실수가 발생하기 쉬워 조심해야함
2. 정확하게 타입을 모델링할 수 없다면, 부정확하게 모델링하지 말아야함
3. any와 unknown 구분해서 사용

## item 35: 데이터가 아닌, API와 명세를 보고 타입 만들기

1. 코드 안전성을 우이해 api또는 데이터 형식에 대한 타입 생성을 고려
2. 데이터보다는 api 명세로부터 코드 생성

## item 36: 해당 분야의 용어로 타입 이름 짓기

1. 코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 용어들이 존재. 그래서 자체적으로 용어를 만들기 보다는 해당 분야에 이미 존재하는 용어를 사용.
2. 동일한 의미를 나타낼 때는 같은 용어를 사용.
3. 의미적으로 구분되야하는 경우에만 용어를 분리.
4. 모호하고 의미없는 이름은 피해야함
5. 이름을 지을때 포함된 내용이나 계산 방식이 아닌 데이터 자체가 무엇인지를 고려해야함.

## item 37: 공식 명칭에는 상표를 붙이기

1. 구조적 타이핑에서는 가끔 이상한 결과가 도출되기도 함.

   ```ts
   interface Vector2D {
     x: number;
     y: number;
   }

   function clacNorm(p: Vector2D) {
     return Math.sqrt(p.x * p.x + p.y * p.y);
   }

   clacNorm({ x: 3, y: 4 }); // 정상 결과 5
   const vec3D = { x: 3, y: 4, z: 1 };
   clacNorm(vec3D); // 정상 결과 5

   // 구조적 타이핑 관점에서는 이상이 X
   // 수학적 결과에서는 오류
   ```

   이를 해결하기 위해서는 공식명칭을 사용하면 됨

   ```ts
   interface Vector2D {
     _brand: "2d"; // _brand를 통해 calculatedNorm 함수가 Vertor2D 타입만 받도록함
     x: number;
     y: number;
   }

   function vec2D(x: number, y: number): Vector2D {
     return { x, y, _brand: "2d" };
   }

   function calcNorm(p: Vector2D) {
     return Math.sqrt(p.x * p.x + p.y * p.y);
   }

   calcNorm(vec2D(3, 4));
   const vec3D = { x: 3, y: 4, z: 1 };
   calcNorm(vec3D); // 비정상 _brand 속성 없음.
   ```

2. 상표기법은 타입 시스템에서 동작하지만 런타임때 상표검사하는것과 동일한 효과를 얻을 수 있음.

   ```ts
   type ABSPath = string & { _brand: "abs" };
   function listAbsPath(path: ABSPath) {
     console.log(path);
   }

   function isAbsPath(path: string): path is ABSPath {
     return path.startsWith("/");
   }
   function f(path: string) {
     if (isAbsPath(path)) {
       listAbsPath(path); // 정상
     }
     listAbsPath(path); // 비정상
   }
   ```
