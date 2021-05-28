# Vuex

**개발하는 애플리케이션의 모든 컴포넌트에 대한 중앙 집중식 저장소 역할 및 관리.**

컴포넌트간 데이터를 주고받기 위해서 부모는 자식에서 props의 방법으로, 자식은 부모에게 Emit event의 방법으로 데이터를 주고 받아야 한다.

또한 형제 컴포넌트간 데이터를 주고 받으려면 너무 복잡해져서 EventBus를 활용해야 그나마 사용을 할 수 있게 됩니다.

간단한 프로젝트인 경우에야 사용이 가능하겠지만 대형 프로젝트인 경우에는 이러한 방법으로는 도저히 감당이 되지 않는다. 이러한 문제를 해결해 주는 것이 Vuex로,
데이터를 store라는 파일을 통해 관리하고 프로젝트 전체에 걸쳐 활용할 수 있게 해 주는 방법이다.

- state
  - 상태값
- mutation
  - 변이. state를 변경하는 방법
  - commit() 으로 호출. 동기처리
- actions
  - mutations을 실행시키는 역할.
  - dispatch(). 비동기처리.
- getters
  - vue component의 computed(계산된 속성) 과 동일. 직접 수정하는 값이 아닌, 상태 값을 활용하여 계산된 속성으로 사용.

## vuex 사용하기

actions 에서 dispatch() 메서드를 이용하여 service의 API를 비동기 call -> response data 를 받아오면,
commit() 메서드를 이용하여 mutations 을 실행 ->
state 데이터를 response data 로 변경(변이) 시킨다. ->
변경된 상태값은 getter 를 통해 vue 컴포넌트에서 값을 받아와 사용할 수 있다.

1. /store/modules/ 에 기능별로 vuex 내용을 작성, 관리할 수 있다.
   - (1) /store/modules/index.js 에서 기능별 js 파일을 export 하고,
   - (2) /store/index.js 에서 (1)의 내용을 import 받아 Vuex 객체를 선언한다.
   - (3) main.js 에서 store 를 import 한 뒤 App.vue 객체 선언 시 적용하여 전체 프로젝트에 반영한다.
2. 1에서 작성한 store 파일은, vue 컴포넌트 내에서 vuex를 import 하여 접근할 수 있다. (참고: ContractInfoMonthTopFiveCharge.vue)
   - computed: {...mapGetters('storename', ['getterprops']), }
   - methods: {...mapActions('storename', ['actionsprops']),}
