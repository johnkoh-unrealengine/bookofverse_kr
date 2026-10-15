# 기본 데이터 자료형

Verse 의 기본 자료형은 숫자 자료형인 `int`, `float`, `rational` ; boolean 값인 `logic` ; 문자를 위한 `char`, `char32`, `string` 이 있습니다. 그리고 계층 구조에서 특별한 역할을 수행하는 두 자료형이 있습니다. 하나는 모든 자료형의 supertype 인 `any` 이고, 다른 하나는 주어지는 값이 무엇이든 다 폐기하는 `void` 입니다.

## Intrinsics[^Intrinsics]

평범한 Verse 코드로 보이는 것들 중 몇몇은 사실 Verse 로 쓰여진게 아닙니다. *Intrinsic functions* 는 컴파일러가 합성하는 operations 입니다. operators 나 container indexing, 또는 `Abs`, `Floor`, `Ceil` 과 같은 function 들이 여기 속합니다. 이것들은 runtime internals[^RuntimeInternals] 에 접근해야 하기 때문에 Verse 로 쓰일 수 없습니다. 그리고 컴파일러가 이들 각각을 이름으로 인식하기 때문에 call 을 최적화 할 수 있습니다. [Runtime API Reference](api/verse_runtime_api.html) 에서 이들의 목록이 나열된 것을 보실 수 있고, 엔진이 C++ 로 구현하는 runtime 구성요소도 함께 보실 수 있습니다.

대부분의 intrinsic functions 는 *first-class value[^FirstClassValue] 로 참조할 수 없습니다*. 이는 그것들을 직접적으로 call 할 수는 있지만, variables 내부에 저장하거나 function 의 arguments 로서 전달될 수는 없다는 것을 의미합니다 :

<!--versetest
assert_semantic_error(3502):
    F := Abs
-->
<!-- 01 -->
```verse
Result := Abs(-42)  # 42 를 return 합니다

# 유효하지 않음 : Calling 없이 참조할 수 없습니다
# F := Abs  # 오류
# 유효하지 않음 : Parameter 로 전달할 수 없습니다
# ApplyFunction(Abs, -42)  # 오류
```

이와 같은 제한이 있는 이유는 intrinsics 가 종종 표준 function 모델에 맞지 않는 특별한 calling 규칙이나 최적화를 필요로 하기 때문입니다. 만약 intrinsic functions 를 전달해야 할 필요가 있는 경우에는 일반적인 function 이나 nested function[^NestedFunction] 으로 그것을 감싸야 합니다.

## Integers[^Integers]

`int` 자료형은 integer, 즉 소수점 자리가 없는 값을 나타냅니다. 하나의 `int` 에는 양수, 음수, 0 을 담을 수 있습니다. 런타임 중의 integers 는 임의의 정밀도를 가지며, 어떠한 고정 크기를 초과해서 커질 수 있습니다. 하지만, integer *literals* 는 반드시 64-bit 의 부호화된 범위(`-9,223,372,036,854,775,808` to `9,223,372,036,854,775,807`) 내에 들어와야 합니다. 64-bit 의 범위를 넘는 integers 는 제한된 지원만 받을 수 있습니다. (예를 들어, 이들은 string interpolation[^StringInterpolation] 에 사용될 수 없고, persisted[^Persisted] 로서도 사용될 수 없습니다)

코드 내에서는 literals[^Literals] 로 `int` 값을 쓸 수 있습니다.

<!--versetest-->
<!-- 02 -->
```verse
A :int= -42                    # 평범한 숫자
#B := 42424242424242424242424242424242424242424242424242 # 딱 봐도 낯선 무서운 숫자...
                               # ... 계산될 수는 있지만 literals 로 쓰일 수는 없습니다

AnswerToTheQuestion :int= 42   # 변하지 않는 variable
CoinsPerQuiver :int= 100       # 화살통 한 통의 가격은 동전 100 개 입니다
ArrowsPerQuiver :int= 15       # 화살통 한 통에는 화실이 15 개 들어 있습니다

# 변할 수 있는 variables (var 와 set 에 대한 자세한 내용은 Mutability 챕터를 확인하세요)
var Coins :int= 225           # 이 플레이어는 지금 225 개의 동전을 가지고 있습니다
var Arrows :int= 3            # 이 플레이어는 지금 3 개의 화살을 가지고 있습니다
var TotalPurchases :int= 0    # 총 구매량을 추적합니다
```

integers 로는 네 종류의 기본 계산을 할 수 있습니다. `+` 는 덧셈을, `-` 는 뺄셈을, `*` 는 곱셈을, `/` 는 나눗셈을 합니다.

<!--versetest
MyInt:int=10
MyHugeInt:int=1010100101
-->
<!-- 03 -->
```verse
var C :int= (-MyInt + MyHugeInt - 2) * 3   # 산수
set C += 1                                 # set C = C + 1 와 같습니다
set C *= 2                                 # set C = C * 2 와 같습니다
```

integers 에서, `/` operator 는 실패할 수 있습니다. 그리고 성공한 경우에 그 결과는 `rational` 자료형을 갖습니다.

### Overflow[^Overflow] 와 VM[^VM] 차이점

`int` 는 VM 마다 64-bit 범위의 끝 부분에서 작동하는 방식이 다릅니다 :

- **VerseVM[^VerseVM]** 은 임의의 정확도를 사용합니다. 64 bit 를 초과하는 산수는 단지 더 큰 값을 생성합니다.
- **BPVM[^BPVM]** 은 64-bit integers 를 쓰고, overflow 발생 시 런타임 에러(`MathIntrinsicCallFailure`)를 발생 시킵니다. 이 때, 대상이 되는 overflow 는 `+=` 나 `*=` 에 의해 발생한 것 뿐만 아니라, 범위 내 최소 값에 대한 unary negation[^UnaryNegation] 에 발생한 경우, bit 단위 연산을 수행하는 intrinsics 에 의해 발생한 경우를 가리지 않습니다.

