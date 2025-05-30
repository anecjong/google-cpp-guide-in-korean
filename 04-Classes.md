# 클래스

클래스는 C++에서 코드의 기본 단위입니다. 이 섹션에서는 클래스를 작성할 때 따라야 할 주요 규칙들을 설명합니다.

## 생성자에서의 작업 수행
### 정의
생성자 내에서 단순 멤버 변수 할당 외에도 임의의 초기화 작업을 수행하는 것이 가능합니다.

### 장점
- 클래스가 초기화되었는지 아닌지에 대해 걱정할 필요가 없습니다. 별도의 `Init()` 같은 초기화 함수를 호출해야하는 번거로움과 깜빡 잊고 호출하지 않아 발생하는 문제를 줄일 수 있습니다.
- 생성자 호출로 완전히 초기화된 객체는 `const`로 선언될 수 있으며, 표준 컨테이너나 알고리즘과 같이 사용하기 더 용이할 수 있습니다.
### 단점
- 가상 함수를 호출한다면, 이 호출은 파생(derived) 클래스의 구현으로 전달되지 않습니다. 다형성(polymorphism)이 제대로 동작하지 않아 항상 기반(base) 클래스의 함수를 호출합니다.
- 생성자가 리턴 값을 반환하지 않으므로 오류를 알리는 쉬운 방법은 없습니다. 프로그램을 강제 종료시키거나 예외를 사용하는 것 외에는 마땅한 방법이 없습니다.
- 생성자 작업이 실패하면 객체는 부분적으로만 초기화된 상태로 남을 수 있습니다. 이런 객체를 그대로 사용하면 문제가 발생하므로 사용 전에 `bool isVaild()`와 같은 상태 확인 메커니즘을 필요로 할 수 있으나 호출을 잊어버리기 쉽습니다.
- 생성자의 주소를 가져올 수 없기 때문에, 생성자 내에서 수행되는 작업들은 별도의 스레드에서 비동기적으로 실행시키거나, 다른 부분으로 작업을 위임하는 등의 유연한 처리가 어렵습니다.
### 의사 결정
- 생성자에서는 가상 함수를 절대 호출하지 마세요.
- 코드 상황에 따라 적절하다면 프로그램 종료가 적절한 방법일 수 있습니다. 오류를 좀 더 유연하게 처리하고 싶다면 생성자에서는 최소한의 초기화만 하고 별도의 `Init()` 멤버 함수에서 나머지 초기화 및 오류 처리를 수행하는 방법을 고려할 수 있습니다.
- 호출 가능한 공개 메서드에 영향을 미치는 다른 상태가 없는 객체(데이터 컨테이너나 유틸리티 클래스)에 대해서는 `Init()` 메서드 사용을 피해야 합니다.

## 암시적 변환
### 정의
암시적 변환은 한 타입(source type)이 다른 타입(destination type)으로 컴파일러가 알아서 바꿔주는 기능입니다. 가장 흔한 예로 `int`를 `double`이 필요한 곳에 넣으면 컴파일로가 자동으로 변환해 줍니다. C++은 언어 기본 타입 간의 변환 외에, 직접 만든 클래스에 대해서도 암시적 변환 규칙을 만들 수 있습니다. 소스 타입에서 암시적 변환은 목적지 타입의 이름을 딴 타입 변환 연산자(`operator bool()`)로 정의됩니다. `explicit` 키워드를 생성자나 변환 연산자 앞에 붙이면 이런 암시적 변환을 막을 수 있으며 `static_cast`처럼 명시적으로 바꾸도록 요청해야 변환됩니다.

### 장점
- 암시적 변환은 명백한 경우 타입을 명시적으로 지정할 필요를 없애주어 타입을 더 사용하기 쉽고 읽기 쉽게 만듭니다. (`std::string s = std::string("hello");`)
- 암시적 변환은 오버로딩의 더 간단한 대안이 될 수 있습니다. `std::string`과 `const char*`에 대해 `std::string_view`로 매개변수를 받으면 간결해집니다.
- 중괄호 초기화 구문은 객체를 초기화하는 간결하고 표현력 있는 방법입니다. (`MyType obj = {var1, var2};`)

