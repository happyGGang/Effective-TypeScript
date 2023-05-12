# 타입 설계

## item : 사용할 때는 너그럽게, 생성할 때는 엄격하게

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

## item : 문서에 타입 정보를 쓰지 않기

1. 주석에 변수명과 타입정보 적기 금지
2. 타입이 명확하지 않은 경우 변수명에 단위 정보 포함

## item : 타입 주변에 null 값 배치하기

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

## item : 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

## item : string 타입보다 더 구체적인 타입 사용하기

## item : 부정확한 타입보다는 미완성 타입을 사용하기

## item : 데이터가 아닌, API와 명세를 보고 타입 만들기

## item : 해당 분야의 용어로 타입 이름 짓기

## item : 공식 명칭에는 상표를 붙이기