따라서, 두 VM 환경에서 동일하게 작동되어야 하는 코드의 경우, 반드시 부호화된 64-bit 의 범위 내에 있어야 합니다.

## Rationals[^Rationals]

`rational` 자료형은 integers 의 비율로서의 분수를 말합니다. `int` 나 `float` 와는 달리, `rational` literal 은 직접적으로는 작성할 수 없습니다. 대신, operator `/` 를 사용하는 integer 의 나눗셈이 rationals 를 만듭니다.

<!--versetest-->
<!-- 04 -->
```verse
X := 7 / 3    # X 는 rational 자료형을 갖게 되고, 정확히 7÷3 을 표현합니다
```

Rationals 는 소수점 숫자에서의 정밀 손실이 없는 *정확한 산수* 를 제공합니다. 따라서 정확한 분수 계산이 필요한 게임 로직에 사용하기 좋습니다. (자원 분배, 턴제 시스템, 확률 계산 등)

`/` 로 나뉜 integer 는 rational 값을 만듭니다. 0 으로 나뉘면 실패합니다 :

<!--versetest-->
<!-- 05 -->
```verse
Half := 5 / 2           # rational: 정확히 5/2
Third := 10 / 3         # rational: 정확히 10/3
Quarter := 1 / 4        # rational: 정확히 1/4

if (not (1 / 0)):
    # 0으로 나뉘면 실패합니다
```

Rationals 는 동등성 비교시 자동적으로 lowest terms[^LowestTerms] 로 변환됩니다 :

<!--versetest-->
<!-- 06 -->
```verse
# 아래의 모든 값은 같습니다 - 즉, 5/2 로 변환됩니다
(5 / 2) = (10 / 4)      # 참
(5 / 2) = (15 / 6)      # 참
(10 / 4) = (15 / 6)     # 참
```

이 정규화는 각각의 값이 어떻게 형성 되었는지와 무관하게, 수학적으로 동등한 rationals 를 비교하면 그 값들이 서로 동등한 값으로써 비교 되도록 보장합니다.

음의 기호는 분자에 맞춰지도록 정규화 됩니다.

<!--versetest-->
<!-- 07 -->
```verse
(1 / -3) = (-1 / 3)     # 참: 음의 기호가 분자로 옮겨 갑니다
(-1 / -3) = (1 / 3)     # 참: 분자와 분모 모두에 음의 기호가 있으면 양의 기호로 바뀝니다
```

This canonical form simplifies equality checking and ensures
consistent behavior.

An important property: *`int` is a subtype of `rational`*. This means
any integer can be used where a rational is expected:

<!--versetest-->
<!-- 08 -->
```verse
ProcessRational(X:rational):rational = X

# Can pass integers directly
ProcessRational(5) = 5/1     # 5 is implicitly 5/1 (rational)
ProcessRational(0) = 0/1     # 0 is implicitly 0/1 (rational)
```

However, you *cannot* return a rational where an int is expected—that
would be a narrowing conversion:

<!--versetest
assert_semantic_error(3510):
    BadFunction(X:rational):int = X
<#
-->
<!-- 09 -->
```verse
BadFunction(X:rational):int = X  # Error
```
<!-- #> -->

Whole number rationals equal their integer equivalents:

<!--versetest-->
<!-- 10 -->
```verse
(2 / 1) = 2             # true
2 = (2 / 1)             # true
(4 / 2) = 2             # true: 4/2 reduces to 2/1, equals 2
(9 / 3) = 3             # true: 9/3 reduces to 3/1, equals 3
```

This enables seamless mixing of integer and rational values in calculations.

Two functions convert rationals to integers: `Floor` rounds toward
negative infinity, and `Ceil` toward positive infinity.

<!--versetest-->
<!-- 11 -->
```verse
# Positive rationals
Floor(5 / 2)= 2         # 2.5 → 2 (down)
Ceil(5 / 2) = 3         # 2.5 → 3 (up)

# Negative rationals - note direction!
Floor((-5) / 2) = -3    # -2.5 → -3 (toward negative infinity)
Ceil((-5) / 2) = -2     # -2.5 → -2 (toward positive infinity)

# With negative denominator
Floor(5 / -2) = -3      # Same as (-5)/2
Ceil(5 / -2) = -2       # Same as (-5)/2

# Both negative
Floor((-5) / -2) = 2    # 2.5 → 2
Ceil((-5) / -2) = 3     # 2.5 → 3
```

Rounding toward negative infinity is *not* truncation: the two agree
on positive values and part company on negative ones. On a rational
argument neither function can fail; the `float` overloads are
`<decides>`.

When you apply `Floor` or `Ceil` to an integer-valued rational (one
that reduces to a whole number), it returns that integer:

<!--versetest-->
<!-- 12 -->
```verse
# Integer-valued rationals
PositiveRational:rational = 5
NegativeRational:rational = -5

Floor(PositiveRational) = 5   # Returns the integer 5
Ceil(PositiveRational) = 5    # Returns the integer 5
Floor(NegativeRational) = -5  # Returns the integer -5
Ceil(NegativeRational) = -5   # Returns the integer -5

# Also works for rationals that reduce to integers
Floor(10 / 2) = 5             # 10/2 = 5/1, returns 5
Ceil(10 / 2) = 5              # 10/2 = 5/1, returns 5
```

Rationals can be used as parameter and return types:

<!--versetest-->
<!-- 13 -->
```verse
# Function returning rational
Half(X:int)<computes><decides>:rational = X / 2

# Use the result
if (Result := Half[7]):
    Floor(Result) = 3   # 7/2 = 3.5, Floor gives 3
    Ceil(Result) = 4    # 7/2 = 3.5, Ceil gives 4
```


Because `int` is a subtype of `rational`, you *cannot* overload based
solely on these types:

<!--versetest
assert_semantic_error(3532):
    ProcessValue(X:int):void = {}
    ProcessValue(X:rational):void = {}