### 단점
- 암시적 변환은 타입 불일치 버그를 숨길 수 있으며, 사용자가 어떤 변환이 일어날 것이라는 사실을 인지하지 못할 수 있습니다.
- 암시적 변환은 코드를 더 읽기 어렵게 만들 수 있으며, 오버로딩이 있는 경우 어떤 코드가 호출되는지 덜 명확하게 만듭니다.
- 단일 인자를 받는 생성자는 의도하지 않았더라도 우연히 암시적 타입 변환으로 사용될 수 있습니다.
- 단일 인자 생성자가 `explicit`으로 표시되지 않으면, 그것이 암시적 변환을 정의하려는 의도인지 단순히 표시하는 것을 잊었는지 확실히 알 수 있는 방법이 없습니다.
- 암시적 변환은 호출 지점에서 모호성을 유발할 수 있으며, 특히 양방향 암시적 변환이 있을 때 문제가 더 심각해질 수 있습니다.
- 중괄호 초기화는 목적지 타입이 암시적일 경우, 특히 리스트가 단일 요소만 가지고 있을 때 동일한 문제를 겪을 수 있습니다.

### 의사 결정
- 타입 변환자(`operator OtherType()`)와 단일 인자로 호출 가능한 생성자는 클래스 정의에서 반드시 `explicit`으로 표시해야 합니다.
- 예외적으로 복사 생성자와 이동 생성자는 타입 변환을 수행하지 않으므로 `explicit`으로 표시해서는 안 됩니다. (표시하는 경우 `ClassName obj2 = obj1;`이 되지 않고 `ClassName obj2(obj1);` 형태만 가능해짐)
- 상호 교환 가능하도록 설계된 타입에 대해 필요하고 적절할 수 있습니다. 예를 들어 두 타입의 객체가 동일한 기본 값의 다른 표현일 경우 그렇습니다. 이 경우 프로젝트 리더에게 이 규칙의 면제를 요청하세요.
- 단일 인자로 호출될 수 없는 생성자는 `explicit`을 생략할 수 있습니다. 단일 `initializer_list` 매개변수를 받는 생성자도 복사 초기화를 지원하기에 `explicit`을 생략해야 합니다.

## 복사 가능하고 이동 가능한 타입
### 정의
이동(move)는 객체의 리소스(동적 할당 메모리 등)의 소유권을 다른 객체로 효율적으로 넘기는 것을 의미합니다. 이동 가능한 타입(movable type)은 임시 객체(temporaries)로부터 초기화되거나 대입될 수 있는 타입으로 주로 소멸될 임시 객체로부터 리소스를 가져오는 방식으로 동작합니다. 복사 가능한 타입(copyable type)은 같은 타입의 다른 어떤 객체로부터 초기화되거나 대입될 수 있는 타입이며, 원본 객체의 값은 변경되지 않는다는 조건이 따릅니다. 예를 들어 `std::unique_ptr<int>`는 이동 가능하지만 복사 불가능한 타입입니다. 다른 예로 `int`와 `string`은 이동 가능하면서 복사도 가능한 타입입니다.

사용자 정의 타입에서 복사/이동은 다음 멤버 함수들에의해 결정됩니다.
- 복사 생성자 (Copy Constructor): `MyClass(const MyClass& other)`
- 복사 대입 연산자 (Copy-Assignment Operator): `MyClass& operator=(const MyClass& other)`
- 이동 생성자 (Move Constructor): `MyClass(MyClass&& other)`
- 이동 대입 연산자 (Move-Assignment Operator): `MyClass& operator=(MyClass&& other)`
만약 이동 관련 함수들이 명시적으로 정의되어 있지 않다면, 이동이 필요한 상황에서 컴파일러는 복사 관련 함수들을 대신 사용하려고 시도합니다.

복사/이동 생성자는 특정 상황에서 컴파일러에 의해 암시적으로 호출될 수 있으며, 예를 들어 함수에 객체를 값으로 전달하거나 반환할 때 호출 될 수 있습니다.

