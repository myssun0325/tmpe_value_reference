# tmpe_value_reference
잠시 누군가를 위해 예전에 공부했던 것을 공유하기 위한..


----------

* Value Type : 각 인스턴스들은 자신만의 각 데이터카피본을 가지고 있습니다. 값 타입을 변수나 상수에 할당하거나 함수에 인자로 전달할 때 새로운 인스턴스(copy되어서)를 생성합니다.

* Reference Type : 각 인스턴스들은 하나의 데이터 카피본을 공유합니다. 타입이 한번 초기화되면 이미 존재하는 인스턴스의 참조만을 넘깁니다.(변수 , 상수 , 함수의 인자)

* Reference는 인스턴스를 저장하는 heap메모리 주소에 대한 참조입니다.

* 참조타입(예: 클래스)의 var와 let의 차이는 다른 인스턴스를 재할당(참조)할수있느냐 없느냐의 차이
* strcut (값타입)은 프로퍼티의 값을 바꿀 수 없다.


### struct is not the only value type and class is not the only reference type in swift.

* 정리하다가 생각난건데 값타입을 써야 side effect를 줄일 수 있다.
* If you always get a unique, copied instance, you can trust that no other part of your app is changing the data under the covers. 

* 특히 멀티스레드환경에서 유용하다. This is especially helpful in multi-threaded environments where a different thread could alter your data out from under you. This can create nasty bugs that are extremely hard to debug.

* When you’re working with Cocoa, many APIs expect subclasses of NSObject, so you have to use a class -> 코코아나 다른 NSObject의 서브클래스를 기대하는 API를 사용할 때는 클래스를 사용해라.

### 타입의 가이드라인

* 값타입
  - instance를 `==` 가지고 비교할 때
  - You want copies to have independent state
  - The data will be used in code across multiple threads
* 참조타입
  - Comparing instance identity with === makes sense
  - You want to create shared, mutable state

* stack과 heap자료
  - value : stack
  - reference : heap

## 그럼 힙과 스택은 뭐가 다르냐

### 스택은?

- 만약 값타입(value type) 인스턴스가 class instance의 일부이면 값은 클래스의 인스턴스와 마찬가지로 heap에 저장됩니다.
- stack은 static memory allocation에 사용된다. heap은 dynamic memory allocation에 사용된다. (둘 다 RAM)
- stack은 CPU에 의해 tightly하게 관리되고 최적화된다.
- 함수가 변수를 생성할 때, stack은 변수를 저장하고, 함수가 종료될 때 변수를 destroy한다.
- 스택에 할당된 변수(Variables)는 메모리에 직접 저장되고, 메모리에 대한 액세스가 빠르다. 
- 함수나 메서드가 다른 함수를 차례대로 호출할 때, 마지막으로 호출된 함수가 값(value)를 return하기 전까지 이전에 호출된 함수들은 모두 remain suspend한다.
  - 스택은 항상 LIFO. 가장 마지막 reserved block은 항상 be freed될 다음 block이다. 이것은 스택을 추적하는것을 매우 간단하게 해준다.(keep track of the stack)
  - 스택에서 block을 해제(free)하는 것은 한개의 포인터만 조정하기만 하면된다.
  - 이런 구성으로 인해 스택은 매우 빠르고 효율적이다.

### 힙은?

- 시스템은 heap을 다른객체에 의해 참조되고있는 데이터를 저장하기위해 사용한다.
- 힙은 시스템이 요청할 수 있고, 메모리 블록(blocks of memory) dynamically하게 할당할 수 있는 메모리 풀이다. (large pool of memory from which the system can request and dynamically allocate blocks of memory)
- 힙은 스택처럼 자동으로 메모리가 해제되지 않기 때문에 추가적으로 메모리를 해제시켜줘야 한다.
- apple device에서는 ARC가 그 작업을 해준다. ARC에 의해 tracked되는 reference count가 있고 이 reference count가 0이 될 때, object가 dellocated된다.
- 이 과정 ( allocation, tracking the references and deallocation)으로 인해 stack보다 느린거다. 그래서 value type이 reference type보다 빠를 수 있는거다.

### 스택오버플로우 답변

- 클래스가 선호될 때
  - Copying or comparing instances doesn't make sense (e.g., Window)
  - 인스턴스의 lifetime이 외부의 영향에 의존적일 때 (Instance lifetime is tied to external effects (e.g., TemporaryFile))
  - Instances are just "sinks"--write-only conduits to external state (e.g.CGContext)
