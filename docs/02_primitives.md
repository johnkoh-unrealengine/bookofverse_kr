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

이와 같은 표준 양식은 동등성 검사를 간결하게 하고, 일관된 방식으로 작동되도록 보장합니다.

중요한 특징 : *`int` 는 `rational` 의 subtype 입니다.* 이것은 rational 이 쓰여야 되는 곳이라면 그곳이 어디든 integer 도 쓰일 수 있다는 의미입니다 :

<!--versetest-->
<!-- 08 -->
```verse
ProcessRational(X:rational):rational = X

# integer 를 직접적으로 보낼 수 있습니다
ProcessRational(5) = 5/1     # 5 는 암묵적으로 5/1 (rational) 입니다
ProcessRational(0) = 0/1     # 0 은 암묵적으로 0/1 (rational) 입니다
```

하지만, int 가 기대되는 곳에 rational 을 return 할 수는 *없습니다* - 그것은 narrowing conversion[^NarrowingConversion] 이 되기 때문입니다 :

<!--versetest
assert_semantic_error(3510):
    BadFunction(X:rational):int = X
<#
-->
<!-- 09 -->
```verse
BadFunction(X:rational):int = X  # 오류
```
<!-- #> -->

나머지가 없는 rationals 는 그 등가물의 integer 와 동등합니다 :

<!--versetest-->
<!-- 10 -->
```verse
(2 / 1) = 2             # 참
2 = (2 / 1)             # 참
(4 / 2) = 2             # 참 : 4/2 가 2/1 로 줄어들므로, 2 와 같습니다
(9 / 3) = 3             # 참 : 9/3 가 3/1 로 줄어들므로, 3 과 같습니다
```

이로 인해 계산에서 integer 와 rational 값을 원활하게 섞어 쓸 수 있습니다.

다음의 두 function 들이 rationals 를 integers 로 변환합니다 : `Floor` 는 내림을 하고, `Ceil` 은 올림을 합니다. 

<!--versetest-->
<!-- 11 -->
```verse
# 양수 rationals
Floor(5 / 2)= 2         # 2.5 → 2 (내림)
Ceil(5 / 2) = 3         # 2.5 → 3 (올림)

# 음수 rationals - 방향을 잘 보세요!
Floor((-5) / 2) = -3    # -2.5 → -3 (음의 방향으로 가까운 정수)
Ceil((-5) / 2) = -2     # -2.5 → -2 (양의 방향으로 가까운 정수)

# 분모가 음수인 경우
Floor(5 / -2) = -3      # (-5)/2 와 같습니다
Ceil(5 / -2) = -2       # (-5)/2 와 같습니다

# 분자와 분모 모두 음수인 경우
Floor((-5) / -2) = 2    # 2.5 → 2
Ceil((-5) / -2) = 3     # 2.5 → 3
```

내림은 버림이 *아닙니다* : 이 둘은 양수에서는 결과가 같지만, 음수에서는 결과가 다릅니다. rational 을 인수로 받는 경우, Floor 와 Ceil 은 모두 실패할 수 없습니다. 하지만 `float` 를 인자로 받는 경우에는 `<decides>` 지정자가 붙으면서 실패할 수 있는 수식이 됩니다.

`Floor` 또는 `Ceil` 을 integer 에 상당하는 rational (정수로 딱 떨어지는 값) 에 적용하면, 그것은 integer 를 return 합니다 :

<!--versetest-->
<!-- 12 -->
```verse
# Integer 에 상당하는 rationals
PositiveRational:rational = 5
NegativeRational:rational = -5

Floor(PositiveRational) = 5   # integer 5 를 return 합니다
Ceil(PositiveRational) = 5    # integer 5 를 return 합니다
Floor(NegativeRational) = -5  # integer -5 를 return 합니다
Ceil(NegativeRational) = -5   # integer -5 를 return 합니다

# Integers 로 딱 떨어지는 rationals 에도 작동합니다.
Floor(10 / 2) = 5             # 10/2 = 5/1, 5 를 return 합니다
Ceil(10 / 2) = 5              # 10/2 = 5/1, 5 를 return 합니다
```

Rationals 는 파라미터로 쓰이거나 return 되는 자료형으로 쓰일 수 있습니다 :

