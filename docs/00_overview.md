# 프로그래밍 언어 Verse

## 개요

Verse 는 Epic Games에서 Fortnite의 Unreal Editor 게임플레이 제작 및 메타버스 환경 구축을 위해 개발한 다중 패러다임 프로그래밍 언어입니다. 함수형, 논리형, 명령형 프로그래밍 전통을 계승한 Verse는 기존 프로그래밍 언어와는 차별화된, 장기적인 발전과 안정성을 고려하여 설계되었습니다.

Verse 는 세가지 근본적인 원칙을 지켜 만들어졌습니다 :

- **그냥 코드 입니다** :
다른 언어에서 특별한 구문이나 구조를 필요로 할 수 있는 복잡한 개념들도 Verse에서는 일반적인 코드로 표현됩니다. 마법 같은 것은 없습니다. 모든 것은 동일한 기본 구조로 이루어져 있어 균일하고 예측 가능한 프로그래밍 모델을 제공합니다.

- **한 종류의 언어로 되어 있습니다** :
하나의 언어 구문이 컴파일에도 쓰이고, 런타임에도 쓰입니다. 둘 사이의 처리 과정 없이 작성 내용이 그대로 실행됩니다.

- **메타버스를 우선시 합니다** :
Verse는 코드가 단일 글로벌 시뮬레이션인 메타버스에서 실행되는 미래를 위해 설계되었습니다. 이는 강력한 호환성 보장부터 부작용을 추적하고 안전한 동시 실행을 보장하는 이펙트 시스템에 이르기까지 언어의 모든 측면에 영향을 미칩니다.

Verse는 다음과 같은 목표를 가지고 있습니다 :

- **간결할 것.** 프로그래밍을 처음 접하는 사람도 쉽게 배울 수 있을 만큼 간단하며, 규칙이 일관적이고, 예외 처리가 필요한 경우가 최소화되어 있습니다.

- **표현력이 좋을 것.** 정교한 게임 로직과 분산 시스템에 필요한 표현력을 갖추고 있으며, 대규모 코드베이스에 맞춰 확장 가능한 고급 기능을 제공합니다.

- **안전할 것.** 신뢰할 수 없는 코드가 공유 환경에서 실행될 수 있을 만큼 안전하며, 강력한 샌드박싱 및 영향 추적 기능을 제공합니다.

- **빠를 것.** 실시간 게임 및 시뮬레이션에서 충분히 빠르며, 순수 연산을 적극적으로 최적화할 수 있는 구현 방식을 갖추고 있습니다.

- **안정적일 것.** 수십 년 동안 지속될 만큼 안정적이며, 강력한 Backward Compatibility[^BackwardCompatibility]를 보장하고, 구조 변경에 신중합니다.

**왜 Verse 인가?**

기존 프로그래밍 언어는 수십 년에 걸친 역사적 유산과 설계상의 타협점을 안고 있습니다. Verse는 과거에서 배우되 과거에 얽매이지 않고 새롭게 시작합니다. Verse는 향후 변화할 프로그래밍의 다음과 같은 특징들에 대응하기 위해 설계되었습니다:

- 코드가 영구적인 메타버스 속에 영원히 잔류한다.
- 수백만 명의 개발자가 공유 코드베이스에 기여한다.
- 프로그램이 기본적으로 안전하고, 동시 실행 가능하며, 조합 가능해진다.
- 하위 호환성 지원이 선택이 아니라 필수가 된다.
- 컴파일 타임과 런타임의 경계가 유연해진다.

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

**Control Flow[^ControlFlow] 로서의 Failure**

