# Mixin ?

믹스인은 이름이 나타내고 있는 것 처럼 컴포넌트에 무언가를 섞어 원하는 것을 구현하는 기능이다.

여러 개의 컴포넌트에서 중복하여 사용되는 코드를, mixnin 을 활용하여 코드의 반복을 피하고 특정 기능을 나타내는 파일을 캡슐화할 수 있다.

![mixin-image](https://media.vlpt.us/images/bluestragglr/post/29770123-f72e-4293-8edf-ebe39725de29/Untitled%202.png)

**mixin 동작과정**

1. 믹스인할 객체를 만들고
2. 컴포넌트에 객체를 믹스인하고
3. 나머지 부분을 구현해 완성한다.

**mixin 객체 생성 원칙**

1. HTML/CSS는 믹스인하지 않는다. (순수 자바스크립트로 작성된 객체)
2. 믹스인에서 선언한 속성(eg. data)를 컴포넌트에서 사용할 수 있으며,
   동일한 이름의 속성을 컴포넌트에서 다시 선언할 경우 컴포넌트에 선언된 값이 우선하여 병합된다.

**vue 에서 적용하기**

1. mixin 파일 생성

```
// @/mixin/MyMixin.js

let MyMixin = {
  mounted() {
    console.log('Mixed!')
  }
}
export default MyMixin
```

2. 컴포넌트에서 import mixin

```
// @/component. js
import MyMixin from 'mixins/MyMixin'
export default {
  mixins: [MyMixin]
}
```

_주의할 점_

믹스인을 사용하는 경우, 라이프사이클 훅은 일반 컴포넌트의 라이프사이클 훅과 똑같이 동작한다. (created / mounted 등 라이프사이클에 맞추어 믹스인을 사용하여야 함)

### References

1. https://velog.io/@bluestragglr/Vue.js-Mixin-%EA%B8%B0%EB%8A%A5-%EB%B0%98%EB%B3%B5-%EC%A0%9C%EA%B1%B0%ED%95%98%EA%B8%B0
