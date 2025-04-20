---
sidebar_position: 4
---
# 工具

工具是[物品][item]，其主要用途是破坏[方块][block]。许多MOD添加了新的工具套装（例如铜工具）或新的工具类型（例如锤子）。

## 自定义工具套装

一个工具套装通常由五种物品组成：镐、斧、铲、锄和剑。（剑在传统意义上不是工具，但为了保持一致性也包含在这里。）所有这些物品都有其对应的类：`PickaxeItem`、`AxeItem`、`ShovelItem`、`HoeItem` 和 `SwordItem`。工具的类层次结构如下：

```text
Item
- DiggerItem
    - AxeItem
    - HoeItem
    - PickaxeItem
    - ShovelItem
- SwordItem
```

工具几乎完全通过七个[数据组件][datacomponents]实现：

- `DataComponents#MAX_DAMAGE` 和 `#DAMAGE` 用于耐久度
- `#MAX_STACK_SIZE` 将堆叠大小设置为 `1`
- `#REPAIRABLE` 用于在铁砧中修复工具
- `#ENCHANTABLE` 用于最大[附魔][enchantment]值
- `#ATTRIBUTE_MODIFIERS` 用于攻击伤害和攻击速度
- `#TOOL` 用于挖掘信息

对于 `DiggerItem` 和 `SwordItem`，它们只是通过工具类记录 `ToolMaterial` 设置组件的代理。注意，其他通常被认为是工具的物品，例如剪刀，不包含在此层次结构中。相反，它们直接扩展 `Item` 并自行处理破坏逻辑。

要使用 `DiggerItem` 或 `SwordItem` 创建一套标准工具，首先需要定义一个 `ToolMaterial`。参考值可以在 `ToolMaterial` 的常量中找到。以下示例使用铜工具，你可以使用自己的材料并根据需要调整这些值。

```java
// 我们将铜的性能设置在石头和铁之间。
public static final ToolMaterial COPPER_MATERIAL = new ToolMaterial(
        // 确定此材料无法破坏的方块的标签。有关更多信息，请参见下文。
        MyBlockTags.INCORRECT_FOR_COPPER_TOOL,
        // 确定材料的耐久度。
        // 石头为 131，铁为 250。
        200,
        // 确定材料的挖掘速度。剑不使用此值。
        // 石头为 4，铁为 6。
        5f,
        // 确定攻击伤害加成。不同工具使用此值的方式不同。例如，剑的伤害为 (getAttackDamageBonus() + 4)。
        // 石头为 1，铁为 2，分别对应剑的 5 和 6 点攻击伤害；我们的剑现在造成 5.5 点伤害。
        1.5f,
        // 确定材料的附魔能力。这表示此工具的附魔效果有多好。
        // 金为 22，我们将铜设置得稍低一些。
        20,
        // 确定哪些物品可以修复此材料的标签。
        Tags.Items.INGOTS_COPPER
);
```

现在我们有了 `ToolMaterial`，可以用它来[注册][registering]工具。所有工具构造函数都有相同的四个参数。

```java
// ITEMS 是 DeferredRegister.Items 的实例
public static final DeferredItem<SwordItem> COPPER_SWORD = ITEMS.registerItem(
    "copper_sword",
    props -> new SwordItem(
        // 要使用的材料。
        COPPER_MATERIAL,
        // 类型特定的攻击伤害加成。剑为 3，铲为 1.5，镐为 1，斧和锄各不相同。
        3,
        // 类型特定的攻击速度修正。玩家的默认攻击速度为 4，因此要达到期望值 1.6f，我们使用 -2.4f。剑为 -2.4f，铲为 -3f，镐为 -2.8f，斧和锄各不相同。
        -2.4f,
        // 物品属性。
        props
    )
);

public static final DeferredItem<AxeItem> COPPER_AXE = ITEMS.registerItem("copper_axe", props -> new AxeItem(...));
public static final DeferredItem<PickaxeItem> COPPER_PICKAXE = ITEMS.registerItem("copper_pickaxe", props -> new PickaxeItem(...));
public static final DeferredItem<ShovelItem> COPPER_SHOVEL = ITEMS.registerItem("copper_shovel", props -> new ShovelItem(...));
public static final DeferredItem<HoeItem> COPPER_HOE = ITEMS.registerItem("copper_hoe", props -> new HoeItem(...));
```

### 标签

创建 `ToolMaterial` 时，会为其分配一个包含无法用此工具破坏的方块的块[标签][tags]。例如，`minecraft:incorrect_for_stone_tool` 标签包含钻石矿石等方块，而 `minecraft:incorrect_for_iron_tool` 标签包含黑曜石和远古残骸等方块。为了更容易地将方块分配到其错误的挖掘等级，还存在一个标签，用于需要此工具才能挖掘的方块。例如，`minecraft:needs_iron_tool` 标签包含钻石矿石等方块，而 `minecraft:needs_diamond_tool` 标签包含黑曜石和远古残骸等方块。

如果你对重新使用工具的错误标签没有意见，可以直接使用。例如，如果我们希望铜工具只是更耐用的石头工具，可以传入 `BlockTags#INCORRECT_FOR_STONE_TOOL`。

或者，我们可以创建自己的标签，如下所示：

```java
// 此标签将允许我们将这些方块添加到无法挖掘它们的错误标签中
public static final TagKey<Block> NEEDS_COPPER_TOOL = TagKey.create(BuiltInRegistries.BLOCK.key(), ResourceLocation.fromNamespaceAndPath(MOD_ID, "needs_copper_tool"));

// 此标签将传递到我们的材料中
public static final TagKey<Block> INCORRECT_FOR_COPPER_TOOL = TagKey.create(BuiltInRegistries.BLOCK.key(), ResourceLocation.fromNamespaceAndPath(MOD_ID, "incorrect_for_cooper_tool"));
```