### 장점
- 복사 가능하고 이동 가능한 타입의 객체는 값으로 전달하고 반환할 수 있어 API를 더 간단하고, 안전하며, 일반적으로 만듭니다.
- 복사/이동 생성자 및 대입 연산자는 컴파일러에게 암시적으로 또는 `= default`를 통해 생성될 수 있어 일반적으로 `Clone()`, `CopyFrom()`, `Swap()`과 같은 대안보다 올바르게 정의하기가 더 쉽습니다.
- 잘 구현된 이동 생성자는 rvalue 객체로부터 리소스를 암시적이고 효율적으로 이전할 수 있게 합니다.

### 단점
- 모든 클래스를 복사 가능하게 만드는 것은 좋지 않습니다.
  - 싱글톤: 전역적으로 유일해야하는 객체는 복사되면 여러 인스턴스가 생겨 싱글톤의 의미가 깨집니다.
  - 스코프 바운드: RAII 객체처럼 특정 스코프를 벗어날 때 정리 작업을 수행하는 객체는 복사되었을 때 누가 실제 리소스를 소유하고 정리해야 하는지 모호해집니다.
  - 뮤텍스: 뮤텍스는 고유한 동기화 객체로 복사된 뮤텍스는 원본 뮤텍스와 다른 락을 가지게 되어 의도된 동기화를 수행할 수 없습니다.
- 다형적으로 사용될 기본 클래스 타입에 대한 복사 연산은 객체 잘림(Object slicing)을 일으켜 파생 클래스 부분의 정보는 사라질 수 있습니다.
- `= default`로 지정되거나 부주의하게 구현된 복사 연산은 잘못될 수 있으며, 진단하기 어려울 수 있습니다.
- 복사 생성자는 암시적으로 호출되어 호출 자체를 놓기치 쉬우며 과도한 복사를 유발해 성능 문제를 일으킬 수 있습니다.

### 의사 결정
- 모든 클래스의 공개 인터페이스는 해당 클래스가 어떤 복사 및 이동 연산을 지원하는 지를 명확히 해야 하며, `public` 섹션에 적절한 연산을 명시적으로 선언하거나 삭제하는 형태로 이루어져야 합니다. 또한 복사 또는 이동 대입 연산자를 제공한다면, 해당 생성자도 제공해야 합니다.
```cpp
class Copyable
{
  public:
  Copyable(const Copyable& other) = default;
  Copyable& operator=(const Copyable& other) = default;

  // The implicit move operations are suppressed by the declarations above.
  // You may explicitly declare move operations to support efficient moves.
};

class MoveOnly
{
  public:
  MoveOnly(MoveOnly&& other) = default;
  MoveOnly& operator=(MoveOnly&& other) = default;

  // The copy operations are implicitly deleted, but you can
  // spell that out explicitly if you want:
  MoveOnly(const MoveOnly&) = delete;
  MoveOnly& operator=(const MoveOnly&) = delete;
};

class NotCopyableOrMovable
{
  public:
  // Not copyable or movable
  NotCopyableOrMovable(const NotCopyableOrMovable&) = delete;
  NotCopyableOrMovable& operator=(const NotCopyableOrMovable&) = delete;

  // The move operations are implicitly disabled, but you can
  // spell that out explicitly if you want:
  NotCopyableOrMovable(NotCopyableOrMovable&&) = delete;
  NotCopyableOrMovable& operator=(NotCopyableOrMovable&&) = delete;
};
```
- 이러한 선언/삭제는 그 기능이 명백한 경우에만 생략될 수 있습니다.
  - 모든 멤버가 `public`이고 단순한 데이터 집합체(aggregate)인 경우, 멤버들이 복사/이동 가능하면 해당 `struct`도 복사/이동 가능하다고 유추할 수 있습니다.
  - 기본 클래스가 복사/이동을 금지(`= delete`)했다면, 파생 클래스도 이를 상속받아 복사/이동이 불가능해집니다. 하지만 기본 클래스가 아무것도 선언하지 않았다면, 파생 클래스에서 복사/이동 가능성을 명확히 해줘야 합니다.
  - 복사에 대해 생성자나 대입 연산자 중 하나를 명시적으로 선언하거나 삭제하면, 다른 복사 연산은 명백하지 않으므로 선언하거나 삭제해야 합니다. 이동 연산에 대해서도 마찬가지입니다.
