
# Vuex 
* state
    - 상태값
* mutation
    - 변이. state를 변경하는 방법
    - commit() 으로 호출. 동기처리 
* actions
    - mutations을 실행시키는 역할.
    - dispatch(). 비동기처리. 
* getters
    - vue component의 computed(계산된 속성) 과 동일. 직접 수정하는 값이 아닌, 상태 값을 활용하여 계산된 속성으로 사용. 

* /store/* 에 작성한 내용은, vue component 내에서 vuex를 import 하여 접근할 수 있다. ...mapActions([])