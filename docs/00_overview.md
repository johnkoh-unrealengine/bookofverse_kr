# 프로그래밍 언어 Verse

## 개요

Verse 는 Epic Games에서 Fortnite의 Unreal Editor 게임플레이 제작 및 메타버스 환경 구축을 위해 개발한 다중 패러다임 프로그래밍 언어입니다. 함수형, 논리형, 명령형 프로그래밍 전통을 계승한 Verse는 기존 프로그래밍 언어와는 차별화된, 장기적인 발전과 안정성을 고려하여 설계되었습니다.

Verse 는 세가지 근본적인 원칙을 지켜 만들어졌습니다.:

- **그냥 코드 입니다.**:
다른 언어에서 특별한 구문이나 구조를 필요로 할 수 있는 복잡한 개념들도 Verse에서는 일반적인 코드로 표현됩니다. 마법 같은 것은 없습니다. 모든 것은 동일한 기본 구조로 이루어져 있어 균일하고 예측 가능한 프로그래밍 모델을 제공합니다.

- **한 종류의 언어로 되어 있습니다.**:
하나의 언어 구문이 컴파일에도 쓰이고, 런타임에도 쓰입니다. 둘 사이의 처리 과정 없이 작성 내용이 그대로 실행됩니다.

- **메타버스를 우선시 합니다.**:
Verse는 코드가 단일 글로벌 시뮬레이션인 메타버스에서 실행되는 미래를 위해 설계되었습니다. 이는 강력한 호환성 보장부터 부작용을 추적하고 안전한 동시 실행을 보장하는 이펙트 시스템에 이르기까지 언어의 모든 측면에 영향을 미칩니다.

Verse는 다음과 같은 목표를 가지고 있습니다.:

- **간결할 것.** 프로그래밍을 처음 접하는 사람도 쉽게 배울 수 있을 만큼 간단하며, 규칙이 일관적이고, 예외 처리가 필요한 경우가 최소화되어 있습니다.

- **표현력이 좋을 것.** 정교한 게임 로직과 분산 시스템에 필요한 표현력을 갖추고 있으며, 대규모 코드베이스에 맞춰 확장 가능한 고급 기능을 제공합니다.

- **안전할 것.** 신뢰할 수 없는 코드가 공유 환경에서 실행될 수 있을 만큼 안전하며, 강력한 샌드박싱 및 영향 추적 기능을 제공합니다.

- **빠를 것.** 실시간 게임 및 시뮬레이션에서 충분히 빠르며, 순수 연산을 적극적으로 최적화할 수 있는 구현 방식을 갖추고 있습니다.

- **안정적일 것.** 수십 년 동안 지속될 만큼 안정적이며, 강력한 하위 호환성을 보장하고, 구조 변경에 신중합니다.

**왜 Verse 인가?**

기존 프로그래밍 언어는 수십 년에 걸친 역사적 유산과 설계상의 타협점을 안고 있습니다. Verse는 과거에서 배우되 과거에 얽매이지 않고 새롭게 시작합니다. Verse는 다음과 같은 미래에 대응하기 위해 설계되었습니다.:

- 코드가 영구적인 메타버스 속에서 영원히 존재하는 미래.
- 수백만 명의 개발자가 공유 코드베이스에 기여하는 미래.
- 프로그램이 기본적으로 안전하고, 동시 실행 가능하며, 조합 가능한 미래.
- 하위 호환성이 선택 사항이 아니라 필수 사항인 미래.
- 컴파일 타임과 런타임의 경계가 유동적인 미래.

함께 할 준비가 되셨나요? Verse 의 기본적인 자료형을 이해하시려면 [Built-in Types](02_primitives.md) 부터 확인 해보세요. Verse 의 모든 것이 값을 계산하는 방식을 보시려면 [Expressions](01_expressions.md) 부터 확인하셔도 됩니다.

이미 다른 프로그래밍 언어에 익숙하신 프로그래머 분들께서는 [Failure System](08_failure.md) 과 [Effects](13_effects.md)  를 보시면 Verse 만의 독특한 기능들을 확인하실 수 있습니다.

## 주요 특징

