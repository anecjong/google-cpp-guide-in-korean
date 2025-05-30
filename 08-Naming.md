# Naming

가장 중요한 일관성 규칙은 이름 짓기로, 이름만 보고도 타입, 변수, 함수, 상수, 매크로 등을 알 수 있는 명확한 규칙을 정하고 일관되게 지키는 것은 매우 중요합니다. 이렇게 하면 코드 가독성이 크게 향상됩니다.

## 이름 선택하기 (Choosing Names)
소유자와 다른 팀에 있는 사람이라도 새로운 독자가 목적이나 의도를 이해할 수 있도록 이름을 지으세요. 새로운 독자가 즉시 이해할 수 있도록 하는 것이 훨씬 중요하므로 짧게 줄여 쓰는 것보다 의미를 명확히 전달하는 것이 우선입니다.

이름이 사용될 문맥을 고려하세요. 이름의 사용 범위가 넓을 수록 많은 문맥 정보를 이름 자체에 담아야 합니다. 반대로, 범위가 좁은 지역 변수는 짧고 간결해도 문맥상 의미가 명확할 수 있습니다.

프로젝트 외부의 누군가에게 알려지지 않았을 가능성이 있는 약어의 사용을 최소화하세요. `customer`를 `cstmr`으로 줄이는 것처럼, 단어 내에서 글자를 삭제해 약어를 만들지 마세요. 약어를 사용하는 경우 하나의 단어(`StartRPC()` 보다는 `StartRpc()`)로 대문자화 하는 것을 선호하세요. `i`, `j`, `k`와 같은 루프 변수, `T`, `U`와 같은 템플릿 파라미터들은 관례적으로 허용됩니다.

`Status`, `OkStatus`, `Error` 등과 같이 프로젝트나 라이브러리 전반에 걸쳐 핵심적인 의미를 가지며 자주 사용되는 어휘 이름들이 있습니다. 이런 이름들은 간결하더라도 그 의미가 문서를 통해 잘 공유되어야 합니다.

## 파일 이름 (File Names)
파일 이름은 모두 소문자여야 하며 밑줄(`_`)이나 대시(`-`)를 포함할 수 있습니다. 프로젝트에서 사용하는 관례를 따르되, 일관된 로컬 패턴이 없다면 `_`를 선호하세요. 파일 이름은 `my_useful_clas.cc` 형태를 따르며 테스트 파일은 `_test.cc`로 끝나게 하세요.

C++ 소스 파일 확장자로 `.cpp` 대신 `.cc`를 사용하는 것이 Google의 스타일입니다. 헤더 파일은 `.h` 확장자를 가져야 하며 특정 지점에서 텍스트로 포함되어야 하는 파일은 `.inc`로 끝나야 합니다.

`/usr/include`에 이미 존재하는 파일 이름(`db.h`)을 사용하지 마세요. 일반적인 이름보다는 파일 내용을 잘 나타내는 구체적인 이름을 사용해야 합니다.

## 타입 이름 (Type Names)
```cpp
// classes and structs
class UrlTable { ...
class UrlTableTester { ...
struct UrlTableProperties { ...

// typedefs
typedef hash_map<UrlTableProperties *, std::string> PropertiesMap;

// using aliases
using PropertiesMap = hash_map<UrlTableProperties *, std::string>;

// enums
enum class UrlTableError { ...
```
모든 종류의 타입 이름(클래스, 구조체, `typedef`, `using`으로 만든 별칭, 열거형, 템플릿 타입 파라미터)는 **PascalCase**를 사용합니다.

C++20에 도입된 컨셉 이름도 동일한 규칙을 따릅니다.

## 변수 이름 (Variable Names)
변수 및 데이터 멤버의 이름은 **snake_case**를 사용합니다. 구조체가 아닌 클래스의 데이터 멤버는 추가로 끝에 `_`를 붙입니다.
- `a_local_variable`
- `a_struct_data_member`
- `a_class_member_`

```cpp
class TableInfo
{
public:
    static const int kTableVersion = 3; // OK - 상수 이름 규칙.
private:
    std::string table_name_;       // OK - 끝에 밑줄.
    static Pool<TableInfo>* pool_; // OK. (정적이지만 상수가 아니므로 끝에 밑줄)
};

struct UrlTableProperties
{
    std::string name;
    int num_entries;
    static Pool<UrlTableProperties>* pool; // 정적이지만 상수가 아니므로 일반 변수 규칙
};
```
클래스 멤버 변수는 **snake_case_**, `static const` (또는 `static constexpr`) 멤버는 상수 이름 규칙(**kPascalCase**)을 따릅니다

