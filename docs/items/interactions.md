---
sidebar_position: 1
---
# 交互

本页面旨在使玩家左键、右键或中键点击物品的相对复杂和令人困惑的过程更易于理解，并澄清在何处以及为何使用某些结果。

## `HitResult`

为了确定玩家当前正在看什么，Minecraft 使用了一个 `HitResult`。`HitResult` 在某种程度上等同于其他游戏引擎中的射线投射结果，最显著的是它包含一个方法 `#getLocation`。

一个命中结果可以是三种类型之一，通过枚举 `HitResult.Type` 表示：`BLOCK`（方块）、`ENTITY`（实体）或 `MISS`（未命中）。类型为 `BLOCK` 的 `HitResult` 可以转换为 `BlockHitResult`，而类型为 `ENTITY` 的 `HitResult` 可以转换为 `EntityHitResult`；这两种类型提供了有关命中[方块][block]或[实体][entity]的额外上下文。如果类型是 `MISS`，这表明既没有命中方块也没有命中实体，不应转换为任何子类。

在[物理客户端][physicalside]的每一帧中，`Minecraft` 类会更新并存储当前所看的 `HitResult` 到 `hitResult` 字段。然后可以通过 `Minecraft.getInstance().hitResult` 访问该字段。

## 左键点击物品

- 检查主手中的[`物品堆`][itemstack]是否启用了所有必需的[功能标志][featureflag]。如果此检查失败，流程结束。
- 触发 `InputEvent.InteractionKeyMappingTriggered`，使用左键和主手。如果[事件][event]被[取消][cancel]，流程结束。
- 根据你正在看的内容（使用 `Minecraft` 中的[`HitResult`][hitresult]），会发生不同的事情：
  - 如果你正在看一个在你范围内的[实体][entity]：
    - 触发 `AttackEntityEvent`。如果事件被取消，流程结束。
    - 调用 `IItemExtension#onLeftClickEntity`。如果返回 true，流程结束。
    - 在目标上调用 `Entity#isAttackable`。如果返回 false，流程结束。
    - 在目标上调用 `Entity#skipAttackInteraction`。如果返回 true，流程结束。
    - 如果目标在 `minecraft:redirectable_projectile` 标签中（默认是火球和风暴冲击）并且是 `Projectile` 的实例，则目标被偏转，流程结束。
    - 计算实体基础伤害（`minecraft:generic.attack_damage`[属性][attribute]的值）和附魔额外伤害为两个单独的浮点数。如果两者都为 0，流程结束。
      - 注意，这不包括主手物品的[属性修饰符][attributemodifier]，这些修饰符会在检查后添加。
    - 将主手物品的 `minecraft:generic.attack_damage` 属性修饰符添加到基础伤害中。
    - 触发 `CriticalHitEvent`。如果事件的 `#isCriticalHit` 方法返回 true，则基础伤害乘以事件的 `#getDamageMultiplier` 方法返回的值（如果[若干条件][critical]通过，默认为 1.5，否则为 1.0，但可能会被事件修改）。
    - 将附魔额外伤害添加到基础伤害中，得到最终伤害值。
    - 调用[`Entity#hurt`][hurt]。如果返回 false，流程结束。
    - 如果目标是 `LivingEntity` 的实例，调用 `LivingEntity#knockback`。
      - 在该方法中，触发 `LivingKnockBackEvent`。
    - 如果攻击冷却时间 > 90%，攻击不是暴击，玩家在地面上且移动速度不超过其 `minecraft:generic.movement_speed` 属性值，则对附近的 `LivingEntity` 执行横扫攻击。
      - 在该方法中，再次调用 `LivingEntity#knockback`，这又会触发第二次 `LivingKnockBackEvent`。
    - 调用 `Item#hurtEnemy`。这可以用于攻击后的效果。例如，锤子在此处将玩家向空中发射（如果适用）。
    - 调用 `Item#postHurtEnemy`。耐久度损耗在此处应用。
  - 如果你正在看一个在你范围内的[方块][block]：
    - 启动[方块破坏子流程][blockbreak]。
  - 否则：
    - 触发 `PlayerInteractEvent.LeftClickEmpty`。

## 右键点击物品

在右键点击流程中，会调用许多返回两种结果类型之一的方法（见下文）。这些方法中的大多数如果返回明确的成功或失败结果，则会取消流程。为了便于阅读，这种“明确的成功或失败”将从现在起称为“确定性结果”。