**모든것은 표현식이다.**

Verse 에는 구문이 없습니다. 모든 것은 값을 생성하는 표현식 입니다. 이것이 값이 필요한 모든 곳에서 어떤 코드 조각이든 사용할 수 있는 조합 시스템을 만듭니다.

<!--versetest
Condition()<computes><decides> :void= {}
Array :[]int= array{1}
-->
<!-- 01 -->
```verse
# 심지어 control flow 조차도 값을 생성합니다.
Result := if (Condition[]) then "yes" else "no"

# Loops 도 표현식입니다.
Multiply := for (X : Array) { X * 42 }
```

**Control Flow 로서의 Failure**

Boolean 조건과 예외 대신, Verse 는 failure 를 기본 control flow 매커니즘으로 사용합니다. 표현식은 성공할 수도 있고(이 경우 값을 만듭니다.), 실패할 수도 있으며, 이를 통해 자연스러운 control flow 패턴을 만듭니다.

<!--versetest
ValidateInput(x:string)<computes><decides>:void= {}
ProcessData(x:string)<computes>:void= {}
myclass := class{
Data:string="hi"
M()<decides>:void=
    ValidateInput[Data] # Square brackets indicate that this function may fail
    ProcessData(Data)   # Data is only processed if valid, parentheses mean must succeed
}
<#
-->
<!-- 02 -->
```verse
ValidateInput[Data] # 대괄호는 이 함수가 실패할 수도 있음을 의미합니다.
ProcessData(Data)   # Data 는 유효한 경우에만 처리되며, 괄호는 이 함수가 반드시 성공함을 의미합니다.
```
<!-- #> -->

[Failure](08_failure.md) 장에서는 failure 될 수 있는 표현식과 failure 컨텍스트를 심층적으로 다루고, [Control Flow](07_control.md) 장에서는 if 표현식을 설명합니다.

**추론 기능을 갖춘 강력한 정적 자료형 검사**

Verse는 강력한 자료형 시스템은 컴파일 타임에 오류를 잡아내고, 추론 방식을 적용해 자료형을 수동으로 지정할 필요성을 최소화합니다. 자료형 시스템 및 보조 자료형에 대한 자세한 내용은 [Types](11_types.md)를 참조하세요.

<!--versetest-->
<!-- 03 -->
```verse
X := 42                    # 이전에는 X : int = 42 라고 해야 했으나, 자료형 추론됨.
Name := "Verse"            # 이전에는 Name : str = "Verse" 라고 해야 했으나, 자료형 추론됨.
```

**효과 추적**

함수는 `<computes>`, `<reads>`, `<writes>`, `<transacts>`, `<decides>`, `<suspends>`와 같은 지정자를 통해 부수효과를 선언합니다. 이러한 부수효과 지정자를 통해 함수가 반환 값을 계산하는 것 외에 어떤 작업을 수행할 수 있는지 즉시 명확히 알 수 있습니다.:

<!--versetest
x := class:
    GetCurrentValue()<reads>:int=1
    var Score:int=0
    PureCompute()<computes>:int = 2 + 2            
    ReadState()<reads>:int = GetCurrentValue()     
    UpdateGame()<transacts>:void = set Score += 10 
<#
-->
<!-- 04 -->
```verse
PureCompute()<computes>:int = 2 + 2            # 부수 효과 없음. 즉, 연산만 함
ReadState()<reads>:int = GetCurrentValue()     # 변경 가능한 상태 읽기 가능
UpdateGame()<transacts>:void = set Score += 10 # 읽기, 쓰기, 할당하기 가능
```
<!-- #> -->

[Effects](13_effects.md) 장에서는 효과 시스템에 대한 자세한 내용을 보실 수 있습니다.

**내장된 동시성**

동시성은 구조화된 동시성 기본 요소를 갖춘 핵심 기능으로, 동시 프로그래밍을 안전하고 예측 가능하게 만듭니다.

<!--versetest
TaskA()<suspends>:void={}
TaskB()<suspends>:void={}
TaskC():void={}
FastPath()<suspends>:void={}
SlowButReliablePath()<suspends>:void={}
M()<suspends>:void=
    # Run tasks concurrently and wait for all
    sync:
        TaskA()
        TaskB()
        TaskC()

    # Race tasks and take first result
    race:
        FastPath()
        SlowButReliablePath()