然后，我们填充我们的标签。例如，让铜能够挖掘金矿石、金块和红石矿石，但不能挖掘钻石或绿宝石。（红石块已经可以被石头工具挖掘。）标签文件位于 `src/main/resources/data/mod_id/tags/block/needs_copper_tool.json`（其中 `mod_id` 是你的MOD ID）：

```json5
{
    "values": [
        "minecraft:gold_block",
        "minecraft:raw_gold_block",
        "minecraft:gold_ore",
        "minecraft:deepslate_gold_ore",
        "minecraft:redstone_ore",
        "minecraft:deepslate_redstone_ore"
    ]
}
```

然后，为了将我们的标签传递到材料中，我们可以为任何对石头工具错误但在我们的铜工具标签中的工具提供一个负约束。标签文件位于 `src/main/resources/data/mod_id/tags/block/incorrect_for_cooper_tool.json`：

```json5
{
    "values": [
        "#minecraft:incorrect_for_stone_tool"
    ],
    "remove": [
        "#mod_id:needs_copper_tool"
    ]
}
```

最后，我们可以将标签传递到我们的材料实例中，如上所示。

如果你想检查工具是否可以使方块状态掉落其方块，可以调用 `Tool#isCorrectForDrops`。可以通过调用 `ItemStack#get` 和 `DataComponents#TOOL` 获取 `Tool`。

## 自定义工具

自定义工具可以通过将 `Tool` [数据组件][datacomponents]（通过 `DataComponents#TOOL`）添加到物品的默认组件列表中来创建，方法是通过 `Item.Properties#component`。`DiggerItem` 是一个实现，它接收一个 `ToolMaterial`，如上所述，用于构造 `Tool`，以及一些其他基本组件，如属性和耐久度。

`Tool` 包含一个 `Tool.Rule` 列表，持有工具时的默认挖掘速度（默认为 `1`），以及挖掘方块时工具应承受的伤害量（默认为 `1`）。`Tool.Rule` 包含三部分信息：一个应用规则的方块的 `HolderSet`，一个可选的挖掘这些方块的速度，以及一个可选的布尔值，用于确定这些方块是否可以从此工具中掉落。如果未设置可选项，则将检查其他规则。如果所有规则都失败，则默认行为是默认挖掘速度，并且方块无法掉落。

:::note
可以通过 `Registry#getOrThrow` 从 `TagKey` 创建 `HolderSet`。
:::

创建任何工具或多功能工具类物品（即将两种或多种工具组合为一的物品，例如将斧头和镐头组合为一个物品）不需要扩展任何现有的 `DiggerItem` 或 `SwordItem`。它可以简单地通过以下部分的组合实现：

- 通过设置 `DataComponents#TOOL` 使用你自己的规则添加 `Tool`，方法是通过 `Item.Properties#component`。
- 通过 `Item.Properties#attributes` 向物品添加[属性修饰符][attributemodifier]（例如攻击伤害、攻击速度）。
- 通过 `Item.Properties#durability` 添加物品耐久度。
- 通过 `Item.Properties#repariable` 允许修复物品。
- 通过 `Item.Properties#enchantable` 允许物品附魔。
- 重写 `IItemExtension#canPerformAction` 以确定物品可以执行哪些[`ItemAbility`][itemability]。
- 如果你希望物品根据 `ItemAbility` 修改右键单击时的方块状态，可以调用 `IBlockExtension#getToolModifiedState`。
- 将你的工具添加到一些 `minecraft:enchantable/*` `ItemTags` 中，以便可以对你的物品应用某些附魔。
- 将你的工具添加到一些 `minecraft:*_preferred_weapons` 标签中，以允许生物更倾向于拾取和使用你的武器。

## `ItemAbility`

`ItemAbility` 是对物品可以执行和不能执行的操作的抽象。这包括左键单击和右键单击行为。NeoForge 在 `ItemAbilities` 类中提供了默认的 `ItemAbility`：

- 挖掘能力。这些能力适用于上述所有四种 `DiggerItem` 类型，以及剑和剪刀的挖掘。
- 斧头右键单击能力，用于剥皮（原木）、刮擦（氧化铜）和去蜡（打蜡铜）。
- 铲子右键单击能力，用于平整（泥土路径）和熄灭（营火）。
- 剪刀能力，用于收获（蜂巢）、移除盔甲（装甲狼）、雕刻（南瓜）、解除武装（绊线）和修剪（停止植物生长）。
- 剑的横扫能力、锄头的耕地能力、盾牌的格挡能力、钓鱼竿的投掷能力、三叉戟的投掷能力、刷子的刷洗能力和火种的点燃能力。

要创建你自己的 `ItemAbility`，请使用 `ItemAbility#get` - 如果需要，它将创建一个新的 `ItemAbility`。然后，在自定义工具类型中，根据需要重写 `IItemExtension#canPerformAction`。

要查询 `ItemStack` 是否可以执行某个 `ItemAbility`，请调用 `IItemStackExtension#canPerformAction`。请注意，这适用于任何 `Item`，而不仅仅是工具。

[block]: ../blocks/index.md
[datacomponents]: datacomponents.md
[enchantment]: ../resources/server/enchantments/index.md#enchantment-costs-and-levels
[item]: index.md
[itemability]: #itemability
[registering]: ../concepts/registries.md#methods-for-registering
[tags]: ../resources/server/tags.md
