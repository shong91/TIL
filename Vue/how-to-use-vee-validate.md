# vee-validate 라이브러리 사용법

## 1. import library

```
import VeeValidate from 'vee-validate'; // main.js 에 import
import { ValidationObserver, ValidationProvider, extend } from 'vee-validate'; // validator를 사용하고 싶은 컴포넌트에 import
```

## 2. define rules

기본적으로 vee-validate 에서 제공하는 룰을 사용할 수 있으며, 커스터마이징도 가능하다.

### 2-1) vee-validate/dist/rules 사용

```
import {
  numeric,
  alpha_dash,
  alpha_space,
  required
} from 'vee-validate/dist/rules';
```

### 2-2) 커스터마이징

금번 프로젝트에서는 src/plugins/globalValidateRules.js 에서 validation rule 을 정의하고,

해당 js 파일을 src/plugins/index.js 에서 Vue 에서 사용할 수 있도록 선언하였다.

## 3. 컴포넌트에서 validator 사용하기

validator 를 사용하기 위해 기본적으로 아래 구조를 따른다.

- validation-observer, validation-provider 사용
- ValidationProvider 에 rules 라는 이름으로 사용하고 싶은 rule을 넘겨주고, ValidationProvider 에 errors 라는 이름의 v-slot을 생성. (provider 가 errors를 자동으로 생성해준다.)
- v-model: v-model에 바인딩된 값으로 validation rules 을 체크한다.

```
<validation-observer v-slot="{ invalid }" slim>
<validation-provider rules="alpha_dash|numeric|required" v-slot="{ errors }">
  <input v-model="value" type="text">
  <span>{{ errors[0] }}</span>
  </validation-provider>
</validation-observer>
```

## References

1. 공식문서: https://vee-validate.logaretm.com/v2/
2. 참고: https://velog.io/@pear/Vee-Validate-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EC%82%AC%EC%9A%A9%EB%B2%95