- 触发 `InputEvent.InteractionKeyMappingTriggered`，使用右键和主手。如果[事件][event]被[取消][cancel]，流程结束。
- 检查若干情况，例如你是否处于旁观者模式，或者主手中的[`物品堆`][itemstack]是否启用了所有必需的[功能标志][featureflag]。如果至少有一个检查失败，流程结束。
- 根据你正在看的内容（使用 `Minecraft` 中的[`HitResult`][hitresult]），会发生不同的事情：
  - 如果你正在看一个在你范围内且不在世界边界外的[实体][entity]：
    - 触发 `PlayerInteractEvent.EntityInteractSpecific`。如果事件被取消，流程结束。
    - 在**你正在看的实体**上调用 `Entity#interactAt`。如果返回一个确定性结果，流程结束。
      - 如果你想为自己的实体添加行为，请覆盖此方法。如果你想为原版实体添加行为，请使用事件。
    - 如果实体打开了一个界面（例如村民交易 GUI 或矿车箱 GUI），流程结束。
    - 触发 `PlayerInteractEvent.EntityInteract`。如果事件被取消，流程结束。
    - 在**你正在看的实体**上调用 `Entity#interact`。如果返回一个确定性结果，流程结束。
      - 如果你想为自己的实体添加行为，请覆盖此方法。如果你想为原版实体添加行为，请使用事件。
      - 对于[`Mob`][livingentity]，`Entity#interact` 的覆盖处理了诸如拴住和生成宝宝（当主手中的 `ItemStack` 是生成蛋时）的事情，然后将特定于生物的处理推迟到 `Mob#mobInteract`。`Entity#interact` 的结果规则也适用于此。
    - 如果你正在看的实体是 `LivingEntity`，则在主手中的 `ItemStack` 上调用 `Item#interactLivingEntity`。如果返回一个确定性结果，流程结束。
  - 如果你正在看一个在你范围内且不在世界边界外的[方块][block]：
    - 触发 `PlayerInteractEvent.RightClickBlock`。如果事件被取消，流程结束。你也可以在此事件中专门拒绝方块或物品的使用。
    - 调用 `IItemExtension#onItemUseFirst`。如果返回一个确定性结果，流程结束。
    - 如果玩家未蹲伏且事件未拒绝方块使用，则触发 `UseItemOnBlockEvent`。如果事件被取消，则使用取消结果。否则，调用 `Block#useItemOn`。如果返回一个确定性结果，流程结束。
    - 如果 `InteractionResult` 是 `TRY_WITH_EMPTY_HAND` 且执行的手是主手，则调用 `Block#useWithoutItem`。如果返回一个确定性结果，流程结束。
    - 如果事件未拒绝物品使用，则调用 `Item#useOn`。如果返回一个确定性结果，流程结束。
- 调用 `Item#use`。如果返回一个确定性结果，流程结束。
- 上述过程运行第二次，这次使用副手而不是主手。

### `InteractionResult`

`InteractionResult` 是一个密封接口，表示物品或空手与某些对象（例如实体、方块等）之间某些交互的结果。该接口分为四个记录，其中有六种潜在的默认状态。

首先是 `InteractionResult.Success`，表示操作应被视为成功，结束流程。成功状态有两个参数：`SwingSource`，表示实体是否应在相应的[逻辑端][side]挥动；以及 `InteractionResult.ItemContext`，它包含交互是否由持有物品引起，以及使用后持有物品变成了什么。挥动源由以下默认状态之一确定：`InteractionResult#SUCCESS` 表示客户端挥动，`InteractionResult#SUCCESS_SERVER` 表示服务器挥动，`InteractionResult#CONSUME` 表示不挥动。物品上下文通过 `Success#heldItemTransformedTo` 设置，如果 `ItemStack` 发生了变化，或者通过 `withoutItem` 设置，如果持有物品与对象之间没有交互。默认设置为存在物品交互但没有转换。

```java
// 在某些返回交互结果的方法中

// 手中的物品将变成一个苹果
return InteractionResult.SUCCESS.heldItemTransformedTo(new ItemStack(Items.APPLE));
```

:::note
`SUCCESS` 和 `SUCCESS_SERVER` 通常不应在同一方法中使用。如果客户端有足够的信息来确定何时挥动，则应始终使用 `SUCCESS`。否则，如果它依赖于客户端上不存在的服务器信息，则应使用 `SUCCESS_SERVER`。
:::

然后是 `InteractionResult.Fail`，由 `InteractionResult#FAIL` 实现，表示操作应被视为失败，不允许进一步交互。流程将结束。这可以在任何地方使用，但在 `Item#useOn` 和 `Item#use` 之外使用时应谨慎。在许多情况下，使用 `InteractionResult#PASS` 更有意义。