<#
-->
<!-- 14 -->
```verse
ProcessValue(X:int):void = {}
ProcessValue(X:rational):void = {}  # Error!
```
<!-- #> -->

The compiler sees `int` as more specific than `rational`, so the
signatures would be ambiguous.

Rationals excel at resource distribution and fairness calculations:

<!--versetest-->
<!-- 15 -->
```verse
# Fair resource distribution
DistributeResources(TotalGold:int, NumPlayers:int)<decides>:int =
    GoldPerPlayer := TotalGold / NumPlayers
    Floor(GoldPerPlayer)  # Converts to whole gold pieces (can be 0)

# To fail when there's insufficient gold, check > 0
DistributeResourcesOrFail(TotalGold:int, NumPlayers:int)<decides>:int =
    GoldPerPlayer := TotalGold / NumPlayers
    Floor(GoldPerPlayer) > 0  # Fails if each player gets 0

# Item affordability calculation
Coins:int = 225
CoinsPerQuiver:int = 100
ArrowsPerQuiver:int = 15

if (NumberOfQuivers := Floor(Coins / CoinsPerQuiver)):
    TotalArrows:int = NumberOfQuivers * ArrowsPerQuiver
    # Player can afford 2 quivers = 30 arrows
```

## Floats

The `float` type represents all non-integer numerical values. It can
hold large values and precise fractions, such as `1.0`, `-50.5`, and
`3.14159`. A float is an IEEE 64-bit float, which means it can contain
a positive or negative number that has a decimal point in the range
`[-2^1024 + 1, … , 0, … , 2^1024 - 1]`, or has the value `NaN` (Not a
Number). The implementation differs from the IEEE standard in the
following ways:

- There is only one `NaN` value.
- `NaN` is equal to itself.
- Every number is equal to itself.
- `0` cannot be negative.

You can include float values within your code as literals:

<!--versetest-->
<!-- 16 -->
```verse
A:float = 1.0
B := 2.14
MaxHealth : float = 100.0

var C:float = A + B
C = 3.14              # succeeds
set C -= 3.14
C = 0.0               # succeeds
# C = 0              # compile error; 0 is not a `float` literal
```

You can use the four basic math operations with floats: `+` for
addition, `-` for subtraction, `*` for multiplication, and `/` for
division. There are also combined operators for doing the basic math
operations (addition, subtraction, multiplication, and division), and
updating the value of a variable:

<!--versetest-->
<!-- 17 -->
```verse
var CurrentHealth : float = 100.0
set CurrentHealth /= 2.0    # Halves the value of CurrentHealth
set CurrentHealth += 10.0   # Adds 10 to CurrentHealth
set CurrentHealth *= 1.5    # Multiplies CurrentHealth by 1.5
```

### Special Float Values

Float operations follow IEEE 754 semantics, which include special
values for infinity and Not-a-Number (NaN):

<!--versetest-->
<!-- 18 -->
```verse
# Infinity from division by zero
PosInf := 1.0 / 0.0
NegInf := -1.0 / 0.0

# NaN from invalid operations
NaNFromZeroDiv := 0.0 / 0.0
NaNFromSqrt := Sqrt(-1.0)
NaNFromInfDiv := PosInf / PosInf

# NaN propagates through operations
Result := NaNFromZeroDiv + 100.0  # Result is NaN
Product := NaNFromSqrt * 2.0      # Product is NaN
```

Division by zero produces `Inf` or `-Inf` rather than failing. Invalid
operations like `0.0 / 0.0`, `Sqrt(-1.0)`, or `Inf - Inf` produce NaN
values. Arithmetic operations with NaN propagate the NaN to the
result. Unlike standard IEEE 754, Verse's NaN equals itself when
compared.

To convert an `int` to a `float`, multiply it by `1.0`: `MyFloat:=MyInt*1.0`.

## Mathematical Functions

The mathematical functions are intrinsics. The [Runtime API
Reference](api/verse_runtime_api.html) lists every one of them with its
signature, its effects, and how it behaves at the edges of its domain; this
section pins down the behaviour that most often catches people out.

`Abs`, `Min` and `Max` are defined for both `int` and `float`. On floats they
follow Verse's rule that `NaN` is a value like any other rather than something
that poisons a comparison, so it propagates out of `Min` and `Max` instead of
being ignored:

<!--versetest-->
<!-- 19 -->
```verse
Abs(5) = 5
Abs(-5) = 5
Abs(3.14) = 3.14

# NaN propagates through comparison
Max(NaN, 5.0) = NaN
Min(NaN, 5.0) = NaN

# Infinity is ordered as you would expect
Max(Inf, 100.0) = Inf
Min(-Inf, 100.0) = -Inf
```

`Ceil`, `Floor`, `Round` and `Int` convert a `float` to an `int`, and all four
can fail: a non-finite argument has no integer to round to, which is why they
are written with square brackets. `Round` breaks ties to even rather than away
from zero, so a long run of rounded values does not drift upwards:

<!--versetest-->
<!-- 20 -->
```verse
Round[1.5] = 2     # tie, rounds to even 2
Round[2.5] = 2     # tie, rounds to even 2
Round[-1.5] = -2   # tie, rounds to even -2
Round[0.5] = 0     # tie, rounds to even 0

Round[1.4] = 1     # no tie, rounds down
Round[1.6] = 2     # no tie, rounds up
```

The rest of the floating-point functions never fail. Where a mathematician
would say the answer does not exist, they return `NaN` or an infinity and let
it flow onwards:

