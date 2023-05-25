# 타입 선언과 @types

## item 45 : devDependencies에 typescript와 @types 추가하기  
npm은 세가지 종류의 의존성을 구분해서 관리하며,  
각각의 의존성은 package.sjon 파일내의 별도영역으로 구분된다.  

 - dependencise : 현재 프로젝트를 실행하는데 필수적인 라이브러리  
                  런타임에 사용되는 라이브러리로 구성  
                  프로젝트를 다른 사용자가 설치한다면, dependencies에 들어 있는 라이브러리도 함께 설치될 것  
 - devDependencise : 현재 프로젝트를 개발하고 테스트하는 데 사용되지만, 런타임에는 필요없음  
                     예를 들어 프로젝트에서 사용 중인 테스트 프레임워크
 - peerDependencise : 런타임에 필요하긴 하지만,  의존성을 직접 관리하지 않는 라이브러리들이 포함  
                      예로는 플러그인이 있음  

타입스크립트는 당연히 devDependencise에 포함됨  