<#
-->
<!-- 05 -->
```verse
# 여러 작업을 동시에 실행하고, 모든 작업이 완료될 때까지 기다립니다.
sync:
    TaskA()
    TaskB()
    TaskC()

# 여러 작업을 동시에 실행하고, 가장 먼저 도출된 결과를 채택합니다.
race:
    FastPath()
    SlowButReliablePath()
```
<!-- #> -->

**예측 실행**

Verse는 코드를 예측 실행하고, 실행에 실패할 경우 변경 사항을 되돌릴 수 있습니다. 이것은 유효성 검사 및 오류 처리에 유연한 패턴을 적용할 수 있도록 돕습니다.

<!--versetest
TryComplexOperation()<computes><decides>:void={}
-->
<!-- 06 -->
```verse
if (TryComplexOperation[]):
    # TryComplexOperation[] 에 의한 변경 사항이 커밋 됩니다.
else:
    # TryComplexOperation[] 에 의한 변경 사항이 자동으로 롤백 됩니다.
```

**실시간 변수를 사용하는 반응형 프로그래밍**

Verse 는 종속성이 변경될 때마다 자동적으로 다시 계산되는 실시간 변수를 사용함으로써, 반응형 프로그래밍을 위한 최고 수준의 지원을 제공합니다. 이는 이벤트 처리를 수동으로 해야 할 부담을 줄여줍니다.

<!--versetest
Log(:string)<transacts>:void={}
-->
<!-- 07 -->
```verse
var MaxHealth:int = 100
var Damage:int = 0
var live Health:int = MaxHealth - Damage

# 종속성이 변경되면 Health 값이 자동으로 업데이트 됩니다.
set Damage = 20      # Health 값이 80 이 됩니다.
set MaxHealth = 150  # Health 값이 130 이 됩니다.

# 이벤트 처리를 위한 반응형 구조
when(Health < 25):
    Log("Low health warning!")
```

Verse provides a foundation for building interactive experiences in persistent virtual environments.

## An Example

The following example demonstrates key language features by building an inventory management system for a game, showing how Verse's constructs create robust, maintainable code.

<!--versetest
# Define item rarity as an enumeration - showing Verse's type system
item_rarity := enum<persistable>:
    common
    uncommon
    rare
    epic
    legendary

# Struct for immutable item data - functional programming style
item_stats := struct<persistable>:
    Damage:float = 0.0
    Defense:float = 0.0
    Weight:float = 1.0
    Value:int = 0

# Class for game items - object-oriented features with functional constraints
game_item := class<final><persistable>:
    Name:string
    Rarity:item_rarity = item_rarity.common
    Stats:item_stats = item_stats{}
    StackSize:int = 1

    # Method with decides effect - can fail
    GetRarityMultiplier()<computes><decides>:float =
        case(Rarity):
            item_rarity.common => 1.0
            item_rarity.uncommon => 1.5
            item_rarity.rare => 2.0
            item_rarity.epic => 3.0
            _ => {false?; 0.0}  # Fails if the item is legendary or unexpected

    # Computed property using closed-world function
    GetEffectiveValue()<reads><decides>:int=
        Floor[Stats.Value * GetRarityMultiplier[]]