<!--versetest-->
<!-- 21 -->
```verse
Sqrt(4.0) = 2.0
Sqrt(-1.0) = NaN   # negative inputs have no real root
Sqrt(Inf) = Inf

Pow(2.0, 3.0) = 8.0     # Pow(Base, Exponent)
Pow(4.0, 0.5) = 2.0     # square root
Pow(2.0, -1.0) = 0.5    # reciprocal
Pow(0.0, 0.0) = 1.0     # by convention
Pow(NaN, 0.0) = 1.0     # a zero exponent always gives 1
Pow(1.0, NaN) = 1.0     # 1 to any power is 1

Exp(0.0) = 1.0
Exp(-Inf) = 0.0
Ln(1.0) = 0.0
Ln(Exp(1.0)) = 1.0      # ln(e) = 1
Ln(0.0) = -Inf
Ln(-1.0) = NaN

Log(10.0, 100.0) = 2.0  # log₁₀(100); the base comes first
Log(2.0, 8.0) = 3.0     # log₂(8)
```

The trigonometric functions work in radians, and `PiFloat` is the nearest
`float` to π rather than π itself. That distinction is visible: angles which
mathematically give exactly zero only come close.

<!--versetest-->
<!-- 22 -->
```verse
Sin(0.0) = 0.0
Sin(PiFloat / 2.0) = 1.0
Cos(0.0) = 1.0
Cos(PiFloat) = -1.0

Sin(NaN) = NaN
Sin(Inf) = NaN          # no meaningful angle

# PiFloat is not π, so these are near zero rather than zero
Sin(PiFloat) > 0.0 and Sin(PiFloat) < 0.000001
Cos(PiFloat / 2.0) > 0.0 and Cos(PiFloat / 2.0) < 0.000001

# The inverses answer an angle: ArcSin and ArcTan in [-π/2, π/2],
# ArcCos in [0, π]
ArcSin(1.0) = PiFloat / 2.0
ArcCos(-1.0) = PiFloat
ArcTan(1.0) = PiFloat / 4.0
```

Beware that `ArcSin` and `ArcCos` do not fail outside `[-1, 1]`, and do not
return `NaN` either: they clamp, so `ArcSin(2.0)` quietly answers π/2. The
engine leaves that behaviour unspecified, so range-check the argument yourself
when a bad input needs to be visible.

`IsFinite` is the guard for all of this. It fails for `Inf`, `-Inf` and `NaN`,
and otherwise succeeds with the value itself, which lets it sit in the middle
of a calculation:

<!--versetest

assert:
    (5.0).IsFinite[]
    (-100.0).IsFinite[]
    not (Inf).IsFinite[]
    not (-Inf).IsFinite[]
    not (NaN).IsFinite[]
    (15.16).IsFinite[] = 15.16
    SafeDivide[10.0, 4.0] = 2.5
    not SafeDivide[10.0, 0.0]
SafeDivide(X:float, Y:float)<computes><decides>:float =
    X.IsFinite[] and Y.IsFinite[]
    Result := X / Y
    Result.IsFinite[]
    Result
<#
-->
<!-- 23 -->
```verse
(5.0).IsFinite[]      # succeeds
(-100.0).IsFinite[]   # succeeds

(Inf).IsFinite[]  # fails
(NaN).IsFinite[]  # fails

# Succeeds with the value, so it can be used inline
(15.16).IsFinite[] = 15.16

# Validation: fails rather than yielding Inf or NaN
SafeDivide(X:float, Y:float)<computes><decides>:float =
    X.IsFinite[] and Y.IsFinite[]
    Result := X / Y
    Result.IsFinite[]
    Result
```
<!-- #> -->

Finally, the named constants:

<!--NoCompile-->
<!-- 24 -->
```verse
PiFloat # 3.14159265358979323846...
Inf     # Positive infinity
-Inf    # Negative infinity (negation of Inf)
NaN     # Not a Number
```
## Booleans

The `logic` type represents the Boolean values `true` and `false`.

<!--versetest-->
<!-- 25 -->
```verse
A:logic = true
B := false

# A = B          # fails
A?                # succeeds
# B?             # fails

true?             # succeeds
# false?         # fails
```

The `logic` type only supports query operations and comparison
operations.  Query expressions use the query operator `?` to check if
a logic value is true and fail if the logic value is `false`.  For
comparison operations, use the failable operator `=` to test if two
logic values are the same, and `<>` to test for inequality.

Many programming languages find it idiomatic to use a type like
`logic` to signal the success or failure of an operation. In Verse, we
use success and failure instead for that purpose, whenever
possible. The conditional only executes the `then` branch if the guard
succeeds:

<!--versetest
ShowTargetLockedIcon():void={}
TargetLocked:?int = option{42}
-->
<!-- 26 -->
```verse
if (TargetLocked?):
    ShowTargetLockedIcon()
```

To convert an expression that has the `<decides>` effect to `true` on
success or `false` on failure, use `logic{ exp }`:

<!--versetest
F()<decides>:void=
    GotIt := logic{5 > 3}
    GotIt?
<#
-->
<!-- 27 -->
```verse
GotIt := logic{5 > 3}    # the comparison succeeds, so GotIt is true
GotIt?                   # so this succeeds
GotIt = false            # and this fails
not GotIt?               # and this fails too
```
<!-- #> -->

## Characters and Strings

Text is represented in terms of characters and strings.  A `char` is a
single **UTF-8 code unit** (not a full Unicode code point). A string
is therefore an array of characters, written as `[]char`. For
convenience, Verse provides the type alias `string` for `[]char`:

<!--versetest-->
<!-- 28 -->
```verse
MyName :string = "Joseph"
MyAlterEgo := "José"
```

Verse uses UTF-8 as the character encoding scheme. Each UTF-8 code unit
is one byte. A Unicode code point may require between one and four
code units. Code points with lower values use fewer bytes, while
higher values require more.

For example:

- `"a"` requires one byte (`{0o61}`),
- `"á"` requires two bytes (`{0oC3}{0oA1}`),
- `"🐈"` (cat emoji) requires four bytes (`{0u1f408}`).

Thus, strings are sequences of code units, not necessarily sequences
of Unicode characters in the abstract sense.