- 이 답변자는 The Swift programming language guide이 class와 struct 사용 case 가이드라인이 조금 모순됐다고 말하고 있다.
- 가이드라인
  - 다음 조건 중 하나 이상의 조건일 때 구조체 사용권장
    1. 연관된 간단한 data values을 encapsulate하는게 주 목적일 때
    2. 인스턴스를 전달하거나 할당할 때 encapsulated values가 참조되는것보다 복사되는 것이 더 합리적일 때
    3. 구조체에 value type으로 저장된 프로퍼티들이 참조되는것보다 복사되는것이 합리적일 때
    4. 이미 존재하는 다른타입으로 부터 프로퍼티나 메서드를 상속할 필요가 없을 때
  - Examples of good candiate for structures include:
    1. The size of a geometric shape, perhaps encapsulating a width property and a height property, both of type Double.
    2. A way to refer to ranges within a series, perhaps encapsulating a start property and a length property, both of type Int.
    3. A point in a 3D coordinate system, perhaps encapsulating x, y and z properties, each of type Double.

- 답변자는 문서의 뉘앙스는 구조체를 사용할경우를 빼고 기본으로 class를 사용하라고 하지만, 실상 반대가 되어야 한다고 생각한다.
- 이런 개념들은 계속 변화하고 있으며, 스위프트 언어 가이드라인은 protocol oriented programming이 발표되기 전에 작성되어서 이런것일 수 있다고 한다.


### 이 외에 다른 블로그에 나와있는 의견들

[참고링크](https://www.letmecompile.com/swift-struct-vs-class-%EC%B0%A8%EC%9D%B4%EC%A0%90-%EB%B9%84%EA%B5%90-%EB%B6%84%EC%84%9D/)

* struct
  - stack memory 영역에 할당 -> 속도가 빠름
  - scope based lifetime: 컴파일타임에 compiler가 언제 메모리를 할당/해제할지 정확히 알고있음.
  - data locality: CPU 캐시 히트율이 높음
  - 상속불가능 (protocol은 사용가능)
  - NSData로 serialize 불가능
  - Codable 프로토콜을 이용하여 쉬운 JSON <-> Struct 변환가능 (swift 4이상)
  - 항상 새로운 변수로 copy가 일어나기때문에 multi-thread 환경에서 공유변수로 인해 문제를 일으킬 확률이 적음

* class
  - heap memory 영역에 할당 (속도가 느림)
  - 런타임에 직접 alloc하며 reference counting을 통해 dealloc이 필요
  - memory fragmentation 등의 overhead가 존재
  - 상속 가능
  - NSData serialize 가능
  - Codable 사용 불가능
  - 런타임에 타입 캐스팅을 통해서 클래스 인스턴스에 따라 여러 동작이 가능
  - deinitializer 존재

* 참고: class안에 struct 변수를 property로 정의하는것 가능하며, 반대로 struct의 property중 하나로 class 인스턴스 변수를 갖고있는 것도 가능하다. 이 경우 해당 struct 변수의 copy가 일어날때 class 인스턴스의 주소값만 복사된다.


### 이 분이 정한 룰

* 상속이 필요하지 않고 모델의 사이즈가 그리 크지 않다면 struct를 사용
* JSON의 필드와 1:1 mapping되는 간단한 모델이 필요하다면 struct를 사용 (JSON대신 다른 데이터 encoder/decoder를 구현가능하지만 Swift에서는 JSON만 제공됨)
* 해당모델을 serialize 해서 전송하거나 파일로 저장할 일이 있다면 class 사용
* 해당 모델이 Obj-C에서도 사용되어야 한다면 class 사용

### 특수케이스: 클로저와 struct

* 앞서 언급했듯이 struct, Int 등의 value type 변수들은 변수에 할당하거나 함수 파라메터로 전달시에 value copy가 일어난다. 하지만 Swift에서 value type 변수가 closure에 의해 capture가 되는 경우에는 reference copy가 기본값이다. 이를 피해서 명시적으로 value copy가 하고싶다면 해당 변수를 capture list에 넣어주면 된다.

  - [Swift Closure vs Objective-C Block 비교 분석](https://www.letmecompile.com/swift-closure-vs-objective-c-block/)

  

### ㅎㅎㅎㅎㅎㅎ

* dsfsdf
  - sdfkljsldjk

* [참고링크](https://www.naver.com/)