# Inventory system with state management and effects
inventory_system := class:
    var Items:[]game_item = array{}
    var MaxWeight:float = 100.0
    var Gold:int = 1000

    # Method demonstrating failure handling and transactional semantics
    AddItem(NewItem:game_item)<transacts><decides>:void =
        # Calculate new weight - speculative execution
        CurrentWeight := GetTotalWeight()
        NewWeight := CurrentWeight + NewItem.Stats.Weight

        # This check might fail, rolling back any changes
        NewWeight <= MaxWeight

        # Only executes if weight check passes
        set Items += array{NewItem}
        Print("Added {NewItem.Name} to inventory")

    # Method with query operator and failure propagation
    RemoveItem(ItemName:string)<transacts><decides>:game_item =
        var RemovedItem:?game_item = false
        var NewItems:[]game_item = array{}

        for (Item : Items):
            if (Item.Name = ItemName, not RemovedItem?):
                set RemovedItem = option{Item}
            else:
                set NewItems += array{Item}
        set Items = NewItems
        RemovedItem?  # Fails if item not found

    # Purchase with complex failure logic and rollback
    PurchaseItem(ShopItem:game_item)<transacts><decides>:void =
        # Multiple failure points - any failure rolls back all changes
        Price := ShopItem.GetEffectiveValue[]
        Price <= Gold  # Fails if not enough gold

        # Tentatively deduct gold
        set Gold = Gold - Price

        # Try to add item - might fail due to weight
        AddItem[ShopItem]

        # All succeeded - changes are committed
        Print("Purchased {ShopItem.Name} for {Price} gold")

    # Higher-order function with type parameters and where clauses
    FilterItems(Predicate:type{_(:game_item)<computes><decides>:void})<reads><decides>:[]game_item =
        for (Item : Items, Predicate[Item]):
            Item

    GetTotalWeight()<transacts>:float =
        var Total:float = 0.0
        for (Item : Items):
            set Total += Item.Stats.Weight
        Total

# Player class using composition
player_character := class:
    Name:string
    var Level:int = 1
    var Experience:int = 0
    var Inventory:inventory_system = inventory_system{}

    LevelUpThreshold:int = 100

    GainExperience(Amount:int)<transacts>:void =
        set Experience += Amount

        # Automatic level up check with failure handling
        loop:
            RequiredXP := LevelUpThreshold * Level
            if (Experience >= RequiredXP):
                set Experience -= RequiredXP
                set Level += 1
                Print("{Name} leveled up to {Level}!")
            else:
                break

    # Method showing qualified access
    EquipStarterGear()<transacts><decides>:void =
        StarterSword := game_item{
            Name := "Rusty Sword"
            Rarity := item_rarity.common
            Stats := item_stats{Damage := 10.0, Weight := 5.0, Value := 50}
        }
        # These might fail if inventory is full
        Inventory.AddItem[StarterSword]

# Example usage demonstrating control flow and failure handling
assert:
    # Create a player (can't fail)
    Hero := player_character{Name := "Verse Hero"}

    # Try to equip starter gear (might fail)
    if (Hero.EquipStarterGear[]):
        Print("Hero equipped with starter gear")

    # Demonstrate transactional behavior
    ExpensiveItem := game_item{
        Name := "Golden Crown"
        Rarity := item_rarity.epic
        Stats := item_stats{Value := 2000, Weight := 90.0}  # Very heavy!
    }

    # This might fail due to weight or insufficient gold
    if (Hero.Inventory.PurchaseItem[ExpensiveItem]):
        Print("Purchase successful!")
    else:
        Print("Purchase failed - gold remains at {Hero.Inventory.Gold}")

    # Use higher-order functions with nested function predicate
    IsRareOrLegendary(I:game_item)<computes><decides>:void =
        I.Rarity = item_rarity.rare or I.Rarity = item_rarity.legendary

    RareItems := Hero.Inventory.FilterItems[IsRareOrLegendary]

    Print("Found {RareItems.Length} rare items")