Because strings are arrays of `char`, you can index into them with
`[]`. Indexing has the `<decides>` effect: it succeeds when the index
is valid and fails otherwise.

<!--versetest
MyName:string="J"
-->
<!-- 29 -->
```verse
TheLetterJ := MyName[0]     # succeeds
TheLetterJ = 'J'            # succeeds
# MyName[100]               # fails
```

The length of a string is the number of UTF-8 code units it contains,
accessed via `.Length`. Note that this is *not the same as the number
of Unicode characters*:

<!--versetest-->
<!-- 30 -->
```verse
"José".Length = 5           # succeeds; 5 UTF-8 code units
"Jose".Length = 4           # succeeds; 4 UTF-8 code units
```

Because `string` is just `[]char`, strings declared as `var` can be mutated:

<!--versetest-->
<!-- 31 -->
```verse
var OuterSpaceFriend :string = "Glorblex"
set OuterSpaceFriend[0] = 'F'
```

Strings can be concatenated using the `+` operator:

<!--versetest
MyName:string="Joe"
MyAlterEgo:string="Jak"
-->
<!-- 32 -->
```verse
MyAttemptAtFormatting := "My name is " + MyName + " but my alter ego is " + MyAlterEgo + "."
```

Verse also supports string interpolation for more readable formatting:

<!--versetest
MyName:string="3"
MyAlterEgo:string="asdsa"
-->
<!-- 33 -->
```verse
Formatting := "My name is {MyName} but my alter ego is {MyAlterEgo}."
```

Interpolation works for any value that has a `ToString()` function in scope.

Write literal characters with single quotes. The type depends on
whether the character falls within the ASCII range (`U+0000`–`U+007F`)
or not:

- `'e'` has type `char`,
- `'é'` has type `char32`.

<!--versetest-->
<!-- 34 -->
```verse
A :char = 'e'                       # ok
B :char32 = 'é'                     # ok
# C :char = 'é'                     # error: type of 'é' is char32
# D :char32 = 'e'                   # error: type of 'e' is char
```

Character literals can also be written using numeric escape sequences:

<!--versetest-->
<!-- 35 -->
```verse
E :char = 0o65                      # ok; same as 'e'
F :char32 = 0u00E9                  # ok; same as 'é'
```

- `char` represents a single UTF-8 code unit (one byte, `0oXX`).
- `char32` represents a full Unicode code point (`0uXXXXX`).

Hex notation:

- `0oXX` for `char`: two hex digits (0o00 to 0off)
- `0uXXXXX` for `char32`: up to six hex digits (0u00000 to 0u10ffff)

Unlike some languages, Verse does not allow implicit conversion between characters and integers.

**Character escape sequences** work in both character and string literals:

| Escape | Meaning | Codepoint |
|--------|---------|-----------|
| `\t` | Tab | U+0009 |
| `\n` | Newline | U+000A |
| `\r` | Carriage return | U+000D |
| `\"` | Double quote | U+0022 |
| `\'` | Single quote | U+0027 |
| `\\` | Backslash | U+005C |
| `\{` | Left brace | U+007B |
| `\}` | Right brace | U+007D |
| `\<` | Less than | U+003C |
| `\>` | Greater than | U+003E |
| `\&` | Ampersand | U+0026 |
| `\#` | Hash/pound | U+0023 |
| `\~` | Tilde | U+007E |

Examples:

<!--versetest-->
<!-- 36 -->
```verse
Tab := '\t'
Newline := '\n'
Quote := '\"'
Brace := '\{'
```

Strings can be compared using the failable operators `=` (equality)
and `<>` (inequality). Comparison is done by code point, and is case
sensitive.  Equality depends on exact code unit sequences, not visual
appearance. Unicode allows multiple encodings for the same abstract
character. For example, `"é"` may appear as the single code point
`{0u00E9}`, or as the two-code-point sequence `"e"` (`{0u0065}`) plus
a combining accent (`{0u0301}`). These two strings look the same, but
they are not equal in Verse.

Checking whether a player has selected the correct item:

<!--versetest-->
<!-- 37 -->
```verse
ExpectedItemInternalName :string = "RedPotion"
SelectedItemInternalName :string = "BluePotion"

if (SelectedItemInternalName = ExpectedItemInternalName):
    true
else:
    false
```

Padding a timer with leading zeros:

<!--versetest-->
<!-- 38 -->
```verse
SecondsLeft :int = 30
SecondsString :string = ToString(SecondsLeft)    # convert int to string

var Combined :string = "Time Remaining: "
if (SecondsString.Length > 2):
    set Combined += "99"               # clamp to maximum
else if (SecondsString.Length < 2):
    set Combined += "0{SecondsString}" # pad with zero
else:
    set Combined += SecondsString
```

String interpolation supports complex expressions, not just simple variables:

<!--versetest
Format(D:float, ?Decimals:int):string=""
-->
<!-- 39 -->
```verse
# Expression interpolation
Age := 30
Message := "Next year: {Age + 1}"

# Function calls with named arguments
Distance := 5.5
Formatted := "Distance: {Format(Distance, ?Decimals:=2)}"
```

Strings can span multiple lines using interpolation braces for continuation:

<!--versetest-->
<!-- 40 -->
```verse
LongMessage := "This is a multi-line {
}string that continues across {
}multiple lines."

# Attention to whitespace:
AnotherMessage := "This is another {
}  multi-line message with     {
    # This comment is ignored
}    many spaces."
```

Empty interpolants `{}` are ignored, which is useful for line
continuation without adding content.

Since `string` is `[]char`, strings and character arrays can be compared:

<!--versetest-->
<!-- 41 -->
```verse
"abc" = array{'a', 'b', 'c'}    # Succeeds
"" = array{}                     # Succeeds - empty string equals empty array
```

Block comments within strings are removed during parsing:

<!--versetest-->
<!-- 42 -->
```verse
Text := "abc<#this comment is removed#>def"    # Same as "abcdef"
```

### ToString()