<!--versetest-->
<!-- 13 -->
```verse
# rational 을 return 하는 Function
Half(X:int)<computes><decides>:rational = X / 2

# 위 function 의 결과를 파라미터로 사용하는 코드
if (Result := Half[7]):
    Floor(Result) = 3   # 7/2 = 3.5 이므로, Floor 는 3 을 반환합니다
    Ceil(Result) = 4    # 7/2 = 3.5 이므로, Ceil 은 4 를 반환합니다
```

`int` 는 `rational` 의 subtype 이므로, 이 두 자료형들에 의해서만 어떠한 함수를 overload[^Overload] 할 수는 *없습니다*. `rational` 이 `int` 도 받을 수 있기 때문에, `int` 를 전달하는 코드에서 `int` 를 매개변수로 받는 함수와 `rational` 을 매개변수로 받는 함수 중 어떤 것을 호출해야 할지 모호해지고, 이를 원인으로 하는 오류가 발생하기 때문입니다 :

<!--versetest
assert_semantic_error(3532):
    ProcessValue(X:int):void = {}
    ProcessValue(X:rational):void = {}
<#
-->
<!-- 14 -->
```verse
ProcessValue(X:int):void = {}
ProcessValue(X:rational):void = {}  # 오류!
```
<!-- #> -->

컴파일러는 `int` 를 `rational` 보다 더 구체적인 것으로 인식하므로, signature[^Signature] 가 모호해집니다.

Rationals 는 자원 분배와 공평한 계산이 필요한 경우에 장점을 갖습니다 :

<!--versetest-->
<!-- 15 -->
```verse
# 공평한 자원 분배
DistributeResources(TotalGold:int, NumPlayers:int)<decides>:int =
    GoldPerPlayer := TotalGold / NumPlayers
    Floor(GoldPerPlayer)  # 정수 갯수로 떨어지는 금화 조각으로 변환됩니다 (0 개 일 수도 있음)

# 금화가 충분하지 않을 때 실패 처리를 하기 위해, > 0 인지 확인합니다.
DistributeResourcesOrFail(TotalGold:int, NumPlayers:int)<decides>:int =
    GoldPerPlayer := TotalGold / NumPlayers
    Floor(GoldPerPlayer) > 0  # 각 플레이어들이 0 개를 갖게 되면 실패 처리됩니다

# 아이템 구매 가능 여부 계산
Coins:int = 225
CoinsPerQuiver:int = 100
ArrowsPerQuiver:int = 15

if (NumberOfQuivers := Floor(Coins / CoinsPerQuiver)):
    TotalArrows:int = NumberOfQuivers * ArrowsPerQuiver
    # 플레이어는 2 개의 화살통(=30 개의 화살)을 구매할 수 있습니다
```

## Floats

`float` 자료형은 정수가 아닌 숫자 값 전체를 표시합니다. 이는 `1.0`, `-50.5`, `3.14159` 등 큰 값과 정확한 분수를 표현할 수 있습니다. float 는 IEEE 의 64 비트 float 을 말하는데, 이는 `[-2^1024 + 1, … , 0, … , 2^1024 - 1]` 범위의 소수점 있는 양수 및 음수 값, 그리고 `NaN` (Not a Number, 숫자가 아님) 값을 담을 수 있다는 의미입니다. 다만, 그 구현 방식은 아래의 면에서 IEEE 의 표준 방식과 다릅니다 :

- 하나의 `NaN` 값만을 갖습니다.
- `NaN` 값은 그 자신의 값과 동등한 값으로 해석됩니다.
- 모든 숫자는 그 자신의 값과 동등한 값으로 해석됩니다.
- `0` 은 음수가 될 수 없습니다.

작성자는 작성하는 코드 안에 float 값을 literals 로 포함할 수 있습니다 :

<!--versetest-->
<!-- 16 -->
```verse
A:float = 1.0
B := 2.14
MaxHealth : float = 100.0

var C:float = A + B
C = 3.14              # 성공
set C -= 3.14
C = 0.0               # 성공
# C = 0              # 컴파일 에러; 위에 C 의 자료형을 `float` 로 선언했는데, 0 은 `float` literal 이 아니기 때문입니다. (0 은 `int` literal 입니다.)
```