Boolean 조건과 예외 대신, Verse 는 failure 를 기본 control flow 매커니즘으로 사용합니다. 표현식은 성공할 수도 있고(이 경우 값을 도출합니다), 실패할 수도 있으며, 이를 통해 자연스러운 control flow 패턴을 만듭니다.

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
ValidateInput[Data]  # 대괄호는 이 함수가 실패할 수도 있음을 의미합니다.
ProcessData(Data)    # 유효한 Data만 처리되며, 괄호는 이 함수가 반드시 성공함을 의미합니다.
```
<!-- #> -->

[Failure](08_failure.md) 장에서는 failure 될 수 있는 표현식과 failure 컨텍스트를 심층적으로 다루고, [Control Flow](07_control.md) 장에서는 if 표현식을 설명합니다.

**추론 기능을 갖춘 강력한 정적 자료형 검사**

Verse는 강력한 자료형 시스템은 컴파일 타임에 오류를 잡아내고, 추론 방식을 적용해 자료형을 수동으로 지정할 필요성을 최소화합니다. 자료형 시스템 및 보조 자료형에 대한 자세한 내용은 [Types](11_types.md)를 참조하세요.

<!--versetest-->
<!-- 03 -->
```verse
X := 42                    # X : int = 42 로 적지 않아도 자료형이 추론됩니다.
Name := "Verse"            # Name : str = "Verse" 로 적지 않아도 자료형이 추론됩니.
```

**Effect 추적**

함수는 `<computes>`, `<reads>`, `<writes>`, `<transacts>`, `<decides>`, `<suspends>`와 같은 Specifiers[^Specifiers] 를 통해 부수 효과를 선언합니다. 이러한 Effect Specifiers 를 보면 함수가 반환 값을 계산하는 것 외에 어떤 작업을 수행할 수 있는지 빠르고 명확하게 알 수 있습니다:

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
PureCompute()<computes>:int = 2 + 2              # 부수 효과가 없습니다. 즉, 내부 연산만 합니다.
ReadState()<reads>:int = GetCurrentValue()       # 변경 가능한 상태를 읽을 수 있습니다.
UpdateGame()<transacts>:void = set Score += 10   # 읽고, 쓰고, 할당할 수 있습니다.
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

**Speculative Execution[^SpeculativeExecution]**

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

Verse 는 영속적 가상 환경에서의 상호 경험을 구축하기 위한 기반을 제공합니다.

## 예시

다음 예시는 게임용 인벤토리 관리 시스템을 구축하여 Verse 의 주요 언어 기능을 보여주고, Verse 의 구문이 어떻게 견고하고 유지보수 가능한 코드를 생성하는지 설명합니다.

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
# 모듈 선언 - 아래처럼 유틸리티 함수를 가져오는 것부터 시작합니다
using { /Verse.org/VerseCLR }

# Enumeration(이는 Verse 의 자료형 시스템을 보여줍니다)으로 게임 내 아이템 희귀도를 정의합니다
item_rarity := enum<persistable>:
    common
    uncommon
    rare
    epic
    legendary

# 게임 내 아이템의 불변 데이터 (여기서는 item 의 stats) 를 위한 구조체를 정의합니다 - 이는 함수형 프로그래밍 스타일의 예시 입니다
item_stats := struct<persistable>:
    Damage:float = 0.0
    Defense:float = 0.0
    Weight:float = 1.0
    Value:int = 0

# 게임 내 아이템의 Class 를 정의합니다 - 이는 함수적 제약 조건을 갖는 객체 지향 기능의 예시 입니다
game_item := class<final><persistable>:
    Name:string
    Rarity:item_rarity = item_rarity.common
    Stats:item_stats = item_stats{}
    StackSize:int = 1

    # <decides> 효과를 갖는 Method - 이는 연산 결과 fail 될 수 있습니다
    GetRarityMultiplier()<decides>:float =
        case(Rarity):
            item_rarity.common => 1.0
            item_rarity.uncommon => 1.5
            item_rarity.rare => 2.0
            item_rarity.epic => 3.0
            _ => {false?; 0.0}   # 해당 아이템의 rarity 가 legendary 이거나 규격 외인 경우 Fail 됩니다

    # Closed-world 함수(연산 대상이 정해져 있고, 그것이 확장되지 않는다고 전제되는 함수)를 사용해 계산된 속성
    GetEffectiveValue()<reads><decides>:int=
        Floor[Stats.Value * GetRarityMultiplier[]]

# 상태 관리 및 효과를 갖춘 inventory_system 라는 이름의 Class 를 선언합니다
inventory_system := class:
    var Items:[]game_item = array{}
    var MaxWeight:float = 100.0
    var Gold:int = 1000

    # 오류 처리 및 트랜잭션 의미론을 보여주는 Method
    AddItem(NewItem:game_item)<transacts><decides>:void =
        # 새 무게를 계산합니다 - 이는 Speculative Execution (투기적 실행) 의 예시입니다
        CurrentWeight := GetTotalWeight()
        NewWeight := CurrentWeight + NewItem.Stats.Weight

        # 위 검사는 fail 될 수 있고, 그 경우 아래와 같이 모든 변경 사항을 되돌립니다
        NewWeight <= MaxWeight

        # 아래 연산은 weight 검사가 통과된 경우에만 실행됩니다
        set Items += array{NewItem}
        Print("Added {NewItem.Name} to inventory")

    # 쿼리 연산자와 오류 전파를 사용하는 Method
    RemoveItem(ItemName:string)<transacts><decides>:game_item =
        var RemovedItem:?game_item = false
        var NewItems:[]game_item = array{}

        for (Item : Items):
            if (Item.Name = ItemName, not RemovedItem?):
                set RemovedItem = option{Item}
            else:
                set NewItems += array{Item}
        set Items = NewItems
        RemovedItem?  # 아이템이 탐색되지 않으면 Fail 됩니다

    # 복잡한 오류 로직 및 롤백 기능을 갖춘 아이템 구매 기능
    PurchaseItem(ShopItem:game_item)<transacts><decides>:void =
        # 여러 failure 가 발생할 수 있는 지점 - failure 가 하나라도 발생하면 모든 변경 사항을 되돌립니다
        Price := ShopItem.GetEffectiveValue[]
        Price <= Gold  # 골드가 충분하지 않으면 Fail 됩니다

        # 잠정적으로 골드를 차감합니다
        set Gold = Gold - Price

        # 아이템 추가를 시도합니다 - 이는 무게에 의해 실패할 수 있습니다
        AddItem[ShopItem]

        # 모든 과정에 성공한 경우 - 변경사항이 커밋 됩니다
        Print("Purchased {ShopItem.Name} for {Price} gold")

    # 자료형 파라미터와 자료형 제약 구문을 사용하는 고차 함수
    FilterItems(Predicate:type{_(:game_item)<computes><decides>:void})<reads><decides>:[]game_item =
        for (Item : Items, Predicate[Item]):
            Item

    GetTotalWeight()<transacts>:float =
        var Total:float = 0.0
        for (Item : Items):
            set Total += Item.Stats.Weight
        Total

# 컴포지션을 사용하는(이 경우엔 inventory_system 이라는 이름의 Class 를 포함함) player_character 라는 이름의 Class 를 선언합니다
player_character<public> := class:
    Name<public>:string
    var Level:int = 1
    var Experience:int = 0
    var Inventory:inventory_system = inventory_system{}

    LevelUpThreshold := 100

    GainExperience(Amount:int)<transacts>:void =
        set Experience += Amount

        # 오류 처리 기능을 갖춘 자동 레벨업 확인 기능
        loop:
            RequiredXP := LevelUpThreshold * Level
            if (Experience >= RequiredXP):
                set Experience -= RequiredXP
                set Level += 1
                Print("{Name} leveled up to {Level}!")
            else:
                break

    # 적격 접근을 보여주는 Method
    EquipStarterGear()<transacts><decides>:void =
        StarterSword := game_item{
            Name := "Rusty Sword"
            Rarity := item_rarity.common
            Stats := item_stats{Damage := 10.0, Weight := 5.0, Value := 50}
        }
        # 아래 로직은 인벤토리가 가득 찬 경우 fail 될 수 있습니다
        Inventory.AddItem[StarterSword]

# Control Flow 와 오류 처리를 보여주는 사용 예시
RunExample<public>()<suspends>:void =
    # player 를 생성합니다 (이는 fail 될 수 없습니다)
    Hero := player_character{Name := "Verse Hero"}

    # StarterGear 장착을 시도합니다 (이는 fail 될 수 있습니다)
    if (Hero.EquipStarterGear[]):
        Print("Hero equipped with starter gear")

    # Transactional (여러 작업을 하나의 논리적 단위로 취급하는) 작동 방식을 보여줍니다
    ExpensiveItem := game_item{
        Name := "Golden Crown"
        Rarity := item_rarity.epic
        Stats := item_stats{Value := 2000, Weight := 90.0}  # 매우 무겁습니다!
    }

    # 무게 또는 골드 부족 때문에 fail 될 수 있습니다
    if (Hero.Inventory.PurchaseItem[ExpensiveItem]):
        Print("Purchase successful!")
    else:
        Print("Purchase failed - gold remains at {Hero.Inventory.Gold}")

    # Predicate(조건자)와 함께 고차 함수를 사용하고 있습니다
    IsRareOrLegendary(I:game_item)<computes><decides>:void =
        I.Rarity = item_rarity.rare or I.Rarity = item_rarity.legendary

    RareItems := Hero.Inventory.FilterItems[IsRareOrLegendary]

    Print("Found {RareItems.Length} rare items")
```
<!-- #> -->

