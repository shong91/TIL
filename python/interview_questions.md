# [python 기술 면접 질문 리스트]

# 파이썬의 특징
## 특징 및 장점
- 객체지향 언어: 캡슐화/추상화/상속/다형성/데이터 숨기기
- 인터프리터 언어: 실행 전 컴파일이 필요 없음
- 동적 타이핑 언어: 변수 사용 전 선언할 필요 없음
- 함수가 first-class object: 변수에 할당 가능, 다른 함수에서 반환 및 전달 가능 
- 간결, 단순

## 단점
- 인터프리터 언어 -> 속도 약점
- 모바일 컴퓨터에 약점이 있음
- 브라우저 안에서 javascript 처럼 객체를 다룰 수 없음
- 동적 타이핑 언어 -> 덕 타이핑(Duck-typing) 사용. 이로 인한 런타임 에러 발생 가능성 있음 

### cf) 컴파일 언어와 인터프리터 언어의 차이점
- 컴파일 언어: 컴파일러를 통해 구현되며, 소스코드를 기계어로 바꾸는 과정을 사전에 처리하여 빠르게 구동될 수 있도록 함(ex Java)
- 인터프리터 언어: 소스코드를 컴파일하지 않고, 인터프리터를 통해 소스코드 실행 시 각 스테이트먼트를 하나 이상의 서브루틴 순서로 변환한 후, 기계어/바이트코드 등 다른 언어로 변환하여 구현한다.
 
(.pyc: .py 파일의 컴파일 된 버전. python 은 성능 향을 위해 자동으로 pyc 파일을 생성하며, 이는 PVM에 의해 실행된다.)


# list, tuple, dict 의 차이점
- list 
    - 변경 가능한 목록 (mutable list). 더 느림
    - 입력 순서가 유지됨 (index)
    - 내부적으로 동적 배열로 구현되어 있음
    - list = [1,2,3,4,5]
      list[-1] : 오른쪽 데이터를 기준으로 1번째 값
      list[::-1]: 리스트를 역순으로 
      list.append(): 리스트의 제일 뒤에 값 추가
      list.expend(): 한 리스트의 요소를 다른 리스트의 요소로 추가하여 붙임 
- tuple
    - 변경 불가한 목록(immutable list). 더 빠름
    - 입력 순서가 유지됨
    - tuple = ('a', 'b', ('ab', 'cd'))
- dict
    - 키/값으로 이루어진 딕셔너리
    - 입력 순서 없음
    - 내부적으로 해시 테이블로 구현되어 있음
    - dict = {'name':'hong', 'age': '20'}
- Set
    - 집합 자료형
    - 입력 순서 없음 (unordered)
    - 중복 허용 X
    - set = set("hello")   -> {'e', 'h', 'l' ,'o'}


# 파이썬 파일 처리 모드 
파이썬은 open()을 통해 파일을 열 때, 아래와 같은 파일 처리 모드가 있다. 
- r: 읽기 전용
- w: 쓰기 전용
- rw: 읽기 & 쓰기
- a: 덧붙이기 
- t: 열기
- rt: 읽기 전용으로 열기
- b: 바이너리 파일

# decorator 의 사용 
함수 객체에 새로운 기능을 추가하는 등, 빠르게 변경할 때 사용하는 방법이다. 

```
def decorator_sample(func):
    def decorator_hook(*args, **kwargs):
        print("before the function call")
        result = func(*args, **kwargs)
        print("after the function call")
        return result
    return decorator_hook

@decorator_sample
def product(x, y):
    # function to multiply two numbers. 
    return x * y

print(product(3,3))

```
# 파이썬에서의 메모리 관리
- python private heap space(개별적인 힙)에서 관리됨 

    -> 내부적으로 힙 매니저가 구현되어 있어, 모든 객체와 자료구조를 가지고 있다. 힙 매니저는 객체에 대한 힙 스페이스 할당/할당 해제를 관리한다. 
- 내장된 garbage collector가 있어서 사용되지 않는 메모리 관리, 재사용 가능

# Lambda와 Def의 차이점
- def
    - 다중 표현
    - 함수를 생성하고 이름을 지정하여 나중에 다시 호출한다. 
    - 선언을 반환할 수 있음
- lambda
    - 단일 함수
    - 함수 객체를 구성하고 반환한다. 
    - 선언을 반환하지 않음

# ternary operator (삼항 연산자)의 사용법 
- [true_value] if [condition] else [false_value]

# *Args/*Kwargs
- *args : 얼마나 많은 argument가 함수에 전달될지 모르는 경우 or 저장된 list, tuple을 함수에 전달할 때 사용
- **kwargs : 얼마나 많은 keyword argument가 함수에 전달될지 모르는 경우 or 저장된 dictionary를 함수에 전달할 때 사용(key 를 가지고 있음)

# 파이썬 generator란
- iterator를 생성하는 특수 함수
- 함수 내 yield 키워드를 통해 데이터 하나씩 return:

    yield는 return과 달리 반환 후에 종료되지 않고 그 상태를 유지한다.
- 메모리의 효율적 사용을 가능케 함:

    list compression을 통한 방법과 generator의 방법을 비교해보면 LC는 리스트를 모두 메모리에 올리기 때문에 그 크기만큼 메모리를 차지하는 사이즈가 커집니다. 하지만 generator는 next()메소드로 차례로 값에 접근하여 메모리에 올리게 됩니다.

# 객체지향/상속 관련 개념
## 상속 (Inheritance)
- 다른 클래스의 모든 멤버(속성, method) 받는 것
- 파이썬은 multiple inheritance도 가능해서 여러 부모 class로부터 상속 가능