- 파일 핸들과 네트워크 소켓같은 리소스를 관리하는 클래스와 같이 복사/이동의 의미가 일반 사용자에게 불분명하거나 예상치 못한 비용을 발생시킨다면 해당 타입은 복사/이동 가능해서는 안 됩니다. 복사 가능한 타입에서 이동 연산은 복사보다 현저히 빠를 때만 가치 있습니다. 클래스가 포인터 멤버를 가지는 등 특별한 관리가 있다면 `default`가 아니라 사용자 정의 로직이 필요합니다.
- 객체 잘림의 위험을 제거하려면 생성자를 `protected`로 만들거나, 소멸자를 `protected`로 선언하거나, 하나 이상의 순수 가상 멤버 함수를 제공하여 기본 클래스를 추상 클래스로 만드는 것을 선호하세요. 구체적인 클래스로부터 파생하는 것을 되도록 피하세요.

## `struct` vs `class`
데이터를 담는 수동적인 객체에만 `struct`를 사용하고, 그 외의 모든 것은 `class`를 사용하세요.

`struct`의 모든 필드는 `public`으로 선언하고, 불변(invariant) 조건은 금지됩니다. 예를 들어 `width > 0`과 같은 조건을 만들지 마세요. 단순한 초기화나 편의를 위한 간단한 메서드는 `struct`에 포함될 수 있으나 객체의 내부 상태를 복잡하게 관리하거나 특정 불변 조건을 강제하지 않아야 합니다.

더 많은 기능이나 불변 조건이 필요하거나, `struct`가 광범위하게 사용되고 앞으로 변경될 가능성이 있다면 `class`가 더 적절합니다. 확실하지 않으면 `class`로 만드세요. STL과 일관성을 위해서 상태를 가지지 않는 타입들, `trait`, `template metafunction`, `functor` 등에는 `class` 대신 `struct`를 사용할 수 있습니다.

`struct`와 `class`의 멤버 변수는 서로 다른 이름 규칙을 가집니다.
- `class`의 멤버 변수: `lower_case_with_underscores_` (끝에 밑줄 _ 추가)
- `struct`의 멤버 변수: `lower_case_with_underscores` (끝에 밑줄 없음)

## `struct` vs `pair`, `tuple`
요소들이 의미 있는 이름을 가질 수 있다면, `std::pair`, `std::tuple` 대신 `struct`를 사용하는 것을 선호하세요. `pair`와 `tuple`을 사용하면 사용자 정의 타입을 정의할 필요가 없어 코드 작성 시 작업을 줄일 수 있지만, 읽을 때는 `.first`, `.second` 또는 `std::get<X>`보다 의미 있는 필드 이름이 항상 명확합니다.

## 상속(Inheritance)
### 정의
파생 클래스가 기반 클래스로부터 상속받을 때, 파생 클래스는 기반 클래스가 정의한 모든 데이터와 연산의 정의를 포함하게 됩니다. 인터페이스 상속(interface inheritance)은 순수 추상 기본 클래스로부터의 상속이며 그 외의 모든 상속은 구현 상속(implemenation inheritance)입니다.

### 장점
- 구현 상속은 기존 타입을 특화시키면서 기본 클래스 코드를 재사용하여 코드 크기를 줄입니다.
- 상속은 컴파일 타임 선언이므로 개발자와 컴파일러 모두 연산을 이해하고 오류를 감지할 수 있습니다.
- 인터페이스 상속은 클래스가 특정 API를 노출하도록 프로그래밍 방식으로 강제하는데 사용할 수 있습니다.

### 단점
- 구현 상속에서 파생 클래스를 구현하는 코드가 기반 클래스에 분산되어 있기 때문에 구현을 이해가 어렵습니다. 기반 클래스의 비가상 함수는 파생 클래스에서 오버라이드할 수 없습니다.
- 다중 상속은 종종 더 높은 성능 오버헤드를 유발하고, 다이아몬드 상속 패턴으로 이루어질 위험이 있습니다.