이 예시는 Verse를 실제 상황에서 사용하는 방법을 보여줍니다. 이 코드가 Verse만의 특징을 갖는 이유를 살펴보겠습니다:

**자료형 시스템과 데이터 모델링**

이 예시는 Verse 의 풍부한 자료형 시스템으로부터 시작합니다. 코드 전체에 걸쳐 자료형이 자연스럽게 흐르도록 설계되었으며, 많은 type annotation[^TypeAnnotation] 들은 추론이 가능하므로 생략되었습니다. `Items:[]game_item`처럼 자료을 명시한 부분은 컴파일러의 요구 사항을 충족하기 위해서가 아니라 의도를 문서로 남기기 위해 그렇게 했습니다. `item_rarity` 라고 명명된 enum[^enumeration] 은 기존 enum 에서 흔히 사용되는 boilerplate[^Boilerplate] 없이 자료형 안전성이 보장되는 상수를 제공합니다. `<persistable>`로 표시된 `item_stats` 구조는 영구 저장소에 저장하고 불러올 수 있어 게임 저장에 필수적입니다. `game_item` class 는 인스턴스를 저장하고 복원하려는 의도를 담아 `<final>` 및 `<persistable>`로 표시 해두었습니다. 영구 저장소에 저장된 데이터는 값을 기준으로 serialize [^Serialize] 되므로, 이러한 class 는 `<unique>` 속성을 가질 수 없습니다.

