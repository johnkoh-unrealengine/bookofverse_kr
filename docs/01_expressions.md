# Expressions[^Expressions]

모든 것이 expressions 입니다. 이 설계 원칙이 Verse 가 다른 많은 프로그래밍 언어들과 구분되는 차별점이 됩니다. 다른 프로그래밍 언어들은 statements[^Statements] 와 expressions 가 서로 구분되기 때문입니다. 당신이 작성하신 모든 코드 조각들은 값을 생성합니다. 심지어 순전히 side effect[^SideEffect] 만 있을 것이라고 예상하시는 구조에서도 마찬가지 입니다. 이 특징은 Verse 에 의한 프로그래밍 모델이 더 자연스럽고 예측 가능한 방식으로 구성될 수 있도록 도와줍니다.

## 기본 Expressions

모든것은 기본 expressions 로부터 시작됩니다. 여기서 말하는 기본 expressions 란, 더 복잡한 표현식을 구성하는 가장 기본적인 단위를 말합니다. 이는 literals[^Literals], identifiers[^Identifiers], 괄호로 묶은 expressions, 그리고 여러 데이터를 가볍게 묶을 수 있는 tuple[^Tuple] 구조를 포함합니다.

### 기본 값

Literals 는 상수 값을 소스 코드로 표현한 것입니다. Verse 는 integers[^Integers], floats[^Floats], characters[^Characters], strings[^Strings], booleans[^Booleans], 그리고 functions[^Functions] 와 같은 기본 자료형에 대한 literals 를 제공합니다. 각각의 자료형은 그것의 고유한 literal 구문을 갖고, 컴파일 시점에 유효한 값과 그 해석을 규정하는 규칙을 갖습니다.

<!--versetest
point := struct{X:float, Y:float}
Condition:logic = true
-->
<!-- 01 -->
```verse
Result := if (Condition?) then 42 else 3.14  # Integer 와 float 자료형인 literals
array{1, 2, 3}                               # 배열 구조 내부의 Integer literals
point{X:=0.0, Y:=1.0}                        # object 구조 내부의 Float literals
```

#### Integer Literals

Integer literals 는 정수를 나타내고, 두 가지 형식으로 쓰일 수 있습니다 :

*소수점 표기법*은 표준 숫자를 사용합니다 :

<!--versetest-->
<!-- 02 -->
```verse
Count := 42
Negative := -17
Zero := 0
Large := 9223372036854775807                # 최대 64-bit 의 부호 있는 integer literals 를 표현할 수 있습니다
```

*16진수 표기법*은 `0x` 접두사 뒤에 (0-9, a-f, A-F) 로 이뤄진 16진수 숫자를 사용합니다 :

<!--versetest-->
<!-- 03 -->
```verse
Byte := 0xFF
Address := 0x1F4A
LowercaseHex := 0xabcdef
UppercaseHex := 0xABCDEF
```

**Literal 자료형에 대한 제한과 Runtime 에서의 작동 방식 :**

Integer literals 는 64-bit 의 부호 있는 정수 값 범위(`-9223372036854775808` to `9223372036854775807`) 내에 있어야 합니다. 이는 코드에 직접 입력할 수 있는 값에 대한 제한으로, 컴파일 시점에 적용 됩니다.

런타임 중에, integer 값은 arbitrary precision arithmetic[^ArbitraryPrecisionArithmetic] 방식을 사용하며, 연산 과정에서 64-bit 제한을 넘을 수 있습니다. 하지만, 64-bit 를 넘은 integers 는 제한된 지원만 받을 수 있습니다. (예를 들어, string interpolation[^StringInterpolation] 을 쓸 수 없거나 persisted[^Persisted] 가 될 수 없습니다.)

#### 자료형이 Float 인 Literals

소수점을 사용한 literals 는 십진법 숫자로 읽히므로, 올바른 소수점을 포함해야 합니다. 또, 경우에 따라서는 `f64`[^f64] 라는 접미사를 포함해야 합니다.

<!--versetest-->
<!-- 04 -->
```verse
Pi := 3.14159
Half := 0.5
Explicit := 12.34f64    # 명시적으로 bit 심도를 표현하는 접미사
```

Scientific notation expresses very large or small numbers using exponents:

<!--versetest-->
<!-- 05 -->
```verse
Large := 1.0e10         # 10,000,000,000 (sign optional)
Small := 1.0e-5         # 0.00001
WithSign := 2.5e+3      # 2,500 (explicit + sign)
Compact := 1.5e2        # 150 (no sign defaults to +)
```

Float literals must include a decimal point (`1.0` is valid, but `1` is an integer). A final decimal point without digits is invalid (`1.` is a syntax error). All floats are 64-bit (IEEE 754 double precision); the `f64` suffix is optional. Unary operators work as with integers: `-1.0`, `+1.0`.

**Overflow and Underflow Behavior:**

Float literals outside the IEEE 754 double-precision range produce
**compile-time errors**:

<!--versetest-->
<!-- 06 -->
```verse
#TooBig := 1.7976931348623159e+308    # Compile error: literal overflow
Maximum := 1.7976931348623158e+308    # OK: Maximum finite float
```

However, **runtime** float arithmetic follows standard IEEE 754 semantics:

<!--versetest-->
<!-- 666 -->
```verse
# Runtime overflow produces infinity
Large := 1.0e308
Overflow := Large * 10.0    # Overflow produces infinity

# Division by zero produces infinity
PosInf := 1.0 / 0.0
NegInf := -1.0 / 0.0

# Underflow produces denormalized numbers or zero
Small := 1.0e-320
Smaller := Small / 1.0e10   # Underflows gracefully
```

Float operations follow IEEE 754 semantics. Operations that would
produce NaN (like `0.0 / 0.0`, `Inf - Inf`, or `Sqrt(-1.0)`) return
NaN values rather than failing. NaN propagates through arithmetic
operations.

#### Character Literals

Character literals represent individual text units. Verse has two character types with different literal syntax:

`char` literals represent UTF-8 code units (single bytes, 0-255):

<!--versetest-->
<!-- 07 -->
```verse
LetterA := 'a'          # Printable ASCII character
Space := ' '
Tab := '\t'             # Escape sequence
LetterA := 0o61         # Hexadecimal notation: 0oXX (97 decimal = 'a')
```

`char32` literals represent Unicode code points:

<!--versetest-->
<!-- 08 -->
```verse
Emoji := '😀'           # Non-ASCII automatically char32
Accented := 'é'
ChineseChar := '好'
HexUnicode := 0u1f600   # Hex notation: 0uXXXXX (😀)
```

Type inference from literals:

- ASCII characters (`U+0000` to `U+007F`): `'a'` has type `char`
- Non-ASCII characters: `'😀'` has type `char32`
- No implicit conversion between `char` and `char32`

Escape sequences work in both `char` and strings:

| Escape | Meaning | Codepoint |
|--------|---------|-----------|
| `\t`   | Tab     | U+0009 |
| `\n`   | Newline | U+000A |
| `\r`   | Carriage return | U+000D |
| `\"`   | Double quote | U+0022 |
| `\'`   | Single quote | U+0027 |
| `\\`   | Backslash | U+005C |
| `\{`   | Left brace (string interpolation) | U+007B |
| `\}`   | Right brace (string interpolation) | U+007D |
| `\<`   | Less than | U+003C |
| `\>`   | Greater than | U+003E |
| `\&`   | Ampersand | U+0026 |
| `\#`   | Hash      | U+0023 |
| `\~`   | Tilde     | U+007E |

Numeric character notation works as follows:

- `0oXX` for `char` (hexadecimal notation, `0o00` to `0oFF` for values 0-255)
- `0uXXXXXX` for `char32` (hexadecimal notation, `0u000000` to `0u10ffff`)

Character literals cannot be empty or contain multiple characters.

#### String Literals

String literals represent text sequences and support interpolation for embedding expressions. Basic strings use double quotes:

<!--versetest-->
<!-- 09 -->
```verse
Greeting := "Hello, World!"
Empty := ""
WithEscapes := "Line 1\nLine 2\tTabbed"
```

String interpolation embeds expressions using curly braces:

<!--versetest
Format(D:float, ?Decimals:int):string=""
-->
<!-- 10 -->
```verse
Name := "Alice"
Age := 30

# Simple interpolation
Message := "Hello, {Name}!"                      # "Hello, Alice!"

# Expression interpolation
Info := "Age next year: {Age + 1}"               # "Age next year: 31"

# Function calls
Score := 100
Text := "Score: {ToString(Score)}"               # "Score: 100"

# Function calls with named arguments
Distance := 5.5
Formatted := "Distance: {Format(Distance, ?Decimals:=2)}"
```

Multi-line strings can span multiple lines using interpolation braces for continuation:

<!--versetest-->
<!-- 11 -->
```verse
LongMessage := "This is a multi-line {
}string that continues across {
}multiple lines."
# Result: "This is a multi-line string that continues across multiple lines."

OtherMessage := "Another message{
}    with some empty{
}    spaces."
# Result := "Another message    with some empty    spaces."
```

The compiler ignores empty interpolants:

<!--versetest-->
<!-- 12 -->
```verse
Text1 := "ab{}cd"        # Same as "abcd"
Text2 := "ab{
}cd"                    # Same as "abcd" (newline ignored)
```

Curly braces must be escaped (`"\{ \}"`) to appear as literal characters in strings. The `string` type is an alias for `[]char` (array of UTF-8 code units). Since UTF-8 code units are single bytes, strings are byte sequences rather than Unicode character sequences. For example, `"José".Length` returns `5` (5 code units/bytes, not 4 characters, since é takes 2 code units).

String-array equivalence:

<!--versetest-->
<!-- 13 -->
```verse
Test1 := logic{"abc" = array{'a', 'b', 'c'}}    # True
Test2 := logic{"" = array{}}                    # True
```

The compiler removes comments from strings:

<!--versetest-->
<!-- 14 -->
```verse
Text1 := "abc<#comment#>def"     # Same as "abcdef"
```

#### Boolean Literals

The `logic` type has two literal values:

<!--versetest-->
<!-- 15 -->
```verse
IsReady := true
IsComplete := false
```

Use boolean values with the query operator `?` or in comparisons:

<!--versetest
StartGame():void = {}
ShowResults():void = {}
IsReady:logic = true
IsComplete:logic = false
-->
<!-- 16 -->
```verse
if (IsReady?):
    StartGame()

if (IsComplete = true):
    ShowResults()
```


The `logic{}` expression creates boolean values from failable expressions (see [Failure](08_failure.md) for details on failable expressions):

