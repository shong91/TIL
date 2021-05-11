# vue 에서 proxy table 사용하기.
webapp과 webapi 를 분리하여 개발하다 보면, webapi 호출이 CORS 이슈로 인해 제대로 불리지 않는 경우가 있다. 이 경우 

1) 백엔드 쪽에서 CORS 를 허용
- 라이브러리/패키지를 사용하여 지정 도메인/포트에 한해 CORS 허용. 
2) 프론트엔드 쪽에서 우회 
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