**Control Flow 로서의 Failure**

코드 전체에 걸쳐서, 예외 코드나 에러 코드가 아닌 failure 가 control flow 를 주합니다. `<decides>` effect 는 fail 될 수 있는 함수를 표시하고, failure 는 표현식을 통 자연스럽게 전파됩니다. `GetRarityMultiplier()` 함수는 사전에 정의되지 않은 희귀도 값을 맞닥뜨리면 예외를 발생시거나 sentinel 값을 반환하지 않고 단순히 fail 되며, 이를 호출한 코드가 그것을 적절하게[^Gracefully] 처리합니다. `AddItem` Method[^Method] 는 failure 가 어떻게 선언적 유효성 검사 구조를 생성하는지 보여줍니다. `NewWeight <= MaxWeight` 표현식은 성공(이 경우 실행 절차를 계속 합니다)할 수도 있고 실패(이 경우 아이템이 추가되는 것을 막습니다)할 수도 있습니다. 명시적인 control flow 는 없고, 단지 무엇이 참이어야 하는지에 대한 선언적 assertion[^Assertion] 이 있을 뿐입니다.

**트랜젝션 의미론과 Speculative Execution**

`<transacts>` 라고 마크 되어 있는 Method 들은 failure 시에 자동으로 롤백 됩니다. `PurchaseItem` 에서 우리는 플레이어로부터 골드를 차감한 뒤, 구매한 아이템을 인벤토리에 추가할 것을 시도 했습니다. 만약 추가에 실패한다면(아마도 무게 제한 때문일 것입니다), 골드 차감 연산은 자동으로 롤백 됩니다. 이는 부분적 상태 갱신과 관련한 버그 발생 가능성을 사전에 모두 제거하는 효과를 갖습니다. 이러한 transactional[^Transactional] 연산 방식은 복잡한 작업에까지 확장됩니다. 여러 변경 사항이 동시에 성공하거나 실패해야 하는 상황에서, Verse는 수동 뒤처리 작업 없이도 연산의 일관성을 보장합니다.