## 상수 이름 (Constant Names)
`constexpr` 또는 `const`로 선언되고 프로그램 실행 동안 값이 고정된 변수는 맨 앞에 "k"를 붙이고  **kPascalCase** 형태를 사용합니다. (예: `kMaxIterations`, `kDefaultTimeoutMs`).

```cpp
void ComputeFoo(absl::string_view suffix)
{
    // 둘 다 허용됨.
    const absl::string_view kPrefix = "prefix";
    const absl::string_view prefix = "prefix";
}
void ComputeFoo(absl::string_view suffix)
{
    // 나쁜 예 - ComputeFoo의 다른 호출은 kCombined에 다른 값을 제공함.
    const std::string kCombined = absl::StrCat(kPrefix, suffix);
}
```
- kPascalCase 적용 대상: 전역 `const`, `static const`, 네임스페이스 `const`, 클래스의 `static const` 멤버.
- 함수 내 지역 `const` 변수: kPascalCase를 사용해도 되고, 일반 변수처럼 snake_case를 사용해도 됩니다.

## 함수 이름 (Function Names)
일반적인 함수(전역 함수, 클래스 멤버 함수)는 **PascalCase**를 사용합니다. Getter/Setter 함수는 예외적으로 **snake_case**를 사용할 수 있습니다.
- Getter: `int num_errors() const;`, Setter: `void set_num_errors();`와 같이 표현하면 실제 멤버 변수와 이름이 유사합니다.

## 네임스페이스 이름 (Namespace Names)
네임스페이스 이름은 **snake_case**를 사용하세요. 네임스페이스 이름을 선택할 때, 네임스페이스 외부의 헤더에서 사용할 때는 정규화된 이름을 사용해야 한다는 점에 유의하세요.

최상위 네임스페이스는 다른 프로젝트와 충돌하지 않도록 고유해야 하며 보통 프로젝트나 회사 이름 기반으로 합니다.

`project_name/component_name/file.h` 는 `project_name::component_name` 네임스페이스로 설정하는 것과 같이 네임스페이스 이름과 디렉토리 구조를 일치시키는 것이 일반적입니다.

중첩된 네임스페이스는 잘 알려진 최상위 네임스페이스(특히 `std` 및 `absl`)의 이름을 피해야 합니다. `my_namespace::std`와 같은 네임스페이스는 중첩 네임스페이스가 전역 `std`와 혼동될 수 있으므로 피해야 합니다.

## 열거자 이름 (Enumerator Names)
```cpp
enum class UrlTableError
{
    kOk = 0,
    kOutOfMemory,
    kMalformedInput,
};

// 나쁜 예 (예전 스타일)
// enum class AlternateUrlTableError
// {
//     OK = 0,
//     OUT_OF_MEMORY = 1,
//     MALFORMED_INPUT = 2,
// };
```
`enum` 또는 `enum class` 내부의 각 열거자의 이름은 상수 이름 규칙과 동일하게 **kPascalCase** 를 사용합니다. 예전에는 **ALL_CAPS_WITH_UNDERSCORES** 스타일을 사용하기도 했지만, 매크로와의 충돌 문제 등으로 인해 현재는 **kPascalCase**를 권장합니다.

## 템플릿 매개 변수 이름 (Template Parameter Names)
템플릿 매개변수는 해당 범주의 이름 지정 스타일을 따라야 합니다. 타입 템플릿 매개변수는 타입 이름 규칙을 따라야 하고, 비타입 템플릿 매개변수는 변수 또는 상수 이름 규칙을 따라야 합니다.

## 매크로 이름 (Macro Names)
- 매크로 사용은 강력히 비권장됩니다. (다른 섹션에서 매크로 사용 지침이 있음)
- 꼭 사용해야 한다면, **ALL_CAPS_WITH_UNDERSCORES** 를 사용하고, 다른 이름과의 충돌을 피하기 위해 프로젝트별 접두사를 붙입니다.

## 별칭 (Aliases)
별칭의 이름은 다른 새 이름과 동일한 원칙을 따르며, 원래 이름이 나타나는 곳이 아니라 별칭이 정의되는 컨텍스트에 적용됩니다.
```cpp
using NewName = OldType;
typedef OldType NewName;
namespace fs = std::filesystem; // 헤더에서는 금지
```

## 이름 규칙의 예외 (Exceptions to Naming Rules)
기존 C 또는 C++ 엔티티와 유사한 것을 명명하는 경우 기존 명명 규칙 체계를 따를 수 있습니다.
- C의 `open()`과 유사한 경우 `my_open()`와 같이 사용 가능
- STL 컨테이너와 유사하다면 `my_container_type`과 같이 사용 가능.