### Composition?
- 파이썬에서 상속의 한 종류
- 기본 클래스에서 상속을 하지만 파생 클래스의 멤버 역할을 하는 기본 클래스의 인스턴스 변수를 사용합

## Polymorphism
- 부모 class 내의 method를 자식 class에서 overriding

## Overloading & Overriding
- Overloading : 같은 공간 내 동일 이름 함수 정의 BUT 매개변수 유형, 개수 다르게
- Overriding : 상위 class의 method를 하위 class에서 재정의

## Call by reference & Call by object
- Call by value : 인자로 받은 값을 복사하여 처리
- Call by reference : 인자로 받은 값의 주소값을 참조
- Call by object : Python 내에서는 모든 것을 객체로 보고 immutable object(int, float, str, tuple)은 call by value로 동작 / mutable object(list, dict, set)은 call by reference로 동작

# 파이썬 GIL이 무엇이고, 왜 성능 문제가 발생하는가?
Global Interpreter Lock. 

파이썬 객체가 다중 스레드가 동시에 실행되지 않도록 하는 것. 이를 통해 쓰레드에 대해 안정성 있는 접근을 보장한다. 

# MVT 패턴의 처리 과정
- 클라이언트로부터 요청을 받으면 URLconf를 이용하여 URL을 분석한다.
- URL 분석 결과를 통해 해당 URL에 대한 처리를 담당할 뷰를 결정한다.
- 뷰는 자신의 로직을 실행하면서 만일 데이터 베이스 처리가 필요하면 모델을 통해 처리하고 그 결과를 반환받는다.
- 뷰는 자신의 로직 처리가 끝나면 템플릿을 사용하여 클라이언트에 전송할 HTML 파일을 생성한다.
- 뷰는 최종 결과로 HTML 파일을 클라이언트에게 보내 응답한다.

# Django ORM: Object Relational Mapping
## 개념
- 객체와 관계형 데이터베이스를 연결해주는 역할
- OOP언어와 데이터를 다루는 RDBMS 와의 상이한 시스템을 매핑하여, 데이터 관련 OOP 프로그래밍을 쉽게 하도록 도와주는 기술

## 작동
- 하나의 모델클래스는 하나의 테이블에 매핑
- 모델 클래스의 속성은 테이블의 컬럼에 매핑

## 장단점
- sql 쿼리문을 몰라도 쉽게 사용이 가능,
- 데이터 베이스엔진을 변경하더라도 ORM을 통한 API 변경은 필요 없다
- 단, ORM이 반드시 효율적인 sql로 변환해 주는 것은 아님

## ORM의 방식: Lazy-Loading (Lazy Evaluation; 지연 연산)
개념
- 명령을 실행할 때마다 db에서 데이터를 가져오는것이 아니라, 모든 명령 처리가 끝나고 실제로 데이터를 불러와야 할 시점이 왔을 때 db에 쿼리를 실행하는 방식
- 매 단계마다 쿼리를 실행하지 않기 때문에 쿼리 요청을 최소화 할 수 있다
- Nested Serializer(중첩)일때 성능 문제 (쿼리 반복 수행)
- 해결법: Eager-Loading
    - 사전에 쓸 데이터를 포함하여 쿼리를 날리기 때문에 비효율적으로 늘어나는 쿼리 요청을 사전에 방지할 수 있다
    - 장고 ORM에서는 prefetch_related로 이미 한번 불러왔기 때문에 가져오기만(fetch)하면 됨 (1:N 관계일 때 select_related
    N:M 관계일 때 prefetch_related)
    - local data cache에 가져온 데이터 저장

# http와 https의 차이점은?
- http: HyperText Transfer Protocol. 
    - HTTP 서버는 기본 포트인 80번 포트에서 서비스 대기 중이며, 클라이언트(웹 브라우저)가 TCP 80 포트를 사용해 연결하면 서버는 요청에 응답하면서 자료(정보)를 전송한다. 
    - HTTP는 단순 텍스트를 주고 받기 때문에 네트워크에서 전송 신호를 인터셉트하는 경우 원하지 않는 데이터 유출이 발생할 수 있다.
    - HTTP의 보안 취약점을 해결하기 위한 프로토콜이 HTTPS다.
- https: HTTP + Secure Socket
    - HTTP와 거의 동일하지만, 데이터를 주고 받는 과정에 '보안'요소가 추가된다
    - 클라이언트는 공개키를 얻어 데이터를 암호화해서 전송하고, 서버는 개인키를 이용해 보호화한다.
 


# 도커 컨테이너 내부에 nat가 존재할 수 있나요?

# 파이썬의 GC는 어떻게 작동하나요?

# 프로세스와 쓰레드는 어떻게 다른가요?

# 왜 celery 백엔드로 SQS를 사용했나요? 그때 문제는 없었나요?

# pypy는 도대체 왜 CPython보다 빠른 걸까요?

# 웹 서비스 응답이 느리다면 어떻게 해결할 수 있을까요?

# python에서 메모리 릭이 발생할 수 있는 경우는 언제일까요?

# 도커와 VM의 차이점은 뭔가요?




## Interview List References:
- https://shelling203.tistory.com/31
- https://118k.tistory.com/646
- https://heehehe-ds.tistory.com/entry/CS-Python-%EA%B8%B0%EC%B4%88-for-%EA%B8%B0%EC%88%A0-%EB%A9%B4%EC%A0%91-%EC%A7%88%EB%AC%B8-%EB%8C%80%EB%B9%84
- https://www.edureka.co/blog/interview-questions/python-interview-questions/#basicinterviewquestions