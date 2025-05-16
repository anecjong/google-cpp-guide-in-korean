# 헤더 파일

일반적으로 모든 `.cc` 파일에는 관련된 `.h` 파일이 있어야 합니다. 헤더 파일을 올바르게 사용하면 코드의 가독성, 크기 및 성능에 큰 차이를 만들 수 있습니다. 다음 규칙들은 헤더 파일 사용 시 발생할 수 있는 여러 함정을 피하는 데 도움이 될 것입니다.

> `.cc`는 초기 유닉스/리눅스 계열 시스템에서 흔히 사용된 확장자로 Google에서는 C++ 소스 파일에 `.cc` 확장자를 사용합니다. `.cpp`는 마이크로소프트 Visual C++ 컴파일러가 윈도우 환경에서 널리 사용되면서 대중화되었습니다. `.cpp`와 `cc` 모두 기술적으로는 동일한 C++ 소스 파일을 의미하며 **일관성**을 고려해서 사용하면 됩니다.

## 자체 포함(Self-contained) Header

자체 완결적인 헤더란 독립적으로 컴파일이 가능하며, 해당 헤더만 포함하면 추가 작업 없이 모든 기능을 사용할 수 있는 헤더 파일을 의미합니다. 이러한 헤더 파일은 `.h` 확장자를 사용해야 합니다.

모든 헤더들은 자체 포함적이어야 하며 헤더를 `include`하기 위해서 특별한 조건이 없어야 합니다. 예를 들어 사용자가 미리 다른 헤더를 포함하거나, 특정 매크로를 지정할 필요가 없으며 중복 `include` 방지를 위해 헤더 가드가 있어야 합니다.

헤더가 인라인 함수나 템플릿을 선언하고 클라이언트가 이를 인스턴스화해야할 때, 인라인 함수와 템플릿의 정의는 헤더 파일 내에 직접 포함하거나 헤더가 포함하는 다른 파일에 있어야 합니다. 과거에는 이러한 정의를 별도의 헤더 파일(`-inl.h`)로 분리되는 방식이 사용되었으나 더 이상 허용되지 않습니다.