The `ToString()` function converts values to their string
representations. It is overloaded, with a version for `int`, for
`float`, for `char`, and one for `string` that hands back what it was
given.

String interpolation implicitly calls `ToString()` on embedded values:

<!--versetest-->
<!-- 43 -->
```verse
Age := 25
Score := 98.5

# These are equivalent:
Message1 := "Age: " + ToString(Age) + ", Score: " + ToString(Score)
Message2 := "Age: {Age}, Score: {Score}"
# Both produce: "Age: 25, Score: 98.5"
```

This makes `ToString()` essential for formatting output, even when you
do not call it directly.

`ToString()` only works on primitive types. User-defined classes and
structs do not have automatic string conversion.

### ToDiagnostic()

The `ToDiagnostic()` function takes a value of any type and produces a
`diagnostic`, useful for debugging and logging. Where `ToString()` is
overloaded per type and refuses anything it has no overload for,
`ToDiagnostic()` accepts everything and may report more detail:

<!--versetest
SomeValue:int=1
-->
<!-- 44 -->
```verse
Diagnosis := ToDiagnostic(SomeValue)
```

`ToDiagnostic()` is primarily used for debugging output rather than
user-facing strings. The exact format it produces may vary between VM
implementations and is not guaranteed to be stable across versions.

## Type type

The `type` type is a *metatype* - a type whose values are themselves
types. Every Verse type can be used as a value of type `type`. This
enables powerful generic programming through parametric functions,
where types are parameters that can be passed around and constrained.

You can create variables and parameters that hold type values:

<!--versetest-->
<!-- 45 -->
```verse
# A name can hold a type value
IntType:type = int

# A function can take one as a parameter and use it in its own signature
CreateDefault(t:type):?t = false

X:?int = CreateDefault(int)
Y:?string = CreateDefault(string)
not X? and not Y?
```

Any type at all can be named this way, using `:=` rather than `:type =`
so that the name becomes usable in the positions where a type is expected:

<!--versetest-->
<!-- 46 -->
```verse
# User-defined types
my_class := class {}
my_struct := struct {Value:int}

# Collections
ArrayType := []int
MapType := [string]int
TupleType := tuple(int, string)
OptionType := ?int

# Functions
FuncType := int->string

# Metatypes
SubtypeValue := subtype(my_class)
TypeLiteralValue := type{_(:int):string}

# The names then stand in for the types they were given
Pair:TupleType = (7, "seven")
Pair(0) = 7
Which:SubtypeValue = my_class
```

This universality makes `type` the foundation for Verse's generic
programming - any type can be abstracted over.

### Type Parameters

The most common use of `type` is in **where clauses** to create
parametric (generic) functions:

<!--versetest-->
<!-- 47 -->
```verse
# Identity function - works with any type
Identity(X:t where t:type):t = X

# Usage - type parameter inferred
Identity(42)        # t = int
Identity("hello")   # t = string
Identity(true)      # t = logic
```

The `where t:type` constraint means "`t` can be any Verse type." The
type system infers `t` from the argument and ensures type safety
throughout the function.

While `where t:type` accepts any type, you can use more specific
constraints like `subtype` to limit which types are valid:

<!-- 48 -->
```verse
# Only accepts types that are subtypes of comparable
Sort(Items:[]t where t:subtype(comparable)):[]t =
    Items  # A real implementation can compare elements, because t is comparable
```

For comprehensive documentation on parametric functions, see the
Functions chapter.

### Type as First-Class Values

Unlike many languages where types only exist at compile time, Verse
treats types as *first-class values* that can be computed, stored, and
manipulated:

<!--versetest-->
<!-- 49 -->
```verse
# Function that returns a type value
GetTypeForSize(Size:int):type =
    if (Size <= 8):
        int
    else:
        string

# Store type in data structure
TypeRegistry:[string]type = map{
    "Integer" => int,
    "Text" => string,
    "Flag" => logic
}
```

Types can be passed between functions:

<!-- 50 -->
```verse
# Helper function that takes a type parameter
CreateArray(ElementType:type, Size:int):[]ElementType =
    array{}

# Function that uses the helper
MakeIntArray():[]int =
    CreateArray(int, 10)
```

### Returning Options of Type Parameters

A common pattern is to have functions return `?t` where `t` is a type
parameter, allowing the function to work with any type while
potentially failing:

<!--versetest
MaybeValue(Value:t, Condition:logic where t:type):?t =
    if (Condition?) then option{Value} else false

assert:
    X:?int = MaybeValue(5, false)
    Y:?float = MaybeValue(3.14, true)
<#
-->
<!-- 51 -->
```verse
# return type `t` must be the same type as the `Value` param type
MaybeValue(Value:t, Condition:logic where t:type):?t =
    if (Condition?) then option{Value} else false

# Usage
X:?int = MaybeValue(5, false)  # Returns false as ?int
Y:?float = MaybeValue(3.14, true)  # Returns option{3.14} as ?float
```
<!-- #> -->


<!--versetest
MaybeValueExplicit(T:type, Value:t, Condition:logic where t:subtype(T)):?T =
    if (Condition?):
        option{Value}
    else:
        false

assert:
    X:?int = MaybeValueExplicit(int, 5, false)
    Y:?float = MaybeValueExplicit(float, 3.14, true)
assert_semantic_error(3509):
    Maybe818(T:type, Value:t, Condition:logic where t:subtype(T)):?T =
        if (Condition?) then option{Value} else false
    G818():void =
        Z:?int = Maybe818(int, 3.14, true)
<#
-->
<!-- 52 -->
```verse
# Alternative: explicitly pass the type parameter
MaybeValueExplicit(T:type, Value:t, Condition:logic where t:subtype(T)):?T =
    if (Condition?):
        option{Value}
    else:
        false

# Usage
X:?int = MaybeValueExplicit(int, 5, false)  # Returns false as ?int
Y:?float = MaybeValueExplicit(float, 3.14, true)  # Returns option{3.14} as ?float
# Z:?int = MaybeValueExplicit(int, 3.14, true) # ERROR: float not subtype of int
```
<!-- #> -->