floats 는 네 가지 기본 산수에 쓸 수 있습니다 : `+` 로 덧셈을, `-` 로 뺄셈을, `*` 로 곱셈을, `/` 로 나눗셈을 할 수 있습니다. 또, 이 네 가지 기본 산수를 수행하면서 변수 값을 업데이트 하는 조합된 operators 도 있습니다 :

<!--versetest-->
<!-- 17 -->
```verse
var CurrentHealth : float = 100.0
set CurrentHealth /= 2.0    # CurrentHealth 값을 반으로 나누고 업데이트 합니다
set CurrentHealth += 10.0   # CurrentHealth 값에 10 을 더하고 업데이트 합니다
set CurrentHealth *= 1.5    # CurrentHealth 값에 1.5 를 곱하고 업데이트 합니다
```

### 특수한 Float 값들

Float 는 IEEE 754 semantics 를 따르며, 여기에는 무한과 숫자가 아닌(NaN) 데이터에 대한 특수한 값이 포함됩니다.

<!--versetest-->
<!-- 18 -->
```verse
# 0 으로 나뉜 값에 의한 무한
PosInf := 1.0 / 0.0
NegInf := -1.0 / 0.0

# 유효하지 않은 operations 에 의한 NaN
NaNFromZeroDiv := 0.0 / 0.0
NaNFromSqrt := Sqrt(-1.0)
NaNFromInfDiv := PosInf / PosInf

# NaN 값은 연산을 통해 전달 됩니다
Result := NaNFromZeroDiv + 100.0  # Result 는 NaN 입니다
Product := NaNFromSqrt * 2.0      # Product 는 NaN 입니다
```

0 으로 나뉜 수는 실패로 처리되지 않고, `inf` 또는 `-inf` 를 생성합니다. `0.0 / 0.0`, `Sqrt(-1.0)`, 또는 `Inf - Inf` 와 같이 유효하지 않은 연산은 NaN 값을 생성합니다. NaN 이 포함된 수학적 연산은 그 NaN 값을 결과에 전달합니다. 표준 IEEE 754 와 달리, Verse 에서의 NaN 은 그 자신과 비교되면 동등한 값으로 취급됩니다. (역자 주 - 표준 IEEE 754 에서, NaN == NaN 은 False, NaN != NaN 은 True 로 읽힙니다. 이와 반대로, Verse 에서는 NaN == NaN 이 Ture, NaN != NaN 이 False 로 읽힙니다.)

`int` 를 `float` 로 변환하려면, `MyFloat:=MyInt*1.0` 처럼 기존에 가지고 있던 값에 `1.0` 을 곱하면 됩니다.

## 수학적 Functions

수학적 function 들은 intrinsics[^Intrinsics] 입니다. The [Runtime API Reference](api/verse_runtime_api.html) 에는 모든 수학적 function 들이 그들의 signature[^Signature], effects 와 함께 나열되어 있고, domain[^Domain] 의 경계에서 그들이 어떻게 작동하는지도 함께 기록되어 있습니다; 이 섹션에서는 사람들이 자주 혼동하는 작동 방식에 대해 명확히 다룹니다.

`Abs`, `Min` 그리고 `Max` 는 `int` 와 `float` 모두에 쓸 수 있습니다. floats 의 경우, `NaN` 은 '비교 구문을 적용할 수 없는' 값으로 간주되지 않고, 다른 값들과 마찬가지로 '처리할 수 있는' 값으로 간주하는 Verse 의 규칙을 따릅니다. 그렇기 때문에 Verse 에서 `NaN` 은 무시되지 않고, `Min` 과 `Max` 로부터 다른 식으로 전달 됩니다 :

<!--versetest-->
<!-- 19 -->
```verse
Abs(5) = 5
Abs(-5) = 5
Abs(3.14) = 3.14

# 비교 구문에서 Nan 값이 전달됩니다
Max(NaN, 5.0) = NaN
Min(NaN, 5.0) = NaN

# 무한대 값이 상식적으로 기대되는 대로 처리됩니다
Max(Inf, 100.0) = Inf
Min(-Inf, 100.0) = -Inf
```