<#
-->
<!-- 08 -->
```verse
# Module declaration - start by importing utility functions
using { /Verse.org/VerseCLR }

# Define item rarity as an enumeration - showing Verse's type system
item_rarity := enum<persistable>:
    common
    uncommon
    rare
    epic
    legendary

# Struct for immutable item data - functional programming style
item_stats := struct<persistable>:
    Damage:float = 0.0
    Defense:float = 0.0
    Weight:float = 1.0
    Value:int = 0

# Class for game items - object-oriented features with functional constraints
game_item := class<final><persistable>:
    Name:string
    Rarity:item_rarity = item_rarity.common
    Stats:item_stats = item_stats{}
    StackSize:int = 1

    # Method with decides effect - can fail
    GetRarityMultiplier()<decides>:float =
        case(Rarity):
            item_rarity.common => 1.0
            item_rarity.uncommon => 1.5
            item_rarity.rare => 2.0
            item_rarity.epic => 3.0
            _ => {false?; 0.0}  # Fails if the item is legendary or unexpected

    # Computed property using closed-world function
    GetEffectiveValue()<reads><decides>:int=
        Floor[Stats.Value * GetRarityMultiplier[]]

# Inventory system with state management and effects
inventory_system := class:
    var Items:[]game_item = array{}
    var MaxWeight:float = 100.0
    var Gold:int = 1000

    # Method demonstrating failure handling and transactional semantics
    AddItem(NewItem:game_item)<transacts><decides>:void =
        # Calculate new weight - speculative execution
        CurrentWeight := GetTotalWeight()
        NewWeight := CurrentWeight + NewItem.Stats.Weight

        # This check might fail, rolling back any changes
        NewWeight <= MaxWeight

        # Only executes if weight check passes
        set Items += array{NewItem}
        Print("Added {NewItem.Name} to inventory")

    # Method with query operator and failure propagation
    RemoveItem(ItemName:string)<transacts><decides>:game_item =
        var RemovedItem:?game_item = false
        var NewItems:[]game_item = array{}

        for (Item : Items):
            if (Item.Name = ItemName, not RemovedItem?):
                set RemovedItem = option{Item}
            else:
                set NewItems += array{Item}
        set Items = NewItems
        RemovedItem?  # Fails if item not found

    # Purchase with complex failure logic and rollback
    PurchaseItem(ShopItem:game_item)<transacts><decides>:void =
        # Multiple failure points - any failure rolls back all changes
        Price := ShopItem.GetEffectiveValue[]
        Price <= Gold  # Fails if not enough gold

        # Tentatively deduct gold
        set Gold = Gold - Price

        # Try to add item - might fail due to weight
        AddItem[ShopItem]

        # All succeeded - changes are committed
        Print("Purchased {ShopItem.Name} for {Price} gold")

    # Higher-order function with type parameters and where clauses
    FilterItems(Predicate:type{_(:game_item)<computes><decides>:void})<reads><decides>:[]game_item =
        for (Item : Items, Predicate[Item]):
            Item

    GetTotalWeight()<transacts>:float =
        var Total:float = 0.0
        for (Item : Items):
            set Total += Item.Stats.Weight
        Total

# Player class using composition
player_character<public> := class:
    Name<public>:string
    var Level:int = 1
    var Experience:int = 0
    var Inventory:inventory_system = inventory_system{}

    LevelUpThreshold := 100

    GainExperience(Amount:int)<transacts>:void =
        set Experience += Amount

        # Automatic level up check with failure handling
        loop:
            RequiredXP := LevelUpThreshold * Level
            if (Experience >= RequiredXP):
                set Experience -= RequiredXP
                set Level += 1
                Print("{Name} leveled up to {Level}!")
            else:
                break

    # Method showing qualified access
    EquipStarterGear()<transacts><decides>:void =
        StarterSword := game_item{
            Name := "Rusty Sword"
            Rarity := item_rarity.common
            Stats := item_stats{Damage := 10.0, Weight := 5.0, Value := 50}
        }
        # These might fail if inventory is full
        Inventory.AddItem[StarterSword]

# Example usage demonstrating control flow and failure handling
RunExample<public>()<suspends>:void =
    # Create a player (can't fail)
    Hero := player_character{Name := "Verse Hero"}

    # Try to equip starter gear (might fail)
    if (Hero.EquipStarterGear[]):
        Print("Hero equipped with starter gear")

    # Demonstrate transactional behavior
    ExpensiveItem := game_item{
        Name := "Golden Crown"
        Rarity := item_rarity.epic
        Stats := item_stats{Value := 2000, Weight := 90.0}  # Very heavy!
    }

    # This might fail due to weight or insufficient gold
    if (Hero.Inventory.PurchaseItem[ExpensiveItem]):
        Print("Purchase successful!")
    else:
        Print("Purchase failed - gold remains at {Hero.Inventory.Gold}")

    # Use higher-order functions with nested function predicate
    IsRareOrLegendary(I:game_item)<computes><decides>:void =
        I.Rarity = item_rarity.rare or I.Rarity = item_rarity.legendary

    RareItems := Hero.Inventory.FilterItems[IsRareOrLegendary]

    Print("Found {RareItems.Length} rare items")
```
<!-- #> -->