This pattern is particularly useful for generic containers and factory
functions that may or may not be able to produce a value.

### Type Constraints

The `type` constraint in where clauses is the most permissive - it
accepts any Verse type. For more specific requirements, Verse provides
additional constraints:

<!--versetest-->
<!-- 53 -->
```verse
# Most permissive: any type
Generic(X:t where t:type):t = X

# More specific: must be subtype of comparable
RequiresComparison(X:t where t:subtype(comparable))<decides>:void =
    X = X  # Can use = because t is comparable

# Even more specific: must be exact subtype
RequiresExactType(X:t, Y:u where t:type, u:subtype(t)):t =
    X  # Y is guaranteed to be compatible with t
```

The type system enforces these constraints at compile time, preventing
invalid type usage.

### Limitations

While `type` enables powerful abstractions, there are some limitations.

There is no way to construct a value of an arbitrary type:

<!--NoCompile-->
<!-- 54 -->
```verse
# Cannot do this - no way to construct a value of arbitrary type t
MakeValue(T:type):T = ???  # What would this return for T=int? T=string?
```

There is no runtime type introspection:

<!--NoCompile-->
<!-- 55 -->
```verse
# Cannot do this - no runtime type introspection
GetFieldNames(T:type):string = ???
```

A `where` clause cannot open a parameter list, so a type parameter has to
arrive alongside a parameter that mentions it:

<!--versetest
Identity(X:t where t:type):t = X

assert:
    Identity(42)

assert_syntax_error(3100){"MakeDefault85(where t:type):t = false"}
<#
-->
<!-- 56 -->
```verse
# Type parameter must be determinable from usage
Identity(X:t where t:type):t = X

# OK: t inferred from argument
Identity(42)

# ERROR: "Expected expression or ')', got 'where'"
# MakeDefault(where t:type):t = false
```
<!-- #> -->

## Any

The `any` type is the *supertype of all types*. Every type in the
language is a subtype of `any`. Because of this, `any` itself supports
very few operations: whatever functionality `any` provides must also
be implemented by every other type. In practice, there is very little
you can do directly with values of type `any`. Still, it is important
to understand the type, because it sometimes arises when working with
code that mixes different kinds of values, or when the type checker
has no more precise type to assign.

One way `any` appears is when combining values that do not share a
more specific supertype. For example:

<!--versetest
letters := enum:
    A
    B
    C

letter := class:
    Value : char
-->
<!-- 57 -->
```verse
Main(Arg : int) : void =
    X := if (Arg > 0) then:
        letters.A
    else:
        letter{Value := 'D'}
```

In this example, the code assigns `X` either a value of type `letters` or
of type `letter`. Since these two types are unrelated, the compiler
assigns `X` the type `any`, which is their lowest common supertype.

A more useful role for `any` is as the type of a parameter that is
required syntactically but not actually used. This pattern can arise
when implementing interfaces that require a certain method signature.

<!--versetest-->
<!-- 58 -->
```verse
FirstInt(X:int, :any) : int = X
```

Here, the second parameter is ignored. Because it can be any value of
any type, it is given the type `any`.

In more general code, the same idea can be expressed using *parametric
types*, making the function flexible while still precise:

<!--versetest-->
<!-- 59 -->
```verse
First(X:t, :any where t:type) : t = X
```

This version works for any type `t`, returning a value of type `t`
while discarding the unused argument of type `any`.

## Void

The `void` type represents the absence of a meaningful result and is
used in places where no result is returned. Technically, `void` is
a function that accepts any value and evaluates to `false`.

This design allows a function with return type `void` to have a body
that evaluates to any type, while ensuring that callers cannot use
the result. The body passes the value it produces to `void`, which
discards it and returns `false`.

A function whose purpose is to perform an effect, rather than compute
a value, has return type `void`.

<!--versetest-->
<!-- 60 -->
```verse
LogMessage(Msg:string) : void =
    Print(Msg)
```

Here, `LogMessage` performs an action (printing) but does not return a
result. The `void` return type makes that explicit.

### void, true, tuple() and false

Verse has a single "there is nothing interesting here" value. It can be written
`false` or `()`, and the types `void`, `true` and `tuple()` all describe it.
These are interchangeable:

<!--versetest
assert:
    Nothing:void    = false
    Unit:tuple()    = false
    Flag:true       = false
    Unit = ()
    Unit = false
    Nothing = false
    Flag = false
<#
-->
<!-- 61 -->
```verse
Nothing:void    = false
Unit:tuple()    = false
Flag:true       = false

# All the same value
Unit = ()
Unit = false
```
<!-- #> -->

`true` is a subtype of `logic`, so the unit value flows into logic positions:

<!--versetest
assert:
    Flag:true = false
    AsLogic:logic = Flag
<#
-->
<!-- 62 -->
```verse
Flag:true = false
AsLogic:logic = Flag
```
<!-- #> -->

The empty value also stands in for empty containers. `false` is accepted
wherever an empty array or map is expected:

<!--versetest
assert:
    Empty:[]int = false
    Empty.Length = 0
    Empty = false
<#
-->
<!-- 63 -->
```verse
Empty:[]int = false
Empty.Length = 0        # 0 - false is the empty array
Empty = false
```
<!-- #> -->

and `void`/`true` interchange through optionals, arrays and maps:

<!--versetest
assert:
    MaybeVoid:?void = option{0}
    MaybeTrue:?true = MaybeVoid
    MaybeTrue?
<#
-->
<!-- 64 -->
```verse
MaybeVoid:?void = option{0}
MaybeTrue:?true = MaybeVoid     # ?void and ?true interchange
```
<!-- #> -->

#### Write the value, not the type name