### 의사 결정
- 모든 상속은 `public`이어야 합니다. 만약 `private` 상속을 하고 싶다면, 대신 기본 클래스의 인스턴스를 멤버로 포함해야 합니다.
- 클래스를 기본 클래스로 사용하는 것을 지원할 의도가 없다면, 해당 클래스에 final을 사용할 수 있습니다.
- `protected`의 사용은 하위 클래스에서 접근해야 할 수도 있는 멤버 함수로 제한하십시오. 데이터 멤버는 `private`이어야 함에 유의하십시오. 필요한 경우 getter/setter를 이용해 접근하세요.
- 파생 클래스가 기반 클래스의 가상 메서드를 오버라이드를 할 때는 `override` 키워드를 사용하세요. 오버라이드를 선언할 때 `virtual` 키워드는 중복이므로 사용하지 마세요.
- 다중 인터페이스 상속은 허용하지만, 다중 구현 상속은 권장되지 않습니다.

## 연산자 오버로딩
### 정의
C++읜 매개변수 중 하나가 사용자 정의 타입이기만 하면, 사용자 코드가 `operator` 키워드를 사용해 내장 연산자의 오버로드된 버전을 선언하는 것을 허용합니다. `operator""` 문법을 사용해 프로그래머가 자신만의 리터럴 접미사(`10.0_km`과 같이 숫자 뒤에 특정 접미사를 붙이기)를 만들 수 있게하며 타입 변환 연산자(`operator bool()`, ...)를 지정할 수 있습니다.

### 장점
- 연산자 오버로딩은 사용자 정의 타입이 내장 타입과 동일하게 동작하도록 해 코드를 간결하고 직관적으로 만들 수 있습니다.
- 사용자 정의 리터럴은 사용자 정의 타입의 객체를 생성하기 위한 매우 간결한 표기법입니다.
```cpp
Distance operator"" _m(long double val)
{
  return Distance(static_cast<double>(val));
}

auto distance = 100.0_m;
```

### 단점
- 정확하고, 일관된 연산자 오버로드 세트를 제공하려면 세심한 주의가 필요하며, 그렇지 못할 경우 혼란과 버그를 유발할 수 있습니다.
- 연산자를 과도하게 사용하면 코드가 난해해집니다.
- 연산자 오버로드는 비용이 많이 드는 연산을 저렴한 내장 연산처럼 보이게 숨길 수 있습니다.
- 오버로드된 연산자의 호출 위치를 찾는 것은 특수한 도구의 도움을 필요로 합니다.
- 오버로드된 연산자의 인자 타입을 잘못 지정하면, 컴파일 오류 대신 다른 오버로드가 호출될 수 있습니다. 예를 들어, `foo < bar`와 `&foo < &bar`는 전혀 다른 동작을 할 수 있습니다.
- 특정 연산자 오버로드는 본질적으로 위험합니다. 단항 `&` (주소 연산자) 오버로딩은 오버로드 선언이 보이는지에 따라 동일한 코드가 다른 의미를 갖게 할 수 있습니다. `&&`, `||`, `,` 연산자의 오버로딩은 내장 연산자의 평가 순서 의미 체계와 일치하게 만들 수 없습니다.
- 연산자는 종종 클래스 외부에 정의되므로, 서로 다른 파일에서 동일한 연산자에 대해 다른 정의를 도입할 위험이 있습니다.
- 사용자 정의 리터럴(UDL)은 숙련된 C++ 프로그래머에게도 생소한 새로운 구문 형태를 만들 수 있게 합니다 (예: `std::string_view("Hello World")`의 약자로 `"Hello World"sv`). 기존 표기법이 덜 간결하더라도 더 명확합니다.
- UDL `operator""`는 네임스페이스 내에 정의될 수 있지만, 사용하려면 `using namespace some_namespace;` 나 `using some_namespace::operator"" _suffix;` 가 필요합니다. Google C++ 스타일 가이드에서는 헤더 파일에서의 `using` 지시어나 광범위한 `using` 선언을 금지합니다.