`Ceil`, `Floor`, `Round`, `Int` 는 `float` 를 `int` 로 변경합니다. 그리고 이들 네 수식은 실패할 수 있습니다 : 유한하지 않은 argument 는 반올림할 정수가 없으므로 대괄호와 함께 쓰입니다. `Round` 는 0 에서 멀어지는 방향으로 일관되게 반올림 하는 대신, break ties to even[^BreakTiesToEven] 방식을 채택합니다. 따라서, 반올림이 반복 적용된 값이라도 그 결과가 큰 값 쪽으로 치우치지 않습니다 :

<!--versetest-->
<!-- 20 -->
```verse
Round[1.5] = 2     # 동률이므로 짝수 2 로 계산됩니다
Round[2.5] = 2     # 동률이므로 짝수 2 로 계산됩니다
Round[-1.5] = -2   # 동률이므로 짝수 -2 로 계산됩니다
Round[0.5] = 0     # 동률이므로 짝수 0 으로 계산됩니다

Round[1.4] = 1     # 동률이 아니므로 1 로 계산됩니다
Round[1.6] = 2     # 동률이 아니므로 2 로 계산됩니다
```

나머지 소수점 function 들은 결코 실패하지 않습니다. 수학자들의 관점에서 답이 존재하지 않는다고 판단할 법한 지점에서도, `NaN` 또는 무한을 반환한 뒤 계산을 이어나갑니다 : 

<!--versetest-->
<!-- 21 -->
```verse
Sqrt(4.0) = 2.0
Sqrt(-1.0) = NaN   # 음수 입력값에 대한 실수 루트 결과값은 존재하지 않습니다
Sqrt(Inf) = Inf

Pow(2.0, 3.0) = 8.0     # 제곱근(밑, 지수)
Pow(4.0, 0.5) = 2.0     # 루트 2
Pow(2.0, -1.0) = 0.5    # 역수
Pow(0.0, 0.0) = 1.0     # 0 의 0 제곱은 관례에 따라 1 입니다
Pow(NaN, 0.0) = 1.0     # 어떤 값의 0 제곱은 언제나 1 입니다
Pow(1.0, NaN) = 1.0     # 1은 어떤 값으로 제곱하든 1 입니다

Exp(0.0) = 1.0          # Exp() 는 밑을 자연상수 e (약 2.71828) 로 하는 지수 함수 입니다. 즉, e 의 0 제곱 = 1
Exp(-Inf) = 0.0
Ln(1.0) = 0.0           # Ln() 은 밑을 자연상수 e (약 2.71828) 로 하는 자연로그 함수 입니다. 즉, 수식에서 e 를 넘기면, 1 = e 의 0 제곱
Ln(Exp(1.0)) = 1.0      # Ln(e) = 1
Ln(0.0) = -Inf
Ln(-1.0) = NaN

Log(10.0, 100.0) = 2.0  # log₁₀(100); 밑이 함수의 첫번째 변수가 됩니다
Log(2.0, 8.0) = 3.0     # log₂(8) 를 나타냅니다
```

삼각함수는 radian 단로 작동하고, `PiFloat` 은 π 자체는 아니지만 π 에 가장 가까 `float` 값입니다. 이 구분은 시작적으로 명확하게 드러나는데, 수학적 계산 방식으로 정확히 0 이 되어야 하는 각도도 단지 0 에 가깝게 계산되기 때문입니다.

<!--versetest-->
<!-- 22 -->
```verse
Sin(0.0) = 0.0
Sin(PiFloat / 2.0) = 1.0
Cos(0.0) = 1.0
Cos(PiFloat) = -1.0

Sin(NaN) = NaN
Sin(Inf) = NaN          # 의미있는 각도가 아닙니다

# PiFloat 은 π 가 아니기 때문에, 아래의 값들도 0 에 가까울 뿐 0 이 아닙니다
Sin(PiFloat) > 0.0 and Sin(PiFloat) < 0.000001
Cos(PiFloat / 2.0) > 0.0 and Cos(PiFloat / 2.0) < 0.000001

# ArcSin, ArcCos, ArcTan 의 역삼각함수들은 다음의 범위에서 각도를 결과로 반환합니다 :
# ArcSin 과 ArcTan 은 [-π/2, π/2] 의 범위
# ArcCos 은 [0, π] 의 범위
ArcSin(1.0) = PiFloat / 2.0
ArcCos(-1.0) = PiFloat
ArcTan(1.0) = PiFloat / 4.0
```

