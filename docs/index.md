# Verse Language Documentation

이 문서에서는 Verse 프로그래밍 언어, 그 철학 및 핵심 개념에 대해 자세히 살펴봅니다.

Verse는 Epic Games에서 개발한 다중 패러다임 프로그래밍 언어로, 함수형, 논리형, 명령형 전통을 기반으로 메타버스 경험을 구축하기 위한 일관된 시스템을 만듭니다.


Verse에는 세 가지 핵심 원칙이 있습니다:

- **그냥 코드 입니다** - 복잡한 개념들도 Verse 의 기본 구조로 표현됩니다.
- **한 종류의 언어로 되어 있습니다** - 하나의 언어 구문이 컴파일에도 쓰이고, 런타임에도 쓰입니다.
- **메타버스를 우선시 합니다** - 글로벌 시뮬레이션 환경을 위해 고안되었습니다.

!!! 주의
      본 문서의 내용은 Verse의 메인 개발 브랜치에 속하며,
      일부 기능은 공식 출시 전 논의를 반영하여 변경될 수 있습니다.
      몇몇 Epic 내부 기능들도 논의의 대상이 될 수 있습니다.

## 문서 구성

- [Overview](00_overview.md) - Introduction to Verse philosophy and features
- [Expressions](01_expressions.md) - Everything is an expression paradigm
- [Primitives](02_primitives.md) - Integers, floats, rationals, logic, strings, and special types
- [Containers](03_containers.md) - Optionals, tuples, arrays, maps, and weak maps
- [Operators](04_operators.md) - Arithmetic, comparison, logical, and assignment operators with precedence
- [Mutability](05_mutability.md) - Mutable variables, references, and state management
- [Functions](06_functions.md) - Open-world vs closed-world functions, parameters, and return values
- [Control Flow](07_control.md) - If/else, loops, code blocks, and comments
- [Failure System](08_failure.md) - First-class failure, failable expressions, and speculative execution
- [Structs & Enums](09_structs_enums.md) - Value types and fixed sets of named values
- [Classes & Interfaces](10_classes_interfaces.md) - Object-oriented programming with inheritance and contracts
- [Type System](11_types.md) - Types as functions and type checking
- [Access Specifiers](12_access.md) - Public, private, and protected visibility
- [Effects](13_effects.md) - Effect families, specifiers, and capability declarations
- [Concurrency](14_concurrency.md) - Structured concurrency with sync, race, rush, branch, and spawn
- [Live Variables](15_live_variables.md) - Reactive values that automatically update
- [Modules & Paths](16_modules.md) - Code organization and the global namespace
- [Persistable Types](17_persistable.md) - Types that can be saved and loaded
- [Code Evolution](18_evolution.md) - Versioning and backward compatibility
