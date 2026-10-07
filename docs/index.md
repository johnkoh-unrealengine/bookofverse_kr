# Verse Language Documentation

이 문서에서는 Verse 프로그래밍 언어와 그 철학 및 핵심 개념에 대해 자세히 살펴봅니다.

Verse는 Epic Games에서 개발한 다중 패러다임 프로그래밍 언어로, 함수형, 논리형, 명령형 전통을 기반으로 메타버스 경험을 구축하기 위한 일관된 시스템을 만듭니다.

Verse 는 세가지 기본 원칙을 갖습니다 :

- **그냥 코드 입니다** - 복잡한 개념들도 기본적인 Verse construct 로 표현됩니다
- **하나의 언어입니다** - 컴파일 타임과 런타임에 같은 constructs 가 사용됩니다
- **메타버스를 우선시 합니다** - 전 지구적 단위의 시뮬레이션 환경에 적합하도록 설계되었습니다

!!! note
    본 문서는 Verse의 메인 개발 브랜치에 대한 내용이며,
    일부 기능은 공식 출시 전에 논의를 거쳐 변경될 수 있습니다.
    여기에는 Epic 내부 기능을 원인으로 한 논의도 포함될 수 있습니다.

## Documentation Sections

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