`ArcSin` 과 `ArcCos` 이 `[-1, 1]` 의 범위 밖에서도 실패하지 않고, 그렇다고 `NaN` 을 반환하지도 않는다는 사실에 주의 하십시오. 범위를 벗어나는 경우 clamp 되므로, `ArcSin(2.0)` 은 단순히 π/2 를 반환하게 됩니다. 엔진은 이와 같은 작동 방식을 오류로 표시하지 않으므로, 부적절한 입력값이 표시되도록 하려면 적절한 범위 내의 argument 를 사용했는지 코드의 작성자가 스스로 체크 해야합니다.

`IsFinite` 는 이와 같은 상황에서 보완 장치로 쓸 수 있습니다. 이는 `Inf`, `-Inf`, `Nan` 에 대해서는 모두 실패로 처리하고, 그 외의 경우에는 결과값 자체를 성공으로 처리하므로, 전체 계산 과정에서 값이 유지될 수 있게 해줍니다.

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
(5.0).IsFinite[]      # 성공
(-100.0).IsFinite[]   # 성공

(Inf).IsFinite[]  # 실패
(NaN).IsFinite[]  # 실패

# 원래 값을 반환하며 성공하므로, 다른 표현식 안에서 직접 사용할 수 있습니다.
(15.16).IsFinite[] = 15.16

# 유효성 검사 : Inf 또는 NaN 을 반환하지 않고 그냥 실패로 처리됩니다
SafeDivide(X:float, Y:float)<computes><decides>:float =
    X.IsFinite[] and Y.IsFinite[]
    Result := X / Y
    Result.IsFinite[]
    Result
```
<!-- #> -->

마지막으로, named constants[^NamedConstants]는 다음과 같습니다 :

<!--NoCompile-->
<!-- 24 -->
```verse
PiFloat # 3.14159265358979323846...
Inf     # 양의 무한
-Inf    # 음의 무한
NaN     # 숫자가 아님 (Not a Number)
```

## Booleans

`logic` 자료형은 `true` 와 `false` 의 Boolean 값을 나타냅니다.

<!--versetest-->
<!-- 25 -->
```verse
A:logic = true
B := false

# A = B          # 실패
A?                # 성공
# B?             # 실패

true?             # 성공
# false?         # 실패
```

`logic` 자료형은 쿼리 연산과 비교 연산만 지원합니다. 쿼리 표현식은 쿼리 연산자 `?` 를 사용하여 논리값이 사실인지 확인하고, 논리 값이 `false` 인 경우에는 실패로 처리됩니다. 비교 연산에서 두 논리값이 같은지 테스트 하려면 실패할 수 있는 연산자인 `=` 를 사용하고, 두 논리값이 서로 다른지 확인하려면 `<>` 연산자를 사용하면 됩니다.

많은 프로그래밍 언어들이 연산 성공과 실패에 대한 신호를 표시하기 위해 `logic` 과 같은 자료형을 관용적으로 씁니다. Verse 에서는 가능하다면 `logic` 과 같은 자료형을 쓰지 않고 성공과 실패 그 자체를 씁니다. 조건문은 guard[^Guard] 가 성공한 경우에만 `then` 분기를 실행합니다 :

<!--versetest
ShowTargetLockedIcon():void={}
TargetLocked:?int = option{42}
-->
<!-- 26 -->
```verse
if (TargetLocked?):
    ShowTargetLockedIcon()