This example demonstrates Verse in a practical context. Let's explore what makes this code uniquely Verse:

**Type System and Data Modeling**

The example begins with Verse's rich type system. Types flow naturally through the code; many type annotations are omitted as they can be inferred. When we do specify types, like `Items:[]game_item`, they document intent rather than just satisfy the compiler. The `item_rarity` enum provides type-safe constants without the boilerplate of traditional enumerations. The `item_stats` struct marked as `<persistable>` can be saved and loaded from persistent storage, essential for game saves. The `game_item` class is marked `<final>` and `<persistable>` so its instances can be saved and restored; because persistable data is serialized by value, such classes cannot also be `<unique>`.

**Failure as Control Flow**

Throughout the code, failure drives control flow rather than exceptions or error codes. The `<decides>` effect marks functions that can fail, and failure propagates naturally through expressions. When `GetRarityMultiplier()` encounters an unknown rarity, it does not throw an exception or return a sentinel value - it simply fails, and the calling code handles this gracefully.
The `AddItem` method demonstrates how failure creates declarative validation. The expression `NewWeight <= MaxWeight` either succeeds (allowing execution to continue) or fails (preventing the item from being added). There's no explicit control flow - just a declarative assertion of what must be true.

**Transactional Semantics and Speculative Execution**

Methods marked with `<transacts>` provide automatic rollback on failure. In `PurchaseItem`, we deduct gold from the player, then try to add the item. If adding fails (perhaps due to weight limits), the gold deduction is automatically rolled back. This eliminates entire categories of bugs related to partial state updates.
This transactional behavior extends to complex operations. When multiple changes need to succeed or fail together, Verse ensures consistency without need for manual clean up.

**Functions as First-Class Values**

The `FilterItems` method accepts a predicate function, demonstrating higher-order programming. The nested function `IsRareOrLegendary` in `RunExample` shows how functions can be defined locally and passed around like any other value. This functional programming style combines naturally with the imperative and object-oriented features.

**Optional Types and Query Operators**

The inventory removal logic uses optional types (`?game_item`) to represent values that might not exist. The query operator `?` extracts values from options, failing if the option is empty. This eliminates null pointer exceptions while providing convenient syntax for handling absent values.

**Pattern Matching and Control Flow**

The `case` expression in `GetRarityMultiplier` demonstrates pattern matching. Unlike a switch statement, `case` is an expression that produces a value. The underscore `_` provides a catch-all pattern, though in this example it leads to failure.
The `if` expression similarly produces values and can bind variables in its condition. The compound conditions show how multiple operations can be chained with automatic failure propagation.

**Module System and Access Control**

The code begins with `using` statements that import functionality from other modules. The path-based module system ensures that dependencies are unambiguous and permanently addressable. Access specifiers like `<public>` control visibility at a fine-grained level.

**Immutable by Default**

Data structures are immutable unless explicitly marked with `var`. This eliminates large classes of bugs and makes concurrent programming safer. When we do need mutation, it is explicit and tracked by the effect system. See [Mutability](05_mutability.md) for complete details on `var` and `set`.

## Naming Conventions

Verse has a set of naming conventions that make code readable and predictable. While the language does not enforce these conventions, following them ensures your code integrates well with the broader Verse ecosystem and is immediately familiar to other Verse developers.

Identifiers should be in PascalCase (CamelCase starting with uppercase):

<!--versetest
player_record := struct:
    Name:string

PlayerDatabase(Id:int)<decides>:player_record =
    if (Id = 0):
        player_record{Name := "Alice"}
    else if (Id = 1):
        player_record{Name := "Bob"}
    else:
        false?
        player_record{Name := ""}