`void` names a type; the unit value is `false` (or `()`). A parameter declared
literally `:void` accepts any value, so passing the type name happens to work
there. It does not work when `void` reaches the parameter through a type
parameter, so write the value:

<!--versetest
assert_valid:
    IgA(:void):int = 42
    G909a():void =
        X := IgA(void)
assert_semantic_error(3509):
    holder909(t:type) := interface:
        Method(X:t):t = X
    holder909_void := holder909(void)
    c909 := class(holder909_void) {}
    G909b():void =
        C := c909{}
        C.Method(void)
assert_valid:
    holder909b(t:type) := interface:
        Method(X:t):t = X
    holder909b_void := holder909b(void)
    c909b := class(holder909b_void) {}
    G909c():void =
        C := c909b{}
        C.Method(false)
Ignore(:void):int = 42
assert:
    Ignore(false) = 42
<#
-->
<!-- 65 -->
```verse
Ignore(:void):int = 42

Ignore(false)    # OK - the unit value
Ignore()         # OK

# A `:void` parameter accepts anything, so even the type name is taken here
Ignore(void)     # OK, but misleading - prefer `false`

# Through a type parameter bound to void, the type name is rejected:
holder(t:type) := interface:
    Method(X:t):t = X

# C.Method(void)    # ERROR
# C.Method(false)   # OK
```
<!-- #> -->

!!! warning
    That last error reads *"expects a value of type `true`, but this
    argument is an incompatible value of type `true`"*. Both sides print as
    `true`, because the type of a type expression renders the same way. If you
    see a message that appears to say a type is incompatible with itself, check
    whether you passed a type name where a value belongs.

#### Conversion is one-way

Any value converts to `void`, and that extends through containers in ordinary
parameter positions — `[]int` satisfies `[]void`. A *type parameter* bound to
`[]void` is invariant, though, so only `[]void` (or `[]true`) fits there:

<!--versetest
assert_valid:
    Main():void =
        X:void = 1
assert_valid:
    Sink(:[]void):void = {}
    Main():void =
        Y:[]int = array{1}
        Sink(Y)
assert_semantic_error(3509):
    box(t:type) := class:
        Put(:t):void = {}
    Main():void =
        Y:[]int = array{1}
        box([]void){}.Put(Y)
<#
-->
<!-- 66 -->
```verse
X:void = 1              # OK - any value converts to void

Sink(:[]void):void = {}
Y:[]int = array{1}
Sink(Y)                 # OK - []int satisfies []void

box(t:type) := class:
    Put(:t):void = {}

# box([]void){}.Put(Y)  # ERROR - a type parameter bound to []void is
                        # invariant, so []int does not fit
```
<!-- #> -->

[^Intrinsics]: 내장함수. 별도의 소스 없이, 컴파일러가 직접 정의하고 지원하는 functions 를 말합니다.
[^RuntimeInternals]: 런타임 내부 구현 요소. 프로그램이 실행되는 동안 실제로 연산을 수행하고 데이터를 관리하는 시스템 내부의 세부 구현 사항을 말합니다.
[^FirstClassValue]: 일급 객체 값. (1) 함수의 실질적인 매개변수가 될 수 있고 (2) 함수의 반환 값이 될 수 있고 (3) 할당의 대상이 될 수 있고 (4) 비교연산을 적용할 수 있는 객체를 일급 객체라고 합니다.
[^NestedFunction]: 중첩 함수. 어떤 다른 함수의 내부에 선언된 함수를 말합니다.
[^Integers]: 정수를 표현할 수 있는 자료형을 말합니다. 1, 10, 100 등을 표현할 수 있습니다.
[^StringInterpolation]: 문자열 보간. 문자열(String) 안에 값이나 표현식을 삽입하는 방식을 말합니다. 예를 들어, Message := "Current level: {Level}" 라는 코드는 쌍따옴표로 표시되는 문자열 내부에 {Level} 이라는 값을 삽입하고 있습니다.
[^Persisted]: 프로그램의 실행이 끝나거나 환경이 바뀌어도 계속 유지되도록 저장된 데이터를 말합니다.
[^Literals]: 리터럴. Damage := 100 에서의 100 처럼, identifier 를 사용하지 않고 직접 입력된 값을 말합니다. 만약 identifier 를 사용한다면 Damage := BaseDamage 처럼 표현될 수 있을 것입니다.
[^Overflow]: 오버플로. 컴퓨터의 정수 연산 계산 결과가 허용 범위를 초과할 때 발생하는 오류를 말합니다.
[^VM]: 버추얼 머신 (Virtual Machine). 프로그래밍 언어로 작성된 코드를 실행하기 위해 소프트웨어로 구현한 가상의 런타임 환경을 말합니다.
[^VerseVM]: Verse Virtual Machine. Verse를 실행하기 위해 마련된 별도의 가상 머신을 말합니다.
[^BPVM]: Blueprint Virtual Machine. Blueprint 의 실행 바이트코드를 처리하는 가상 머신을 말합니다.
[^UnaryNegation]: 단항 부정. 하나의 피연산자 앞에 마이너스(-) 기호 또는 부정 연산자(!, ~)를 붙여 값을 반전시키는 연산을 말합니다. 64-bit 의 int 범위 최소값은 -9,223,372,036,854,775,808 이고, 최대값은 9,223,372,036,854,775,807 입니다. 따라서, 최소값을 단항 부정하면 -(-9,223,372,036,854,775,808) = 9,223,372,036,854,775,808 이 되므로, 범위의 최대값보다 1 커지게 됩니다. 즉, 본문의 의미는 이 1 차이에 의한 overflow 에 의해서도 (MathIntrinsicCallFailure) 가 발생한다는 것입니다.
[^Rationals]: 유리수. 즉, 분수로 나타낼 수 있는 수를 말합니다.
[^LowestTerms]: 기약분수. 분모와 분자의 공약수가 1뿐이어서 더 이상 약분할 수 없는 분수를 말합니다.↩
