# 개발 환경 설정

1. IDE: VS code
2. install plugin in VS code: Vetur Extension, Vue peek Extension, Vue 3 snippet Extension
3. install NodeJS: check version (cmd: node -v, npm -v)
4. install Chrome extension: Vue Devtools
5. install Vue CLI & create project
   $npm install vue-cli -g
    $vue -V or $vue --version 
    $vue init <template-name> <project-name> ex) vue init webpack hello-world

   \*\* 초기 생성 파일

   1. .babelrc: 웹 브라우저의 크로스 브라우징을 위해 ES6 -> ES5 로 코드 변환 필요. 바벨이 트랜스파일링 세팅을 정의한다.
   2. node_modules, package.json: npm을 통해 설치한 패키지들.

   3. src>assets: 애플리케이션에서 사용되는 정적 리소스. 빌드 시 webpack 이 처리한다.
   4. src>static: 이미지, 폰트와 같은 정적 리소스. webpack을 거치지 않는다.

# vue 에서 proxy table 사용하기.

webapp과 webapi 를 분리하여 개발하다 보면, webapi 호출이 CORS 이슈로 인해 제대로 불리지 않는 경우가 있다. 이 경우

1. 백엔드 쪽에서 CORS 를 허용

- 라이브러리/패키지를 사용하여 지정 도메인/포트에 한해 CORS 허용.

2. 프론트엔드 쪽에서 우회

- proxy table을 이용하여 지정 도메인/포트에 한해 CORS 허용.
- 프론트엔드가 열려있는 localhost:${port}/api/* 로 시작하는 http request 들은 proxy table의 target 주소인 localhost:${api-port}/ 으로 CORS 이슈 없이 요청이 가게 된다.

```config.js
devServer: {
	// ...
	proxy: {
        	"/api": {
            	target: 'http://localhost:${api-port}'
            },
        },
    // ...
}
```