-->
<!-- 09 -->
```verse
# Variables and constants use PascalCase
PlayerHealth:int = 100
MaxInventorySize:int = 50
IsGameActive:logic = true

# Functions use PascalCase
CalculateDamage(Base:float, Multiplier:float):float =
    Base * Multiplier

GetPlayerName(Id:int)<decides>:string =
    PlayerDatabase[Id].Name

# Classes and structs use snake_case
player_character := class:
    Name:string
    Level:int

inventory_item := struct:
    ItemId:int
    Quantity:int

# Enums and their values use snake_case
game_state := enum:
    main_menu
    in_game
    paused
    game_over
```

Generic type parameters use single lowercase letters or short descriptive names:

<!--versetest-->
<!-- 10 -->
```verse
# Single letter for simple generics
Find(Array:[]t, Target:t where t:type):?int = false

# Descriptive names for complex relationships
Transform(Input:in_t, Processor:type{_(:in_t):out_t} where in_t:type, out_t:type):?out_t = false
```


Module names always use PascalCase, including path segments:

<!--NoCompile-->
<!-- 11 -->
```verse
# Module definition
InventorySystem := module:
    # Module contents

# Path segments also use PascalCase
using { /Fortnite.com/Characters/PlayerController }
using { /MyGame.com/Systems/CombatSystem }
using { /Verse.org/Random }
```

Class and struct fields use PascalCase, and methods follow the same PascalCase convention as functions:

<!--versetest-->
<!-- 12 -->
```verse
player := class:
    Name:string          # PascalCase for fields
    var Health:float= 0.0

    # Methods use PascalCase like functions
    TakeDamage(Amount:float):void =
        set Health = Max(0.0, Health - Amount)

    IsAlive():logic =
        logic{Health > 0.0}
```

## Code Formatting

Verse code follows consistent formatting patterns to emphasize readability.

Use four spaces to indent code blocks. The colon introduces a block, with subsequent lines indented:

<!--versetest
Condition()<decides><transacts>:void = {}
DoSomething()<transacts>:void = {}
DoSomethingElse()<transacts>:void = {}
Inventory:[]int = array{1, 2, 3}
ProcessItem(Item:int)<transacts>:void = {}
UpdateDisplay()<transacts>:void = {}
ImplementationHere()<transacts>:void = {}

-->
<!-- 13 -->
```verse
if (Condition[]):
    DoSomething()
    DoSomethingElse()

for (Item : Inventory):
    ProcessItem(Item)
    UpdateDisplay()

class_definition := class:
    Field1:int
    Field2:string

    Method():void =
        ImplementationHere()
```

Complex expressions benefit from clear formatting that shows structure:

<!--versetest
player_type := struct{Health:int = 75}
BaseDamage:float = 100.0
LevelMultiplier:float = 1.5
BonusPercentage:float = 10.0
rarity_type := enum{common; uncommon; rare; epic; legendary}
-->
<!-- 14 -->
```verse
Player:player_type = player_type{}
Rarity:rarity_type = rarity_type.rare

# Multi-line conditionals
Result := if (Player.Health > 50):
    "healthy"
else if (Player.Health > 20):
    "injured"
else:
    "critical"

# Chained operations with clear precedence
FinalDamage :=
    BaseDamage *
    LevelMultiplier *
    (1.0 + BonusPercentage / 100.0)

# Pattern matching with aligned cases
DamageMultiplier := case(Rarity):
    rarity_type.common => 1.0
    rarity_type.uncommon => 1.5
    rarity_type.rare => 2.0
    rarity_type.epic => 3.0
    rarity_type.legendary => 5.0
```

Functions follow a consistent pattern with effects and return types clearly specified:

<!--versetest
difficulty_level := enum{easy; medium; hard}
ValidateAmount(Amount:int)<transacts><decides>:void = {}
DeductBalance(Amount:int)<transacts>:void = {}
RecordTransaction()<transacts>:void = {}
GetBaseReward(Difficulty:difficulty_level)<decides>:?int = option{100}
CalculateTimeBonus(CompletionTime:float):int = 50
-->
<!-- 15 -->
```verse
# Simple pure function
Add(X:int, Y:int)<computes>:int = X + Y

# Function with effects
ProcessTransaction(Amount:int)<transacts><decides>:void =
    ValidateAmount[Amount]
    DeductBalance(Amount)
    RecordTransaction()

# Multi-line function with clear structure
CalculateReward(
    PlayerLevel:int,
    Difficulty:difficulty_level,
    CompletionTime:float
)<decides>:int =
    BaseReward := GetBaseReward[Difficulty]?
    LevelBonus := PlayerLevel * 10
    TimeBonus := CalculateTimeBonus(CompletionTime)
    BaseReward + LevelBonus + TimeBonus
```

