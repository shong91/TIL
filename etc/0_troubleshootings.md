1. .babelrc

    **error**
    1) 여러개의 매개변수를 받는 ...args(가변 파라미터) 이 정상적으로 컴파일 되지 않음. 
        ```
        syntax error: unexpected token (${...mapgetters line:column})
        ```
    **solution**
    1) 가변 파라미터는 ES6에서 등장한 문법. ES6 모듈을 읽을 수 있도록 .babelrc 파일에서 플러그인 추가해주어야 함 (.babelrc 파일이 존재하지 않아 문제 발생)
    2) 아래 코드를 .babelrc 파일에 추가하여 해결 (전체 코드는 구글링)
    ```
    "presets": ["env", "stage-2"],
    "plugins": ["transform-vue-jsx", "transform-es2015-modules-commonjs", "dynamic-import-node"]
    ```
    3) babelrc vs babel.config.js ?
    플러그인, 프리셋 등에 대한 설정을 관리한다. 
    - .babelrc
        * 전역 설정파일
    - babel.config.js
        * 지역 설정파일
    4) one more question: main.js 의 import 'babel-polyfill' 만으로는 왜 해결되지 않는걸까?
    - babel은 문법을 변환해주는 역할
    - polyfill은 프로그램이 처음에 시작될 때 현재 브라우저에서 지원하지 않는 함수를 검사해서 각 object의 prototype에 붙여주는 역할
    - 즉, babel은 컴파일-타임에 실행되고 babel-polyfill은 런-타임에 실행된다.