> 포인트 클라우드에 자주 사용되는 pcl(Point Cloud Library)도 [pcl/2d/include/pcl/2d/impl/convolution.hpp](https://github.com/PointCloudLibrary/pcl/blob/master/2d/include/pcl/2d/impl/convolution.hpp)에서 `impl`을 따로 배치하는 것을 확인할 수 있습니다. 

템플릿의 모든 인스턴스화가 하나의 `.cc` 파일 내에서만 일어나는 경우에만, 템플릿 정의가 해당 파일에 위치할 수 있습니다.

> 템플릿의 모든 인스턴스화가 하나의 `.cc` 파일 내에서만 일어나는 경우로 명시적 인스턴스화(Explicit Instantiation)을 수행할 때가 있으며, 템플릿 정의와 모든 인스턴스화를 `.cc` 파일 내에서만 수행합니다. 명시적 인스턴스화를 사용하면 여러 파일에서 같은 템플릿 인자를 사용하는 클래스가 사용되어도 인스턴스화가 한 번만 수행되기 때문에 빌드 시간 단축, 결과 실행 파일 크기 최적화를 할 수 있습니다. 참고로 명시적으로 인스턴스화하지 않은 타입으로 템플릿을 사용하려고 하면 컴파일 타임 에러가 아닌 **링크 타임 에러**가 발생합니다.
```cpp
// mytemplate.h
#ifndef MYTEMPLATE_H
#define MYTEMPLATE_H

// 템플릿 선언은 헤더에서 수행
template <typename T>
class MyTemplate {
public:
    T value_;
    MyTemplate(T val);
    void print();
};


// mytemplate.cc
#include "mytemplate.h"

#include <iostream> // 템플릿 정의에 필요한 헤더

// 템플릿 정의는 .cc 파일에서 수행
template <typename T>
MyTemplate<T>::MyTemplate(T val) : value_(val) {
    // ...
}

template <typename T>
void MyTemplate<T>::print() {
    // 메소드 정의
    // ...
    std::cout << "Value: " << value_ << std::endl;
}

// 모든 인스턴스화가 여기서 발생하도록 명시적으로 지정
// 여러 파일에서 사용되어도 인스턴스화가 한 번만 수행되기 때문에 빌드 시간 단축, 결과 실행 파일 크기 최적화를 할 수 있다.
template class MyTemplate<int>;
template class MyTemplate<double>;
// 만약 다른 타입(예: char)으로 사용될 필요가 있다면 여기에 추가해야 함.
```

예외적으로, 자체 완결적이지 않은 포함용 파일이 필요한 경우가 있으며, 보통 다른 파일의 중간과 같은 특정 위치에 포함되도록 설계되고, 헤더 가드를 사용하지 않거나 필요한 의존성을 모두 포함하지 않을 수 있습니다. 이런 파일은 `.inc` 확장자를 사용하고, 꼭 필요한 경우에만 최소한으로 사용하세요. 가능하면 항상 자체 완결적인 헤더를 사용하는 것이 좋습니다.

## #define 가드

모든 헤더 파일은 다중 포함을 방지하기 위해 `#define` 가드를 가져야 합니다. 심볼 이름의 형식은 `<PROJECT>_<PATH>_<FILE>_H_`로 사용하며 프로젝트 소스 트리의 전체 경로를 기반으로 해 고유성을 보장해야 합니다. 예를 들어, foo 프로젝트의 foo/src/bar/baz.h 파일은 다음과 같은 가드를 가져야 합니다.

```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

...

#endif  // FOO_BAR_BAZ_H_
```

## 필요한 헤더 포함하기

소스나 헤더 파일이 다른 곳에 정의된 심볼을 참조하는 경우, 해당 파일은 그 심볼의 선언이나 정의를 제공하기 위한 헤더 파일을 직접 포함해야 합니다. 헤더 파일을 포함하는 것은 오로지 위의 이유에서여야만 하며 단순히 이 헤더를 포함하면 컴파일이 될 것 같아서, 이 헤더가 다른 유용한 것을 포함하고 있을지도 몰라서와 같은 이유로 포함해서는 안 됩니다.

전이적 포함(transitive inclusions)에 의존하지 마세요. 이렇게 하면 클라이언트 코드를 깨뜨리지 않고도 헤더에서 더 이상 필요하지 않은 `#include` 문을 제거할 수 있습니다. 심지어 `.cc` 파일이 그에 대응하는 `.h` 파일을 통해 필요한 헤더를 간접적으로 얻을 수 있는 상황에서도, `.cc` 파일이 직접 사용하는 헤더는 별도로 직접 포함해야 합니다.

> `.cc`에서도 직접 `#include`를 사용하라는 규칙으로 구글의 Abseil 코드에서 확인할 수 있습니다. [crc32c.h](https://github.com/abseil/abseil-cpp/blob/master/absl/crc/crc32c.h)와 [crc32c.cc](https://github.com/abseil/abseil-cpp/blob/master/absl/crc/crc32c.cc)에서 각각 헤더를 포함하며 두 파일에서 사용하는 헤더 목록이 다른 것을 확인할 수 있습니다.

## 전방 선언 (Forward Declaration)

### 정의
```cpp
// C++ 소스 파일에서 전방 선언의 예:
class MyClass;              // 클래스 전방 선언
void SomeFunction();        // 함수 전방 선언
extern int some_variable;   // 변수 전방 선언
ABSL_DECLARE_FLAG(my_flag); // 매크로를 통한 플래그 전방 선언
```
전방 선언은 엔티티의 완전한 정의 없이 그 존재만 컴파일러에게 알려주는 것입니다.
> C++에서 엔티티는 일반적으로 코드 내에서 명명되고 참조될 수 있는 모든 요소(타입, 변수, 함수, 상수, 네임스페이스, 매크로 등)를 의미합니다.

### 장점
- `#include`를 사용하면 컴파일러가 더 많은 파일을 열고 처리하게 만들지만, 전방 선언은 불필요한 헤더 포함을 줄여 컴파일 시간을 단축할 수 있습니다.
- `#include`를 사용하면 헤더 파일에서 연관되지 않은 변경이 생겨도 재컴파일 되지만, 전방 선언은 이런 불필요한 재컴파일 과정을 줄일 수 있습니다.

### 단점
- 전방 선언은 의존성을 숨길 수 있으며, 헤더가 변경되더라도 사용자 코드의 필수적인 재컴파일이 생략될 수 있습니다. (전방 선언이 있지만 적절한 `#include`를 사용하지 않았다면 링크 오류나 런타임 오류 발생할 수 있습니다.)
- `#include`와 달리 전방 선언은 자동화된 툴이 해당 심볼을 정의하는 파일을 찾아내기 어렵게 만듭니다.
- 전방 선언은 라이브러리의 변경에 의해 깨질 수 있습니다. 이는 라이브러리 개발자가 API를 유연하게 개선하는 것을 제약합니다.
- `std::` 네임스페이스에 있는 심볼을 전방 선언하는 것은 정의되지 않은 행동(Undefined Behavior)을 야기할 수 있습니다.
- 전방 선언으로 충분한지 아니면 전체 `#include`가 필요한지 판단하기 어려울 수 있습니다. 어떤 경우에는 포인터/참조만 사용하지만, 어떤 경우에는 멤버에 접근해야 하므로 완전한 정의가 필요하며 이는 구분하기 어려울 수 있습니다. 아래 예시에서 `#include "b.h"`가 전방 선언으로 변경되면 `test` 함수는 `void f(void*)`를 호출합니다.
```cpp
// b.h:
struct B {};
struct D : B {};

// good_user.cc (헤더 포함 버전):
#include "b.h"
void f(B*);
void f(void*);
void test(D* x) { f(x); }  // f(B*)를 호출합니다

// bad_user.cc (전방 선언 버전):
class B;
class D;
void f(B*);
void f(void*);
void test(D* x) { f(x); }  // 상속 관계를 모르므로 f(void*)를 호출합니다!
```
- 여러 심볼을 전방 선언하는 것은 `#include`를 수행하는 것보다 더 장황하게 표현될 수 있습니다.
- 전방 선언을 한 클래스를 사용할 때에는 포인터나 참조를 사용하게 되며, 포인터 멤버를 사용하면 코드가 느려지고 복잡해 질 수 있습니다.
```cpp
// header.h
class Dependency; // 전방 선언

class MyClass {
private:
    Dependency m_dependency; // Dependency의 크기를 알 수 없어 컴파일 에러가 발생합니다.
    Dependency* m_dependency; // Dependency* 포인터의 크기는 알 수 있어 컴파일이 가능합니다.
};
```
### 결론
다른 프로젝트에서 정의된 엔티티를 전방 선언하는 것을 피하세요. 대신 필요한 헤더를 직접 포함하세요. 

## 헤더 파일에서 함수 정의하기

헤더 파일에서 함수의 정의는 정의가 짧을 때(10줄 이하)만 선언 지점에 포함하세요. 그 외의 경우에는 일반적으로 구현은 .cc 파일에 넣는 것이 좋습니다. ODR(One Definition Rule)은 C++ 언어의 중요한 규칙으로, 프로그램 전체에서 각 함수, 클래스, 변수 등이 **정확히 한 번만** 정의되어야 한다는 원칙입니다. 헤더 파일은 여러 소스 파일에 포함될 수 있으므로, 헤더에 정의가 있다면 ODR 위반 위험이 있습니다.

### 정의
헤더 파일의 정의된 함수들은 인라인 함수라고 불리는데, 이 용어는 다음과 같은 상황을 가리키는 여러 의미로 사용되는 용어입니다.
- 선언과 함께 정의된 심볼: 선언 위치에서 독자에게 바로 정의가 노출됩니다.
- 인라인 확장 가능: 헤더 파일에 정의된 함수나 변수는 컴파일러가 인라인 확장(호출 대신 코드 직접 삽입)할 수 있도록 정의가 제공되므로, 더 효율적인 오브젝트 코드를 생성할 수 있습니다.
- ODR-safe 엔티티: ODR(One Definition Rule)을 위반하지 않는 개체로, 헤더 파일에 정의된 것들은 종종 ODR을 만족하기 위해 `inline` 키워드가 필요합니다.
인라인의 개념은 함수 뿐만 아니라 C++17부터 도입된 `inline` 변수에도 적용될 수 있습니다. 

### 장점
- 함수를 선언 위치에 바로 정의하면 getter나 setter와 같은 간단한 함수들의 반복적인 코드를 줄일 수 있습니다.
- 작은 함수는 인라인 확장될 경우 함수 호출 오버헤드가 사라져 성능이 향상될 수 있습니다.
- 함수 템플릿과 `constexpr` 함수는 선언된 헤더 파일에 정의되어야 합니다.

### 단점
- 긴 함수가 API 선언 사이에 끼어 있으면 API를 보기 어렵게 만듭니다.
- 공개된 정의는 구현 세부사항을 노출하는데 캡슐화 원칙에 어긋나며 사용자에게 불필요한 정보입니다.

### 결론
10줄 이하의 짧은 함수만 선언 위치에서 정의하세요. 더 긴 함수는 성능이나 기술적인 이유(템플릿 등)로 반드시 헤더에  있어야 하는 것이 아니면 `.cc` 파일에 두세요.

정의가 반드시 헤더에 있어야 한다고 해서, 공개 API 부분에 있어야한다는 충분한 이유가 되지는 않습니다. 대신 정의는 헤더의 내부 영역, 클래스의 `pirvate` 섹션, `internal`이라는 단어를 포함하는 네임스페이스 내부, 또는 특정 주석 아래에 배치할 수 있습니다.

일단 정의가 헤더 파일에 포함되면 명시적 `inline` 지정자를 사용하거나, 암묵적으로 `inline` 처리가 되는 함수 템플릿 또는 클래스 본문 내의 정의된 함수여야 ODR-safe 합니다.

```cpp
template <typename T>
class Foo
{
public:
    int bar()
    {
        return bar_;
    }

    void MethodWithHugeBody();

private:
    int bar_;
};

// Implementation details only below here

template <typename T>
void Foo<T>::MethodWithHugeBody()
{
    ...
}
```

## 포함(`#include`)의 이름과 순서

아래의 순서로 헤더를 포함(`#include`)하세요.
- Related Headers: 현재 소스 파일과 직접 관련된 헤더 파일(보통 같은 이름)
- C System Headers: `<unistd.h>`, `<stdlib.h>` 같은 C 시스템 헤더
- C++ Standard Library Headers: `<string>`, `<vector>` 같은 C++ 표준 라이브러리 헤더
- Other Libraries' Headers: 서드파티 라이브러리 헤더
- Project's Headers: 프로젝트 내부 헤더 파일

> 관련 헤더(Related Headers)를 가장 먼저 포함하면, 해당 헤더가 다른 헤더에 의존하고 있을 때 빌드 오류가 즉시 나타납니다. 이렇게 하면 다른 개발자가 실수로 포함해둔 헤더에 의존하는 상황을 방지할 수 있습니다.

프로젝트의 모든 헤더 파일은 UNIX 디렉토리 별칭인 `.` (현재 디렉토리) 또는 `..` (부모 디렉토리)를 사용하지 않고 프로젝트의 소스 디렉토리의 하위로 나열되어야 합니다. 예를 들어, google-awesome-project/src/base/logging.h는 다음과 같이 포함되어야 합니다:

```cpp
#include "base/logging.h"
```

라이브러리가 요구하는 경우에만 각괄호(angle bracket, `< >`)를 사용하여 헤더를 포함해야 합니다. 아래는 그 예시입니다.
- C 및 C++ 표준 라이브러리 헤더(예: `<stdlib.h>`, `<string>`).
- POSIX, 리눅스 및 윈도우 시스템 헤더(예: `<unistd.h>` 및 `<windows.h>`).
- 드물게 서드파티 라이브러리(예: `<Python.h>`).
각 그룹이 끝날 때마다 빈 줄로 구분하세요.

`stddef.h`와 같은 C 헤더는 C++ 대응물(`cstddef`)과 교환 가능합니다. 어느 스타일을 사용해도 괜찮으나, 기존 코드와의 일관성을 유지하세요.

각 섹션 내에서 헤더는 알파벳 순으로 정렬되어야 합니다. 오래된 코드는 이 규칙을 따르지 않을 수 있으나 가능할 때 수정해야 합니다.

예를 들어, google-awesome-project/src/foo/internal/fooserver.cc의 포함은 다음과 같을 수 있습니다:

```cpp
#include "foo/server/fooserver.h"

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include "base/basictypes.h"
#include "foo/server/bar.h"
#include "third_party/absl/flags/flag.h"
```

**예외 사항**

```cpp
#include "foo/public/fooserver.h"

#include "base/port.h"  // LANG_CXX11 때문에.

#ifdef LANG_CXX11
#include <initializer_list>
#endif  // LANG_CXX11
```

시스템별 코드에 조건부 `include`가 필요할 수 있으며, 다른 `include`가 선행되어야 할 수 있습니다.

### 참고 사항
**전통적인 소스 코드 위치**
```
project/
  ├── include/
  │   └── project/
  │       ├── feature.h
  │       └── utils.h
  └── src/
      ├── feature.cpp
      └── utils.cpp
```
- 공개 API(헤더)와 내부 구현(소스)이 명확히 구분됩니다.
- 라이브러리 배포 시 헤더만 쉽게 제공할 수 있습니다.
- 관련 파일을 편집할 때 디렉토리 간 이동이 필요합니다.
- https://github.com/google/googletest - 외부 배포 목적으로 위와 같이 정리되었을 것으로 예상됩니다.

**Google 방식 (헤더와 소스 같은 디렉토리)**
```
src/
  ├── base/
  │   ├── logging.h
  │   ├── logging.cc
  │   ├── string_utils.h
  │   └── string_utils.cc
  └── project/
      ├── feature.h
      └── feature.cc
```

- 관련 파일이 같은 위치에 있어 탐색이 쉽습니다.
- 라이브러리 사용자 입장에서 공개 API와 내부 구현이 명확히 구분되지 않을 수 있습니다.
- https://github.com/abseil/abseil-cpp.git