**First-Class Values[^FirstClassValues] 로써의 함수**

`FilterItems` method 는 predicate function[^PredicateFunction] 을 인수로 받고 있는데, 이는 고차 프로그래밍의 예가 됩니다. `RunExample` 내부에 있는 nested function[^NestedFunction] 인 `IsRareOrLegendary`는 '함수'가 어떻게 다른 '값'들과 마찬가지로 locally[^Locally] 정의되고, 전달될 수 있는지 보여줍니다. 이러한 함수형 프로그래밍 스타일은 명령형 기능 및 객체지향형 기능과 자연스럽게 결합됩니다.

**Optional[^Optional] 자료형과 쿼리 Operators[^Operators]**

인벤토리의 제거 로직은 존재하지 않을 수도 있는 값을 표현하기 위해 (`?game_item`) 이라는 Optional 자료형을 사용합니다. 쿼리 operator `?` 는 options 에서 값을 추출하고, 만약 그 option 이 비어있으면 fail 됩니다. 이것은 absent values[^AbsentValues] 를 다루기 위한 편리한 구문을 제공함과 동시에 null pointer exceptions[^NullPointerExceptions] 를 제거 해주기도 합니다.

**패턴 매칭과 Control Flow**

`GetRarityMultiplier` 내부의 `case` 표현식은 패턴 매칭을 보여줍니다. switch 구문과 달리, `case` 는 값을 생성하는 표현식 입니다. 밑줄 `_` 는 catch-all pattern[^CatchAllPattern] 이 사용 됐음을 보여주지만, 이 예시에서는 결과적으로 fail 됩니다. `if` 표현식도 마찬가지로 값을 생성하며, 조건에 변수들을 binding[^Binding] 할 수 있습니다. 예시의 compound conditions[^CompoundConditions] 는 여러 작업이 어떻게 자동 failure 전파에 의해 속박되는지를 보여줍니다.

**모듈 시스템 및 접근 제어**