## Comments

Comments are ignored during execution but help with understanding and maintaining code. Verse offers several styles of comments to suit different documentation needs. The simplest is the single-line comment, which begins with `#` and continues to the end of the line:

<!--versetest-->
<!-- 16 -->
```verse
CalculateDamage := 100 * 1.5   # Apply critical hit multiplier
```

When you need to document something within a line of code without breaking it up, inline block comments provide the perfect solution. These are enclosed between `<#` and `#>`:

<!--versetest
BaseValue:int = 100
Multiplier:int = 2
Bonus:int = 10
-->
<!-- 17 -->
```verse
Result := BaseValue <# original amount #> * Multiplier <# scaling factor #> + Bonus
```

The same can be used to write multi-line block comments, making them ideal for explaining complex algorithms or providing detailed context:

<!--versetest-->
<!-- 18 -->
```verse
<# This function implements the quadratic damage falloff formula
   used throughout the game. The falloff ensures that damage
   decreases smoothly with distance, creating strategic positioning
   choices for players. #>
CalculateFalloffDamage(Distance:float, MaxDamage:float):float =
    MaxDamage  # Implementation here
```

Block comments nest, which allows you to temporarily disable code that already contains comments without having to remove or modify existing documentation:

<!--versetest-->
<!-- 19 -->
```verse
<# Temporarily disabled for testing
   OriginalFunction()  <# This had a bug #>
   NewFunction()       # Testing this approach
#>
```

Indented comments begin with a `<#>` on its own line; everything indented by four spaces on subsequent lines becomes part of the comment:

<!--versetest
DoSomething():void = {}
-->
<!-- 20 -->
```verse
<#>
    This entire block is a comment because it is indented.
    It provides a clean way to write longer documentation
    without cluttering each line with comment markers.

DoSomething()  # Not part of the comment.
```

## Syntactic Styles

Verse offers flexible syntax to accommodate different programming styles. The same logic can be expressed using braces, indentation, or inline forms, allowing you to choose the clearest representation for each context.

The braced style uses curly braces to delimit blocks, familiar from C-family languages:

<!--versetest
Score:int = 85
-->
<!-- 21 -->
```verse
Result := if (Score > 90) {
    "excellent"
} else if (Score > 70) {
    "good"
} else {
    "needs improvement"
}
```

The indented style uses colons and indentation to define structure, similar to Python:

<!--versetest
Score:int = 85
-->
<!-- 22 -->
```verse
Result := if (Score > 90):
    "excellent"
else if (Score > 70):
    "good"
else:
    "needs improvement"
```

For simple expressions, the inline style keeps everything on one line:

<!--versetest
Score:int = 85
-->
<!-- 23 -->
```verse
Result := if (Score > 90) then "excellent" else if (Score > 70) then "good" else "needs improvement"
```

The dotted style uses a period to introduce the expression:

<!--versetest
Score:int = 85
-->
<!-- 24 -->
```verse
Result := if (Score > 90). "excellent" else if (Score > 70). "good" else. "needs improvement"
```

You can even mix styles when it makes sense:

<!--versetest
ComplexCondition()<transacts><decides>:void = {}
AnotherCheck()<transacts><decides>:void = {}
YetAnotherValidation()<transacts><decides>:void = {}
-->
<!-- 25 -->
```verse
Result := if:
    ComplexCondition[] and
    AnotherCheck[] and
    YetAnotherValidation[]
then { "condition met" } else { "condition not met" }
```

All these forms produce the same result. The choice between them is about readability and context. 
Use braces when working with existing brace-heavy code, indentation for cleaner vertical layouts,
and inline forms for simple expressions. This flexibility lets you write code that reads naturally.