最后是 `InteractionResult.Pass` 和 `InteractionResult.TryWithEmptyHandInteraction`，分别由 `InteractionResult#PASS` 和 `InteractionResult#TRY_WITH_EMPTY_HAND` 实现。这些记录表示操作应被视为既不成功也不失败，流程应继续。`PASS` 是所有 `InteractionResult` 方法的默认行为，除了 `BlockBehaviour#useItemOn`，它返回 `TRY_WITH_EMPTY_HAND`。更具体地说，如果 `BlockBehaviour#useItemOn` 返回的不是 `TRY_WITH_EMPTY_HAND`，则无论物品是否在主手中，都不会调用 `BlockBehaviour#useWithoutItem`。

某些方法具有特殊行为或要求，这些将在以下章节中解释。

#### `Item#useOn`

如果你希望操作被视为成功，但不希望手臂挥动或授予 `ITEM_USED` 统计点，请使用 `InteractionResult#CONSUME` 并调用 `#withoutItem`。

```java
// 在 Item#useOn 中
return InteractionResult.CONSUME.withoutItem();
```

#### `Item#use`

这是唯一一个从 `Success` 变体（`SUCCESS`、`SUCCESS_SERVER`、`CONSUME`）中使用转换后的 `ItemStack` 的实例。通过 `Success#heldItemTransformedTo` 设置的结果 `ItemStack` 替换了启动使用的 `ItemStack`，如果它已更改。

`Item#use` 的默认实现在物品可食用（具有 `DataComponents#CONSUMABLE`）且玩家可以吃该物品（因为他们饿了，或者因为物品始终可食用）时返回 `InteractionResult#CONSUME`，在物品可食用（具有 `DataComponents#CONSUMABLE`）但玩家不能吃该物品时返回 `InteractionResult#FAIL`。如果物品可装备（具有 `DataComponents#EQUIPPABLE`），则在与持有物品交换时返回 `InteractionResult#SUCCESS`，持有物品被交换的物品替换（通过 `heldItemTransformedTo`），或者如果盔甲上的附魔具有 `EnchantmentEffectComponents#PREVENT_ARMOR_CHANGE` 组件，则返回 `InteractionResult#FAIL`。否则返回 `InteractionResult#PASS`。

在考虑主手时在此处返回 `InteractionResult#FAIL` 将阻止副手行为运行。如果你希望副手行为运行（通常你会希望这样），请改为返回 `InteractionResult#PASS`。

## 中键点击

- 如果 `Minecraft.getInstance().hitResult` 中的[`HitResult`][hitresult]为 null 或类型为 `MISS`，流程结束。
- 触发 `InputEvent.InteractionKeyMappingTriggered`，使用左键和主手。如果[事件][event]被[取消][cancel]，流程结束。
- 根据你正在看的内容（使用 `Minecraft.getInstance().hitResult` 中的 `HitResult`），会发生不同的事情：
  - 如果你正在看一个在你范围内的[实体][entity]：
    - 如果 `Entity#isPickable` 返回 false，流程结束。
    - 如果你不在创造模式，流程结束。
    - 调用 `IEntityExtension#getPickedResult`。生成的 `ItemStack` 被添加到玩家的库存中。
      - 默认情况下，此方法转发到 `Entity#getPickResult`，可以被模组开发者覆盖。
  - 如果你正在看一个在你范围内的[方块][block]：
    - 调用 `Block#getCloneItemStack` 并成为“选定的”`ItemStack`。
      - 默认情况下，这返回 `Block` 的 `Item` 表示。
    - 如果按下了 Control 键，玩家处于创造模式且目标方块具有[`方块实体`][blockentity]，则将方块实体的数据添加到“选定的”`ItemStack`。
    - 如果玩家处于创造模式，“选定的”`ItemStack` 被添加到玩家的库存中。否则，如果存在与“选定的”物品匹配的快捷栏槽，则将该快捷栏槽设置为活动状态。

[attribute]: ../entities/attributes.md
[attributemodifier]: ../entities/attributes.md#attribute-modifiers
[block]: ../blocks/index.md
[blockbreak]: ../blocks/index.md#breaking-a-block
[blockentity]: ../blockentities/index.md
[cancel]: ../concepts/events.md#cancellable-events
[critical]: https://minecraft.wiki/w/Damage#Critical_hit
[effect]: mobeffects.md
[entity]: ../entities/index.md
[event]: ../concepts/events.md
[featureflag]: ../advanced/featureflags.md
[hitresult]: #hitresult
[hurt]: ../entities/index.md#damaging-entities
[itemstack]: index.md#itemstacks
[itemuseon]: #itemuseon
[livingentity]: ../entities/livingentity.md
[physicalside]: ../concepts/sides.md#the-physical-side
[side]: ../concepts/sides.md#the-logical-side