예시의 코드는 다른 모듈로부터 기능을 가져오는 `using` 구문으로 시작합니다. 경로 기반 모듈 시스템은 dependencies[^Dependencies]의 명확성과 영구적 접근성을 보장합니다. `<public>' 과 같은 Access specifiers[^AccessSpecifiers] 는 세부적인 수준에서 visibility[^Visibility] 를 제어합니다.

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

[^BackwardCompatibility]: 하위 호환성. 최신 버전 소프트웨어가 구버전 기능을 그대로 쓸 수 있는 성질을 말합니다.
[^ControlFlow]: 제어 흐름. 조건문, 반복문, 분기, 실패 기반 실행 등 프로그램이 어떤 경로로 실행될지를 결정하는 방법을 말합니다.
[^Specifiers]: 지정자.
[^SpeculativeExecution]: 예측 실행. 투기적 실행이라고도 합니다. 실행 결과가 채택될지 확정되지 않은 상태에서 일단 실행한 뒤, 채택이 확정되면 결과값을 반영하고, 그렇지 않으면 결과값을 폐기한 후 롤백하는 방식을 말합니다.
[^enumeration]: 열거형.
[^TypeAnnotation]: 자료형 주석.
[^Boilerplate]: 상용구 코드. 기계적으로 반복 기재해야 했던 준비 코드를 말합니다.
[^Serialize]: 직렬화. 메모리에서 계산 중인 데이터를 저장, 전송할 수 있는 데이터로 변환하는 절차를 말합니다.
[^Gracefully]: 원문에는 handle this gracefully 라고 표현되므로, 직역하면 '우아하게 처리한다' 고 번역될 수 있습니다. 하지만 프로그래밍에서 'graceful 하게 처리한다'는 것은 'failure 또는 예외 상황이 발생하더라도 프로그램이 적절하게 대응하여 정상적인 흐름을 유지할 수 있도록 처리한다' 는 의미를 갖는다고 합니다.
[^Method]: 다른 객체나 클래스에 소속된 함수를 의미합니다. '멤버 함수' 라고 부르기도 합니다. 이와 대조적으로, 독립적으로 존재하는 함수는 Function 이라고 부르며 구분합니다.
[^Assertion]: 검증 조건. 연산되는 시점에 참이어야 한다고 명시하는 조건을 말합니다. 참이면 연산을 계속 진행하고, 거짓이면 fail 됩니다. 이 예시에서는 NewWeight <= MaxWeight 부분이 이에 해당합니다. 4딸라
[^Transactional]: 여러 작업을 하나의 논리적 단위로 취급하는 성질을 가진. 아킬레우스의 발목 - 발목이 적셔지지 않았기 때문에(fail 되었으므로), 다른 모든 신체부위가 스틱스 강물에 적셔졌더라도(success) 전체 연산 결과(무적 효과 부여)가 fail(무적 효과 부여 실패) 됨.
[^FirstClassValues]: 일급 객체 값. (1) 함수의 실질적인 매개변수가 될 수 있고 (2) 함수의 반환 값이 될 수 있고 (3) 할당의 대상이 될 수 있고 (4) 비교연산을 적용할 수 있는 객체를 일급 객체라고 합니다.
[^PredicateFunction]: 프레디케이트 함수. 입력값을 받아 참(True) 또는 거짓(False)을 반환하는 함수를 말합니다.
[^NestedFunction]: 중첩 함수. 어떤 다른 함수의 내부에 선언된 함수를 말합니다.
[^Locally]: 지역적으로. 어떤 함수에 선언된 값을 그 함수 내부에서만 쓸 수 있게 한 경우, 그 값이 '지역적으로' 선언되었다고 합니다. <> Globally (전역적으로)
[^Optional]: 자료형(Types) 중 하나. '값이 있을 수도 있고 없을 수도 있는 상태'를 표현할 수 있다는 특징이 있습니다. 형용사로써 직역하면 '선택적인' 이라는 의미가 되지만, float, int 등과 같은 자료형을 나타내는 명사로 이해하는게 받아들이기 쉽습니다. 
[^Operators]: 연산자. 값과 변수를 이용한 특정 연산을 수행하도록 지시하는 기호를 말합니다. 산술, 할당, 비교, 논리, 증감, 삼항 연산자를 총칭합니다.
[^AbsentValues]: 결측값. NA(Not Available) 이나 404 not found 처럼, 시스템이 가리킨 주소를 찾아갔는데 들어있는 값이 없는 없어 '비어있다' 고 읽어야 하는 값.
[^NullPointerExceptions]: 할당된 주소가 존재하지 않기 때문에 발생하는 예외. (1) Null : '없는 번호 입니다.' 처럼, 아예 찾아갈 주소가 없는 경우를 말합니다 (2) NA : Not Available. 주소는 있지만 막상 찾아간 주소에 아무것도 없는 경우를 말합니다 (3) NAN : Not A Number. 찾아갈 주소고 있고, 찾아보니 들어있는 값도 있는데, 숫자가 아니거나 수학적으로 처리할 수 없는 경우를 말합니다.
[^CatchAllPattern]: 처리 방식을 따로 지정하지 않은 나머지 모든 값들을 일괄처리 하는 방식을 말합니다.
[^Binding]: A = x 처럼, 어떤 변수에 값을 지정하는 것을 말합니다. 예시에서 if 구문의 조건 입력 부분인 괄호를 보면, 등호를 이용해 변수에 값을 binding 해주고 있습니다.
[^CompoundConditions]: 복합 조건. 예시의 if 구문 괄호 안의 내용과 같이, 하나의 조건식에 2개 이상의 조건을 건 경우, 그 조건들을 복합 조건이라고 합니다.
[^Dependencies]: 의존성. 어떤 코드가 정상적으로 작동하기 위해 다른 코드나 모듈 등을 필요로 하는 관계를 말합니다.
[^AccessSpecifiers]: 접근 한정자. 어떤 코드 요소를 '어디에서 접근할 수 있는가'를 지정하는 역할을 합니다. Public, Private 등이 있습니다.
[^Visibility]: 가시성. 다른 코드에서 어떤 변수를 참조할 수 있는지 없는지의 여부를 말합니다.