<!--versetest
Operation()<computes><decides>:void = {}
Optional:?int = option{1}
X:int = 1
Y:int = 1
-->
<!-- 17 -->
```verse
# Converts <decides> expression to logic value
Success := logic{Operation[]}        # True if succeeds, false if fails
HasValue := logic{Optional?}         # True if optional has value
IsEqual := logic{X = Y}              # True if equal, false otherwise
```

The `logic{}` expression requires at least a superficial possibility of failure. Pure expressions without `<decides>` effect cause errors:

<!--versetest-->
<!-- 18 -->
```verse
# ERROR: logic{0} has no decides effect
# ERROR: logic{} is empty
Valid := logic{false?}               # OK: false? can fail
```

Multiple expressions inside `logic{}` can be separated by semicolons or commas (see [Semicolons vs Commas](#semicolons-vs-commas) for details):

<!--versetest-->
<!-- 19 -->
```verse
Result1 := logic{true?; true?}       # Semicolon separator
Result2 := logic{true?, true?}       # Comma separator
```

#### Path Literals

Path literals identify modules and packages using a hierarchical naming scheme:

<!--NoCompile-->
<!-- 21 -->
```verse
/Verse.org/Verse                    # Standard library path
/YourGame/Player/Inventory          # Custom module path
/user@example.com/MyModule          # Personal namespace
```

Path syntax follows specific rules:

- Starts with `/`
- Contains label (alphanumeric, `.`, `-`)
- Identifiers must start with letter or `_`

The Modules chapter covers path literals in detail.

### Identifiers and References

Identifiers serve as references to values, whether they are constants,
variables, functions, or types. An identifier consists of:

- **First character:** Letter (A-Z, a-z) or underscore (`_`)
- **Subsequent characters:** Letters, digits (0-9), or underscores
- **Reserved:** Single underscore `_` cannot be used as an identifier

Identifiers are case-sensitive and use only ASCII characters—Unicode
characters are not supported in identifiers.

<!--NoCompile-->
<!-- 22 -->
```verse
int               # Reference to the int type
GetValue          # Reference to a function
Counter           # Reference to a variable
my_class          # Reference to a class
_private          # Leading underscore allowed
variable123       # Digits allowed after first character

# Invalid identifiers:
# 123invalid      # Cannot start with digit
# my-variable     # Hyphen not allowed
# café            # Unicode not supported
# _               # Single underscore is reserved
```

The language does not syntactically distinguish between different kinds
of identifiers (types, functions, variables)—the context determines how
each identifier is used.

### Parentheses and Grouping

Parentheses serve dual purposes: they group expressions to control
evaluation order, and they create tuple expressions. A parenthesized
expression simply evaluates to the value of its contents, allowing you
to override the default operator precedence or improve readability:

<!--versetest
A:int = 1
B:int = 2
C:int = 3
X:int = 5
Y:int = 10
Positive:string = "positive"
Negative:string = "negative"
-->
<!-- 23 -->
```verse
(A + B) * C       # Group addition before multiplication
if (X > 0 and Y > 0) then Positive else Negative
```

### Tuples

Tuples provide a way to group two or more values with little
ceremony. The syntax distinguishes between parentheses used for
grouping and those used for tuple construction through the presence of
commas:

<!--versetest
X:int = 5
Y:int = 10
-->
<!-- 24 -->
```verse
(X, Y)            # Two-element tuple
(1, "hello", true) # Mixed-type tuple
```

Tuples can be accessed using function-call syntax with a single integer argument:

<!--versetest-->
<!-- 25 -->
```verse
point := (10, 20)
x := point(0)     # Access first element
y := point(1)     # Access second element
```

Write tuple types as follows:

<!--versetest
GetPoint():tuple(int,int) = (10, 20)
GetData():tuple(int,string,logic) = (42, "hello", true)
<#
-->
<!-- 26 -->
```verse
tuple(int,int)
tuple(int,string,logic)
```
<!-- #> -->

While the compiler accepts single-element tuple types like `tuple(int)`,
there is currently no syntax to construct a single-element tuple value.

## Postfix Operations

Postfix operations are operations that follow their operand and can be
chained together. This creates a left-to-right reading order that
feels natural and allows for intuitive composition.

### Member Access

The dot operator provides access to members of objects, modules, and
other structured values. Member access expressions evaluate to the
value of the specified member:

<!--NoCompile-->
<!-- 27 -->
```verse
Player.Health           # Access field
Config.MaxPlayers       # Access nested value
math.Sqrt(16.0)         # Access module function
Point.X                 # Access struct field
```

Member access can be chained, creating paths through nested structures:

<!--versetest
item := class{Name:string = "Sword"}
inventory := class{Items:[]item = array{item{}}}
player_type := class{Inventory:inventory = inventory{}}
game := class{Players:[]player_type = array{player_type{}}}
M()<decides>:void =
    Game:game = game{}
    Game.Players[0].Inventory.Items[0].Name
<#
-->
<!-- 28 -->
```verse
Game.Players[0].Inventory.Items[0].Name
```
<!-- #> -->

### Computed Access

Square brackets provide computed access to elements, whether for
arrays, maps, or other indexable structures. Verse evaluates the expression within
brackets to determine which element to access:

<!--versetest
ComputeIndex():int = 0
M()<decides>:void =
    Array:[]int = array{1, 2, 3}
    Map:[string]int = map{"key" => 42}
    Matrix:[][]int = array{array{1, 2}, array{3, 4}}
    Row:int = 0
    Col:int = 1
    Data:[]int = array{10, 20, 30}
    Array[0]
    Map["key"]
    Matrix[Row][Col]
    Data[ComputeIndex()]
<#
-->
<!-- 29 -->
```verse
Array[0]                # Array indexing
Map["key"]              # Map lookup
Matrix[Row][Col]        # Nested indexing
Data[ComputeIndex()]    # Dynamic index computation
```
<!-- #> -->

The square bracket syntax `Func[]` is **required** for calling
functions that may fail (those with the `<decides>` effect). Use regular
parentheses `Func()` for functions that always succeed. Array
indexing also uses `[]` because it can fail when the index is out of bounds.

```verse
GetValue()<decides>:int = ...
GetData():int = ...

# Must use [] for functions that may fail
if (X := GetValue[]):
    Print("Got: {X}")

# Must use () for functions that always succeed
Y := GetData()

# ERROR: Cannot use () for failable functions
# Z := GetValue()  # Compile error!
```

### Function Calls

Function calls use parentheses with comma-separated arguments. The
language treats function calls as expressions that evaluate to the
function's return value:

<!--versetest
Sqrt(X:int):float = 4.0
MaxOf(A:int, B:int):int = if (A > B) then A else B
Initialize():void = {}
GetData():int = 42
Transform():int = 10
Process(X:int, Y:int)<decides>:void = {}
M()<decides>:void =
    A:int = 5
    B:int = 10
    Sqrt(16)
    MaxOf(A, B)
    Initialize()
    Process[GetData(), Transform()]
<#
-->
<!-- 30 -->
```verse
Sqrt(16)                        # Single argument
MaxOf(A, B)                     # Multiple arguments
Initialize()                    # No arguments
Process[GetData(), Transform()] # Nested calls, outer call may fail
```
<!-- #> -->

## Object Construction

Object construction uses a distinctive brace syntax to indicates the
creation of a new instance. The syntax requires explicit field
initialization using the `:=` operator:

<!--versetest
point := struct{ X:int, Y:int }
player := struct{Name:string, Level:int, Health:int}
config := struct { MaxPlayers:int, Difficulty:string, EnablePvP:logic }
-->
<!-- 31 -->
```verse
point{X:=10, Y:=20}
player{Name:="Hero", Level:=1, Health:=100}
config{
    MaxPlayers := 16,
    EnablePvP := true,
    Difficulty := "normal"
}
```

The use of `:=` for field initialization reinforces that these are
binding operations—you're binding values to fields at construction
time. Object constructors can be nested, creating complex
initialization expressions:

<!--versetest
point:=struct{ X:int, Y:int}
inventory:=struct{Capacity:int}
player:=struct{ Position:point, Inventory:inventory}
config:=struct{Difficulty:string}
game_state:=struct{Player:player, Settings:config}
-->
<!-- 32 -->
```verse
Game := game_state{
    Player := player{
        Position := point{X:=0, Y:=0},
        Inventory := inventory{Capacity:=20}
    },
    Settings := config{Difficulty:="hard"}
}
```

## Control Flow as Expressions

One of Verse's distinctive features is that control flow constructs
are expressions, not statements. This means that if-expressions,
loops, and case expressions all produce values that can be used in
larger expressions.

### Conditional

The if-then-else construct is an expression that evaluates to one of
two values based on a condition:

<!--versetest
ComputeA():int=1
ComputeB():int=1
X:int = 5
Condition:logic = true
-->
<!-- 33 -->
```verse
Result := if (X > 0) then "positive" else "negative"
Value := if (Condition=true) then ComputeA() else ComputeB()
```

The else clause can be omitted, though this affects the type of the
expression. Verse supports multiple syntactic forms for
if-expressions, including parenthesized conditions and indented
bodies:

<!--versetest
Condition:logic = true
Value1:int = 42
Value2:int = 100
-->
<!-- 34 -->
```verse
# Standard form
if (Condition?) then Value1 else Value2

# Indented form
if:
    Condition?
then:
    Value1
else:
    Value2
```

### For

For expressions iterate over collections and produce values. The basic
form iterates over elements:

<!--versetest
Process(Item:int):void={}
Collection:[]int = array{1, 2, 3}
-->
<!-- 35 -->
```verse
for (Item : Collection) { Process(Item) }
```

An extended form provides access to both index and item--in the case
of a `Map`, indices are not limited to integers:

<!--versetest
Collection:[]int = array{1, 2, 3}
-->
<!-- 36 -->
```verse
for (Index -> Item : Collection) {
    Print("Item at {Index} is {Item}")
}
```

Since for expressions are themselves expressions, they produce array
values and compose with other expressions. Verse evaluates the body of a for
expression for each successful iteration, and these evaluations determine
the value of the expression as a whole.

### Loop

Loop expressions provide indefinite iteration, continuing until
explicitly terminated through failure or other control flow:

<!--versetest
GetNext():int=1
Done(Value:int)<computes><decides>:void={}
Process(Value:int):void={}
M():void=
    loop {
        Value := GetNext()
        if (Done[Value]) then break
        Process(Value)
    }
<#
-->
<!-- 37 -->
```verse
loop {
    Value := GetNext()
    if (Done[Value]) then break
    Process(Value)
}
```
<!-- #> -->

The loop construct can use indented syntax for clarity.

A loop expression produces a value of type `true`, regardless of what
expressions appear in its body. This value has no practical use—loops are typically used for their side effects rather than their return value.

```verse
Result := loop:
    ProcessData()
    if (ShouldStop[]):
        break
# Result has type 'true' (and returns `true`)
```

### Case

Case expressions provide multi-way branching based on value matching:

<!--versetest
color := enum:
    Red
    Yellow
    Green
    Other
Color:color = color.Red
-->
<!-- 38 -->
```verse
Description := case(Color) {
    color.Red => "Danger",
    color.Yellow => "Warning",
    color.Green => "Safe",
    _ => "Unknown"
}
```

The `_` pattern serves as a catch-all, ensuring the case expression is
exhaustive. Case expressions evaluate to the value of the matched
branch, making them useful for value computation as well as control
flow.

## Binary Operations

Binary expressions follow a carefully designed precedence hierarchy
that balances mathematical conventions with programming practicality.

### Assignment and Binding

At the lowest precedence level, assignment operators bind values to
identifiers. The `:=` operator creates immutable bindings, while `set
=` performs mutable assignment:

<!--versetest-->
<!-- 39 -->
```verse
X := 42           # Immutable binding
Y := X * 2        # Binding to computed value
Z := W := 10      # Right-associative chaining
```

Assignment operators are right-associative, meaning that `a := b := c`
groups as `a := (b := c)`. This allows for natural chaining of
assignments while maintaining clarity about evaluation order.

Compound assignments provide shorthand for common update patterns:

<!--versetest
F()<transacts>:void=
    var Counter :int = 0
    var Total :int = 0
    Factor:=2
    set Counter += 1
    set Total *= Factor
<#
-->
<!-- 40 -->
```verse
set Counter += 1      # Equivalent to: set Counter = Counter + 1
set Total *= Factor   # Equivalent to: set Total = Total * Factor
```
<!-- #> -->

Compound assignment operators evaluate the left-hand side expression only once, which is observable when the expression has side effects:

<!--versetest
assert:
    var TestArray:[]int = array{10, 20, 30, 40, 50}
    var Index:int = 0
    Inc():int =
        set Index += 1
        Index

    # Compound assignment: Inc() called ONCE
    set TestArray[Inc()] += 1

    # Verify: Index = 1 (Inc called once)
    Index = 1
    # TestArray[1] = 20 + 1 = 21
    TestArray[1] = 21
-->
```verse
var Index:int = 0
Inc():int =
    set Index += 1
    Index

# Compound assignment calls Inc() one
set Array[Inc()] += 1
# Result: Array[1] = Array[1] + 1

# Expanded form would call Inc() twice
# set Array[Inc()] = Array[Inc()] + 1
# Result: Array[1] = Array[2] + 1  (different!)
```

In the compound assignment `set Array[Inc()] += 1`, Verse calls the function `Inc()`
once to determine the index, then reads that location,
increments it, and stores the result back.

### Range Expressions

The range operator (`..`) creates integer ranges for iteration in
`for` loops. Ranges are **inclusive on both ends** and can only appear
directly in for loop iteration clauses:

<!--versetest
End()<computes>:int=10
Count:int=10
Start:int=1
Process(I:int):void={}
F():void=
    for (I := 1..10):
        for (J := I..(I+10)):
            for (K:= J..End()) {}
<#
-->
<!-- 41 -->
```verse
1..10             # Range from 1 to 10 (inclusive)
Start..End        # Variable-defined range
for (I := 0..Count):  # Must use := syntax, not :
    Process(I)
```
<!-- #> -->

Ranges are not first-class values. They cannot be stored in variables
or used outside of `for` loop iteration clauses. See the [Range
Operator Restrictions](07_control.md#for-expressions)
section for details.

### Logical Operations

Logical operators combine boolean values with short-circuit
evaluation. Their result is either success or failure. Verse uses
keyword operators (`and`, `or`, `not`) rather than symbols, improving
readability:

<!--versetest
ProcessQuadrant()<computes>:void = {}
Validated:logic= true
UseDefault()<computes><decides>:void = {}
IsReady()<computes><decides>:void = {}
Wait()<computes>:void = {}
M()<transacts>:void =
    X:int = 5
    Y:int = 10
    if (X > 0 and Y > 0) then ProcessQuadrant()
    Result := logic{Validated? or UseDefault[]}
    if (not IsReady[]) then Wait()
<#
-->
<!-- 42 -->
```verse
if (X > 0 and Y > 0) then ProcessQuadrant()
Result := logic{Validated? or UseDefault[]}
if (not IsReady[]) then Wait()
```
<!-- #> -->

The precedence ensures that `and` binds tighter than `or`, matching
mathematical logic conventions, the `logic{}` expression turns success
or failure into a value:

<!--NoCompile-->
<!-- 43 -->
```verse
# Evaluates as: (ExpA and ExpB) or (ExpC and ExpD)
Condition := logic{ExpA and ExpB or ExpC and ExpD}
```

**Important:** Variable bindings do not escape from logical operations.
When you use `:=` inside `and`, `or`, or `not` expressions, those
bindings are only evaluated for short-circuit control flow and are **not**
accessible afterward:

<!--NoCompile-->
<!-- 998 -->
```verse
Arr:[]int = array{10, 20}

# ERROR: Bindings in logical operations are NOT accessible
if ((X := Arr[0]) and (Y := Arr[1])):
    # X and Y are not bound here - this will cause a compilation error!
    Z := X + Y

# Simple if binding DOES work
if (X := Arr[0]):
    # OK: X is accessible here
    Y := X + 1
```

### Comparison Operations

Comparison operators also either succeed or fail and can be chained
for range checking:

<!--versetest
InRange():void={}
Value:int = 50
X:int = 75
Minimum:int = 0
Maximum:int = 100
A:int = 5
B:int = 10
-->
<!-- 44 -->
```verse
if (0 <= Value <= 100) then InRange()
IsValid := logic{X > Minimum and X < Maximum}
Same := logic{A = B}
Different := logic{A <> B}
```

All comparison operators have the same precedence and evaluate
**left-to-right**. Crucially, *comparison operators return their left
operand* when the comparison succeeds, and *comparison chains have special
syntax* that checks all adjacent pairs.

<!--versetest
assert:
    X := 0 < 10
    X = 0  # Returns left operand (0)

    Value:int = 50
    Result := 0 <= Value <= 100
    Result = 0  # Chain returns leftmost operand (0)

    # Chain checks BOTH comparisons
    Value2:int = 75
    not(10 <= Value2 <= 50)  # Fails because 75 > 50
<#
-->
<!-- 999 -->
```verse
X := 0 < 10
# X equals 0 (the left operand)

0 <= Value <= 100
# Special chain syntax that checks BOTH:
#   - 0 <= Value (lower bound)
#   - Value <= 100 (upper bound)
# Returns 0 (leftmost operand) if both succeed
```
<!-- #> -->

Verse does **not** evaluate the comparison chain `A <= B <= C` as `(A <= B) <= C`.
Instead, it is special syntax that checks both `A <= B` **and** `B <= C`, while
returning the leftmost operand (`A`) on success. This enables natural
mathematical notation for ranges without requiring `and` operators.

### Arithmetic Operations

Arithmetic operations follow standard mathematical precedence, with
multiplication and division binding tighter than addition and
subtraction:

<!--versetest
A:int = 1
B:int = 2
C:int = 3
-->
<!-- 45 -->
```verse
Result := A + B * C      # Multiplication first
Average := (A + B) / 2   # Parentheses override precedence
```

Integer division by zero fails and has the `<decides>` effect.
When dividing integers, `X / Y` can fail if `Y` is `0`, allowing you to handle
this case safely:

<!--versetest
X:int = 10
Y:int = 0
assert:
    not(Result := X / Y)
-->
<!-- 997 -->
```verse
if (Result := X / Y):
    Print("Division succeeded")
else:
    Print("Cannot divide by zero")
```

Float division by zero does not fail; it returns infinity according to
IEEE 754 floating-point semantics.

Unary operators have the highest precedence among arithmetic operations:

<!--versetest
Flag:logic = true
Value:int = 1
X:int = 1
Y:int = 2
-->
<!-- 46 -->
```verse
Negative := -Value
Inverted := logic{not Flag=true}
Result := -X * Y    # Unary minus applies to x only
```

## Set Expressions

While Verse emphasizes immutability, practical programming sometimes
requires mutation. Set expressions provide mutation of variables and
fields:

<!--versetest
c := class { var Field:int = 0 }
GetObj()<transacts>:c = c{}
GetArr()<transacts>:[]int = array{1}
GetMap()<transacts>:[string]string = map{ "hi" => "hp" }
Element:int = 5
Value:int = 100
Index:int = 0
Key:string = "key"
MappedValue:string = "value"
assert:
    var X:int = 0
    var Obj:c = GetObj()
    var Arr:[]int = GetArr()
    var Map:[string]string = GetMap()

    set X = 10
    set Obj.Field = Value
    set Arr[Index] = Element
    set Map[Key] = MappedValue
<#
-->
<!-- 47 -->
```verse
set X = 10                    # Variable assignment
set Obj.Field = Value         # Field assignment
set Arr[Index] = Element      # Array element assignment
set Map[Key] = MappedValue    # Map entry assignment
```
<!-- #> -->

Set expressions are themselves expressions that **return the value being
assigned** (the right-hand side). For example, `set Obj.Field = Value`
returns `Value`, not `Obj`. This allows chaining assignments:

```verse
set Y = set X = 5  # Both X and Y become 5
```

Though set expressions have a value, they are typically used for their side
effects. The left-hand side must be a valid LValue—something that can be
assigned to.

Verse supports complex LValues, allowing updates deep within data structures:

<!--versetest
item := class{Name:string = "Item"}
inventory := class{var Items:[]item = array{item{}}}
player := class{var Inventory:inventory = inventory{}}
game := class{var Players:[]player = array{player{}}}
M()<transacts><decides>:void =
    Game:game = game{}
    CurrentPlayer:int = 0
    Slot:int = 0
    NewItem:item = item{}
    set Game.Players[CurrentPlayer].Inventory.Items[Slot] = NewItem
<#
-->
<!-- 48 -->
```verse
set Game.Players[CurrentPlayer].Inventory.Items[Slot] = NewItem
```
<!-- #> -->

## Semicolons vs Commas

Verse uses semicolons and commas as separators in various contexts,
but they have fundamentally different semantics in most
situations. Understanding when each is appropriate is essential for
writing correct Verse code.

**Semicolons** (within parentheses) create *sequences* - they evaluate expressions in order and return the value of the last expression:

<!--versetest
assert:
    Result := (1; 2; 3)
    Result = 3
-->
<!-- 49 -->
```verse
Result := (1; 2; 3)     # Evaluates 1, then 2, then 3; returns 3
# Note: Parentheses are required
# Result := 1; 2         # ERROR: Not valid without parentheses
```

**Commas** (within parentheses) create *tuples* - they group multiple values into a single composite value:

<!--versetest-->
<!-- 50 -->
```verse
Result := (1, 2, 3)     # Creates a tuple of three elements
# Result = (1, 2, 3) (type: tuple(int, int, int))
# Note: Parentheses are required
# Result := 1, 2         # ERROR: Not valid without parentheses
```

### Context-Specific Behavior

In expression contexts (like assignments), semicolons and commas require
parentheses to create sequences and tuples. The distinction is clear when
comparing parenthesized expressions:

<!--versetest-->
<!-- 51 -->
```verse
# Semicolon: sequence (returns last value)
X := (0; 1)              # X = 1, type is int

# Comma: tuple (groups values)
Y := (0, 1)              # Y = (0, 1), type is tuple(int, int)
```

This applies to function return values as well:

<!--versetest-->
<!-- 52 -->
```verse
GetInt():int = (1.0; 2)                    # Returns 2 (int)
GetTuple():tuple(float, int) = (1.0, 2)    # Returns (1.0, 2)
```

Semicolons in argument position create a *sequence that executes
before the call*, with only the last value passed as the argument:

<!--versetest
Process(X:int):void={}
LogEvent(S:string):int=1
-->
<!-- 53 -->
```verse
# Semicolon executes side effects, then passes last value
Process(LogEvent("called"); 42)   # Logs "called", then calls Process(42)

# Equivalent to:
LogEvent("called")
Process(42)
```

This pattern enables side effects in argument position:

<!--versetest
MultiplyByTen(X:int):int = X * 10
-->
<!-- 54 -->
```verse
Result := MultiplyByTen(2; 3)     # Evaluates 2 (discards it), calls Multiply(3)
Result = 30
```

Commas separate distinct arguments in the standard way:

<!--versetest
Add(A:int, B:int):int = A + B
-->
<!-- 55 -->
```verse
Sum := Add(10, 20)                # Two separate arguments
Sum = 30
```

Semicolons are *not allowed* in parameter lists - you must use commas:

<!--versetest
assert_semantic_error(3540):
    InvalidFunc(A:int; B:int):void = {}
-->
<!-- 56 -->
```verse
# VALID: Comma-separated parameters
ValidFunc(A:int, B:int):void = {}

# INVALID: Semicolon in parameters
# InvalidFunc(A:int; B:int):void = {}
```

### In Specific Scopes

Within block expressions (braces), semicolons and commas are interchangeable as separators between definitions:

<!--versetest-->
<!-- 57 -->
```verse
# In block scope, all three separators work:
block:
    X:int = 0; Y:int = 0      # Semicolon separator

block:
    X:int = 0, Y:int = 0      # Comma separator

block:
    X:int = 0                 # Newline separator (most common)
    Y:int = 0
```

In `logic{}` constructor - both semicolons and commas work, but with
different semantics based on the construct's behavior:

<!--versetest-->
<!-- 58 -->
```verse
# Both evaluate all expressions and return logic value
Result1 := logic{true?; true?}    # Sequence of queries
Result2 := logic{true?, true?}    # Also valid
```

In `option{}` constructor - follows the standard sequence vs tuple rule:

<!--versetest-->
<!-- 59 -->
```verse
# Semicolon: sequence, wraps last value
Option1 := option{1; 2}?          # 2

# Comma: tuple, wraps the tuple
Option2 := option{1, 2}?          # (1, 2)
```

In `for` expressions - semicolon typically separates the iteration clause from filter conditions, while commas separate multiple conditions:

<!--versetest-->
<!-- 60 -->
```verse
# Semicolon separates iteration from filter
for (X := 1..3; X <> 2) { X }

# Comma separates multiple filter conditions
for (X := 1..3, X <> 2) { X }      # Same meaning in this context
```

In `array{}` constructors, you can separate elements with commas **or**
semicolons (but not mixed):

<!--versetest-->
<!-- 61 -->
```verse
CommaArray := array{1, 2, 3}       # Commas work
SemiArray := array{1; 2; 3}        # Semicolons also work
# MixedArray := array{1, 2; 3}     # ERROR: Cannot mix separators
```

### Newlines as Separators

In addition to semicolons and commas, **newlines** can serve as
separators in compound expressions and blocks. Newlines behave like
semicolons - they create sequences:

<!--versetest-->
<!-- 62 -->
```verse
# These are equivalent:
Result1 := (1; 2; 3)

Result2 := (
    1
    2
    3
)
# Both return 3
```

## Compound and Block Expressions

Compound expressions, delimited by braces, group multiple expressions
into a single expression. The value of a compound expression is the
value of its last sub-expression:

<!--versetest
ComputeIntermediate():int=3
CalculateAdjustment(o:int):int=3
-->
<!-- 63 -->
```verse
Result := {
    Temp := ComputeIntermediate()
    Adjustment := CalculateAdjustment(Temp)
    Temp + Adjustment
}
```

Compound expressions create new scopes for variables, allowing local bindings that do not affect the enclosing scope:

<!--versetest-->
<!-- 64 -->
```verse
block:
    X := 10    # Local to this block
    Y := 20
    X + Y
               # X and Y no longer accessible
```

You can separate expressions within a compound using semicolons, commas,
or newlines. Semicolons and newlines create sequences (returning the
last value), while commas create tuples. See [Semicolons vs
Commas](#semicolons-vs-commas) for the complete
rules:

<!--versetest
A:int = 1
B:int = 2
C:int = 3
M():void =
    X := { A; B; C }
    Y := { A, B, C }
    Z := {
        A
        B
        C
    }
<#
-->
<!-- 65 -->
```verse
{ A; B; C }           # Semicolon separation (returns C)
{ A, B, C }           # Comma separation (returns tuple (A, B, C))
{                     # Newline separation (returns C)
    A
    B
    C
}
```
<!-- #> -->

## Array Expressions

Array expressions create array values using the `array` keyword
followed by elements in braces:

<!--versetest-->
<!-- 66 -->
```verse
NumArray := array{1, 2, 3, 4, 5}
Empty := array{}
Mixed := array{1, "two", 3.0}  # Mixed types if allowed
```

You can also construct arrays using indented syntax for clarity with
longer lists:

<!--versetest-->
<!-- 67 -->
```verse
Colors := array:
    "red"
    "green"
    "blue"
    "yellow"
```

[^Expressions]: 표현식. evaluation(평가) 된 결괏값을 도출하는 코드를 말합니다.
[^Statements]: 명령문. 동작을 지시하는 코드를 말합니다. 
[^SideEffect]: 부수 효과. 함수 내부에서 이뤄지는 연산이 해당 함수 외부의 상태 변경에 미치는 효과를 말합니다.
[^Literals]: 리터럴. Damage := 100 에서의 100 처럼, identifier 를 사용하지 않고 직접 입력된 값을 말합니다. 만약 identifier 를 사용한다면 Damage := BaseDamage 처럼 표현될 수 있을 것입니다.
[^Identifiers]: 식별자. 특정 코드 요소를 구분하여 가리키기 위해 부여한 이름을 말합니다. 변수의 이름, 클래스의 이름, 함수의 이름 등이 예시가 됩니다.
[^Tuple]: 튜플. 서로 관련된 여러 값을 하나의 값으로 묶어 놓은 것을 말합니다. 예를 들어, X=100, Y=200, Z=300 로 분리된 세 값을 (100, 200, 300) 으로 묶는다면, 이 묶인 값을 Tuple 이라고 할 수 있습니다.
[^Integers]: 정수를 표현할 수 있는 자료형을 말합니다. 1, 10, 100 등을 표현할 수 있습니다.
[^Floats]: 소수를 표현할 수 있는 자료형을 말합니다. 1.618, 3.14 등을 표현할 수 있습니다.
[^Characters]: 단일 문자를 표현할 수 있는 자료형을 말합니다. A, a, ! 등을 표현할 수 있습니다.
[^Strings]: 문자열을 표현할 수 있는 자료형을 말합니다. Hello World, Lorem Ipsum 등을 표현할 수 있습니다.
[^Booleans]: 참 또는 거짓을 표현할 수 있는 자료형을 말합니다.
[^Functions]: 함수를 표현할 수 있는 자료형을 말합니다. 함수는 입력을 받아 결과를 도출하는 코드를 말하지만, Verse 에서는 함수 자체가 값으로 취급될 수 있어서 functions 라는 용어가 자료형 중 하나로도 쓰입니다. 
[^ArbitraryPrecisionArithmetic]: 임의 연산 방식. 저장 공간의 크기를 미리 32비트, 64비트 등으로 제한하지 않고, 필요한 만큼 늘려가며 계산하는 방식을 말합니다. int64 와 같은 자료형은 저장 공간이 64-bit 로 제한되어 있어서, 범위를 넘는 값을 표현하면 Overflow (값이 순환되거나 오류를 발생시키는 현상) 가 발생합니다. 임의 연산 방식을 사용할 경우, 범위가 제한되어 있지 않아 Overflow 발생을 예방할 수 있습니다.
[^StringInterpolation]: 문자열 안에 값을 삽입하는 기능을 말합니다. Print("Found {Items.Quantity} Items !") 라는 코드에서, {Items.Quantity} 부분을 통해 값을 넣는 등을 예로 들 수 있습니다.
[^Persisted]: 프로그램의 실행이 끝나거나 환경이 바뀌어도 계속 유지되도록 저장된 데이터를 말합니다.
[^f64]: 숫자를 처리하는 방식 중 하나를 말합니다. 64bit 중 1bit 는 부호(sign) 표시에, 11bit 는 지수(exponent) 표시에, 나머지 52bit 는 분수로 표현되는 유효숫자(significand) 표시에 사용하는 방식입니다. 32bit 를 사용하는 f32 방식에 비해 소수점 이하 수를 정밀하게 표현할 수 있습니다. 단, 예시에서는 접미사로써 사용되는 것이므로, '앞선 값을 64비트 부동소수점 값으로 취급한다.' 는 의도를 명시적으로 표기하는 효과를 얻는데 그치고, 이 접미사를 적는다고 해서 12.34 라는 값의 소수점 셋째 자리 이하에 없던 값이 추가적으로 생성되는 것은 아닙니다.