```

`<decides>` 효과를 갖는 표현식 연산 결과가 성공인 경우 `true` 로, 실패인 경우 `false` 로 변형하기 위해 `logic{ exp }` 를 씁니다 :

<!--versetest
F()<decides>:void=
    GotIt := logic{5 > 3}
    GotIt?
<#
-->
<!-- 27 -->
```verse
GotIt := logic{5 > 3}    # 비교 결과가 성공이므로, GotIt 은 참 입니다
GotIt?                   # 그러므로 이 결과도 성공입니다
GotIt = false            # 그리고 이 식은 실패입니다
not GotIt?               # 그리고 이 식도 실패입니다
```
<!-- #> -->

## 문자와 문자열

텍스트는 문자(characters)와 문자열(strings)로 표현됩니다. `char` 은 전체 Unicode code point[^CodePoint]가 아니라, 단일 **UTF-8 code unit** 입니다. 그러므로 문자열은 문자의 배열이고, `[]char` 처럼 쓰입니다. 편의성을 위해서, Verse 는 `[]char` 자료형에 대한 별칭으로 `string` 을 제공합니다 :

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
[^LowestTerms]: 기약분수. 분모와 분자의 공약수가 1뿐이어서 더 이상 약분할 수 없는 분수를 말합니다.
[^NarrowingConversion]: 축소 변환. 더 큰 범위나 높은 정밀도를 가진 데이터 자료형을 더 작은 범위나 낮은 정밀도를 가진 데이터 자료형으로 변환하는 것을 말합니다. 축소 변환되는 경우, 정밀한 값을 갖는 데이터 부분이 손실되거나, overflow 가 발생할 수 있습니다. 1.25L 짜리 음료수를 255ml 짜리 컵에 전부 쏟아 부으면, 음료(Data)가 넘쳐서(overlow) 손실되기 마련인 점을 생각해보면 되겠습니다.
[^Overload]: 오버로드. 같은 클래스에 이름은 같고 매개변수만 다른 메서드를 여러개 만드는 것을 말합니다. 예를 들어, Calculate 라는 클래스 안에 서로 다른 매개변수를 쓰는 add(int a, int b) 와 add(float a, float b) 를 동시에 정의할 수 있는데, 이를 두고 add 라는 함수가 overload 되어있다고 표현할 수 있습니다.
[^Signature]: 시그니처. 함수 시그니처 라고도 합니다. 함수(Function)를 호출하는 방법과 규격에 대한 명세를 말합니다. 그 구성요소로는 함수의 이름, 매개변수 목록, 반환 자료형 등이 있습니다. 선언부에 기재된 이와 같은 정보로 인해, 개발자들은 함수 내부 코드를 다 읽어보지 않아도 해당 함수를 쓰기 위해 어떤 데이터를 넣어야 할지 파악할 수 있습니다.
[^Domain]: 정의역. 함수에 입력할 수 있도록 허용된 수나 값들의 전체 범위를 말합니다. Domain 의 경계값 이라는 것은, 그 범위의 끝에 있는 값들을 말합니다. 예를 들어, 정의역이 '0 보다 큰 integer' 로 정의되어 있다면, 0 은 Domain 에 포함되지 않는 경계값이고, 1 은 Domain 에 포함되는 경계값 입니다.
[^BreakTiesToEven]: 반올림 방식의 하나를 말합니다. Banker's rounding 이라고도 합니다. 반올림할 자리가 정확히 중간값(0.5 등) 인 경우, 가장 가까운 짝수로 계산하되, 앞자리가 홀수이면 큰 방향의 짝수로, 앞자리가 짝수이면 작은 값의 짝수로 계산합니다. 예를 들어 2.5 는 2 로 계산되고, 3.5 도 4 로 계산됩니다.
[^NamedConstants]: 명명된 상수. 고유한 식별자(이름)를 부여받아 코드에서 해당 이름으로 참조할 수 있는 상수를 말합니다.
[^Guard]: 가드. 조건문에서, 조건의 성립 여부를 확인하는 표현식을 말합니다. 예를 들어, if (Health > 0): Print("Alive") 라는 조건문이 있다면, Health > 0 부분이 guard 입니다.
[^CodePoint]: Unicode 에서, 각각의 문자에 부여한 고유한 숫자 값을 말합니다. 예를 들어, 글자 'A' 의 code point 는 U+0041 이고, '가' 의 code point 는 U+AC00 입니다. 이러한 code point 를 실제 메모리에 저장하려면 바이트(Byte) 단위로 인코딩해야 하는데, UTF-8 이 이를 위한 대표적인 인코딩 방식입니다. UTF-8 은 Code Point의 크기에 따라 1~4바이트를 사용해서 저장합니다. 예를 들어, U+0041 를 저장하려면 1 바이트가 필요하고, U+AC00 을 저장하려면 3 바이트가 필요합니다. 그리고 이때 UTF-8 이 저장을 위해 사용하는 기본 단위인 '1 바이트' 가 곧 UTF-8 의 Code Unit 입니다. (참고로, 모든 인코딩 방식에서의 Code Unit 이 1 바이트인 것은 아닙니다. 예를 들어, UTF-16 인코딩 방식에서의 Code Unit 은 2 바이트 입니다.)