### 의사 결정
- 최소 놀람의 원칙(Principle of Least Astonishment)을 따라 사용자가 연산자를 보고 기대하는 바로 그 동작을 하도록 만들어야 합니다.
- 자신이 소유한 타입에 대해서만 연산자를 정의하고, 다른 라이브러리 타입이나 내장 타입에 대해 연산자를 오버로드하지 마세요
- 연산자 정의는 관련된 타입 정의와 함께 두고, 관련 연산자 일관성을 위해 `operator==`를 정의했다면 `operator!=`도 정의하세요.
- 내부를 변경하지 않는 이항 연산자는 비멤버함수로 정의하는 것을 선호하세요. `friend` 키워드를 사용해 클래스의 `private` 멤버에 접근 가능하게 하세요.
- 값이 동등 비교 가능한 타입 `T`에 대해, 비멤버 `operator==`를 정의하고 두 `T` 타입 값이 언제 같다고 간주되는지 문서화하세요.
- 연산자 오버로드 정의를 피하기 위해 굳이 애쓰지 마십시오. 예를 들어, `Equals()`, `CopyFrom()`, `PrintTo()` 대신 `==`, `=`, `<<`를 정의하는 것을 선호하세요. 반대로, 다른 라이브러리가 기대한다는 이유만으로 연산자 오버로드를 정의하지 마세요. 예를 들어, 타입에 자연스러운 순서가 없지만 `std::set`에 저장하고 싶다면, `<`를 오버로드하는 대신 사용자 정의 비교자(custom comparator)를 사용하세요.
- `&&`, `||`, `,`, 단항 `&`, `operator""`를 오버로드하지 마세요. 다른 곳(표준 라이브러리 포함)에서 제공하는 그러한 리터럴도 사용하지 마세요.

## 접근 제어
클래스의 데이터 멤버는 상수가 아닌 한 `private`으로 만드세요. 접근자(accessor, 보통 `const`) 형태의 약간의 간단한 상용구(boilerplate) 코드를 작성하는 비용이 들지만, 불변 조건(invariants)에 대한 추론을 단순화할 수 있습니다.

기술적인 이유로, Google Test를 사용할 때 `.cc` 파일에 정의된 테스트 픽스처(test fixture) 클래스의 데이터 멤버는 `protected`로 하는 것을 허용합니다. 만약 테스트 픽스처 클래스가 사용되는 `.cc` 파일 외부(예: `.h` 파일)에 정의된다면 데이터 멤버를 `private`으로 하세요.

## 선언 순서
유사한 선언들을 함께 그룹화하고 `public` 부분을 먼저 배치하세요. 클래스 정의는 `public`, `protected`, `private` 순서로 작성합니다.

각 섹션 내에서 유사한 종류의 선언들을 함께 그룹화하고, 아래의 순서를 따르는 것을 선호하세요.
1. 타입 및 타입 별칭 (`typedef`, `using`, `enum`, 중첩 구조체/클래스, `friend` 타입)
2. (선택 사항, `struct` 전용) 비정적 데이터 멤버
3. 정적 상수 (Static constants):
4. 팩토리 함수 (Factory functions):
5. 생성자 및 대입 연산자 (Constructors and assignment operators)
6. 소멸자 (Destructor)
7. 그 외 모든 함수 (정적 및 비정적 멤버 함수, `friend` 함수)
8. 그 외 모든 데이터 멤버 (정적 및 비정적)

```cpp
class MyExampleClass
{
public:
  // 1. Types and type aliases
  using DataMap = std::map<std::string, int>;
  enum class Status
  {
    kOk,
    kError
  };

  // 3. Static constants
  static constexpr int kDefaultRetries = 3;

  // 4. Factory functions (example)
  static std::unique_ptr<MyExampleClass> Create()
  {
    return std::make_unique<MyExampleClass>();
  }

  // 5. Constructors and assignment operators
  MyExampleClass();
  MyExampleClass(const MyExampleClass& other);
  MyExampleClass& operator=(const MyExampleClass& other);

  // 6. Destructor
  ~MyExampleClass();

  // 7. All other functions
  Status processData(const DataMap& data);
  void reset();
  static void logMessage(const std::string& msg);

protected:
  // ... protected members in the same order if any ...
  void protectedHelper();

private:
  // 7. (Helper functions might be private too)
  bool validateInput(const DataMap& data);

  // 8. All other data members
  int retry_count_;
  DataMap internal_data_;
  static Logger* static_logger_;
};
```
- 큰 메서드 정의를 클래스 정의 내부에 인라인으로 넣지 마세요.