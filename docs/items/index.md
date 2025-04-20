# 物品(item)

与方块一起，物品是Minecraft的关键组成部分。方块构成了你周围的世界，而物品存在于物品栏中。

## 什么是物品？

在深入创建物品之前，理解物品的本质以及它与[方块][block]的区别是很重要的。让我们通过一个例子来说明：

- 在世界中，你遇到一个泥土方块并想要挖掘它。这是一个**方块**，因为它被放置在世界中。（实际上，它不是一个方块，而是一个方块状态（blockstate）。有关更详细的信息，请参阅[方块状态文章][blockstates]。）
    - 并非所有方块在破坏时都会掉落自身（例如树叶），有关更多信息，请参阅[战利品表][loottables]文章。
- 一旦你[挖掘了方块][breaking]，它会被移除（=替换为空气方块），并掉落泥土。掉落的泥土是一个物品**[实体][entity]**。这意味着像其他实体（猪、僵尸、箭等）一样，它可以被水推动或被火和岩浆烧毁。
- 当你拾取泥土物品实体时，它会成为物品栏中的一个**物品堆(itemstack)**。物品堆简单来说就是一个物品实例，带有一些额外的信息，例如堆叠大小。
- 物品堆由其对应的**物品**支持（这就是我们要创建的内容）。物品持有[数据组件][datacomponents]，其中包含所有物品堆初始化时的默认信息（例如，每把铁剑的最大耐久度为250），而物品堆可以修改这些数据组件，从而允许同一物品的两个不同堆具有不同的信息（例如，一把铁剑剩余100次耐久，而另一把铁剑剩余200次耐久）。有关通过物品完成的操作以及通过物品堆完成的操作的更多信息，请继续阅读。
    - 物品与物品堆之间的关系大致类似于[方块][block]与[方块状态][blockstates]之间的关系，即方块状态始终由方块支持。这不是一个非常准确的比较（例如，物品堆不是单例），但它提供了一个关于这一概念的基本理解。

## 创建一个物品

现在我们理解了物品是什么，让我们创建一个！

与基本方块类似，对于不需要特殊功能的基本物品（例如木棍、糖等），可以直接使用`Item`类。为此，在注册期间，使用`Item.Properties`参数实例化`Item`。可以通过`Item.Properties#of`创建此`Item.Properties`参数，并通过调用其方法进行自定义：

- `setId` - 设置物品的资源键（resource key）。
    - 每个物品**必须**设置此项；否则会抛出异常。
- `overrideDescription` - 设置物品的翻译键。创建的`Component`存储在`DataComponents#ITEM_NAME`中。
- `useBlockDescriptionPrefix` - 一个便捷的助手，使用翻译键`block.<modid>.<registry_name>`调用`overrideDescription`。这应该在任何`BlockItem`上调用。
- `requiredFeatures` - 设置此物品所需的功能标志。这主要用于原版在小版本中的功能锁定系统。不建议使用此功能，除非你正在集成一个由原版功能标志锁定的系统。
- `stacksTo` - 设置此物品的最大堆叠大小（通过`DataComponents#MAX_STACK_SIZE`）。默认为64。例如末影珍珠或其他只能堆叠到16的物品使用此项。
- `durability` - 设置此物品的耐久度（通过`DataComponents#MAX_DAMAGE`）以及初始损坏值为0（通过`DataComponents#DAMAGE`）。默认为0，表示“无耐久度”。例如，铁工具在此处使用250。注意，设置耐久度会自动将最大堆叠大小锁定为1。
- `fireResistant` - 使使用此物品的物品实体免受火和岩浆的伤害（通过`DataComponents#FIRE_RESISTANT`）。用于各种下界合金物品。
- `rarity` - 设置此物品的稀有度（通过`DataComponents#RARITY`）。目前，这仅更改物品的颜色。`Rarity`是一个枚举，包含四个值：`COMMON`（白色，默认）、`UNCOMMON`（黄色）、`RARE`（青色）和`EPIC`（浅紫色）。请注意，MOD可能会添加更多稀有度类型。
- `setNoRepair` - 禁用此物品的铁砧和工作台修复功能。原版未使用。
- `jukeboxPlayable` - 设置数据包`JukeboxSong`的资源键，当插入唱片机时播放。
- `food` - 设置此物品的[`FoodProperties`][food]（通过`DataComponents#FOOD`）。

有关示例或Minecraft使用的各种值，请查看`Items`类。

### 剩余物品和冷却时间

物品可能具有在使用时应用的附加属性，或在设定时间内防止物品被使用：

- `craftRemainder` - 设置此物品的制作剩余物品。原版在制作后留下空桶的填充桶使用此项。
- `usingConvertsTo` - 设置通过`Item#use`、`IItemExtension#finishUsingItem`或`Item#releaseUsing`完成使用后返回的物品。`ItemStack`存储在`DataComponents#USE_REMAINDER`中。
- `useCooldown` - 设置物品再次使用前的秒数（通过`DataComponents#USE_COOLDOWN`）。

### 工具和盔甲

某些物品的行为类似于[工具][tools]和[盔甲][armor]。这些通过一系列物品属性构造，只有部分用法委托给其关联的类：

- `enchantable` - 设置堆栈的最大[附魔][enchantment]值，允许物品被附魔（通过`DataComponents#ENCHANTABLE`）。
- `repairable` - 设置可用于修复此物品耐久度的物品或标签（通过`DataComponents#REPAIRABLE`）。必须具有耐久度组件且不为`DataComponents#UNBREAKABLE`。
- `equippable` - 设置物品可装备到的槽位（通过`DataComponents#EQUIPPABLE`）。
- `equippableUnswappable` - 与`equippable`相同，但禁用通过使用物品按钮（默认右键）快速交换。

更多信息可以在相关页面找到。

### 更多功能

直接使用`Item`仅允许非常基本的物品。如果你想添加功能，例如右键交互，则需要一个继承`Item`的自定义类。`Item`类有许多可以重写的方法以执行不同的操作；有关更多信息，请参阅`Item`和`IItemExtension`类。

物品的两个最常见用例是左键点击和右键点击。由于其复杂性及其涉及其他系统，它们在单独的[交互文章][interactions]中进行了说明。

### `DeferredRegister.Items`

所有注册表都使用`DeferredRegister`注册其内容，物品也不例外。然而，由于添加新物品是绝大多数MOD的一个重要功能，NeoForge提供了继承`DeferredRegister<Item>`的`DeferredRegister.Items`辅助类，并提供了一些特定于物品的助手：

```java
public static final DeferredRegister.Items ITEMS = DeferredRegister.createItems(ExampleMod.MOD_ID);

public static final DeferredItem<Item> EXAMPLE_ITEM = ITEMS.registerItem(
    "example_item",
    Item::new, // 将传递属性的工厂。
    new Item.Properties() // 要使用的属性。
);
```

内部，这将通过将属性参数应用于提供的物品工厂（通常是构造函数），简单地调用`ITEMS.register("example_item", registryName -> new Item(new Item.Properties().setId(ResourceKey.create(Registries.ITEM, registryName))))`。ID设置在属性上。

如果你想使用`Item::new`，可以完全省略工厂并使用`simple`方法变体：

```java
public static final DeferredItem<Item> EXAMPLE_ITEM = ITEMS.registerSimpleItem(
    "example_item",
    new Item.Properties() // 要使用的属性。
);
```

这与前面的示例完全相同，但稍微简短一些。当然，如果你想使用`Item`的子类而不是`Item`本身，则必须使用前一种方法。

这两种方法还有省略`new Item.Properties()`参数的重载：

```java
public static final DeferredItem<Item> EXAMPLE_ITEM = ITEMS.registerItem("example_item", Item::new);

// 省略Item::new参数的变体
public static final DeferredItem<Item> EXAMPLE_ITEM = ITEMS.registerSimpleItem("example_item");
```

最后，还有方块物品(blockitem)的快捷创建方式。除了`setId`，它们还调用`useBlockDescriptionPrefix`将翻译键设置为方块使用的键：

```java
public static final DeferredItem<BlockItem> EXAMPLE_BLOCK_ITEM = ITEMS.registerSimpleBlockItem(
    "example_block",
    ExampleBlocksClass.EXAMPLE_BLOCK,
    new Item.Properties()
);

// 省略属性参数的变体：
public static final DeferredItem<BlockItem> EXAMPLE_BLOCK_ITEM = ITEMS.registerSimpleBlockItem(
    "example_block",
    ExampleBlocksClass.EXAMPLE_BLOCK
);

// 省略名称参数的变体，改为使用方块的注册名称：
public static final DeferredItem<BlockItem> EXAMPLE_BLOCK_ITEM = ITEMS.registerSimpleBlockItem(
    // 必须是`Holder<Block>`的实例
    // DeferredBlock<T>也适用
    ExampleBlocksClass.EXAMPLE_BLOCK,
    new Item.Properties()
);

// 省略名称和属性的变体：
public static final DeferredItem<BlockItem> EXAMPLE_BLOCK_ITEM = ITEMS.registerSimpleBlockItem(
    // 必须是`Holder<Block>`的实例
    // DeferredBlock<T>也适用
    ExampleBlocksClass.EXAMPLE_BLOCK
);
```

:::note
如果你将注册的方块保存在单独的类中，你应该在物品类之前加载方块类。
:::

### 资源

如果你注册了物品并通过`/give`或[创造模式标签][creativetabs]获取物品，你会发现它缺少适当的模型和纹理。这是因为纹理和模型由Minecraft的资源系统处理。

要为物品应用简单的纹理，你必须创建一个客户端物品模型JSON和一个纹理PNG。有关更多信息，请参阅[客户端物品][citems]部分。

## `ItemStack`s

与方块和方块状态类似，大多数你期望使用`Item`的地方实际上使用的是`ItemStack`。`ItemStack`表示容器（例如物品栏）中的一个或多个物品堆。再次类似于方块和方块状态，方法应该由`Item`重写并在`ItemStack`上调用，`Item`中的许多方法会传递一个`ItemStack`实例。

一个`ItemStack`由三个主要部分组成：

- 它表示的`Item`，可通过`ItemStack#getItem`或`getItemHolder`获取`Holder<Item>`。
- 堆叠大小，通常在1到64之间，可通过`getCount`获取并通过`setCount`或`shrink`更改。
- [数据组件][datacomponents]映射，其中存储了特定于堆栈的数据。可通过`getComponents`获取。组件值通常通过`has`、`get`、`set`、`update`和`remove`访问和修改。

要创建一个新的`ItemStack`，调用`new ItemStack(Item)`，传入支持的物品。默认情况下，这使用1的计数和没有NBT数据；如果需要，还有接受计数和NBT数据的构造函数重载。

`ItemStack`是可变对象（见下文），但有时需要将它们视为不可变对象。如果你需要修改一个被视为不可变的`ItemStack`，可以使用`#copy`或`#copyWithCount`克隆堆栈，如果需要特定的堆叠大小。

如果你想表示堆栈没有物品，请使用`ItemStack.EMPTY`。如果你想检查`ItemStack`是否为空，请调用`#isEmpty`。

### `ItemStack`的可变性

`ItemStack`是可变对象。这意味着如果你调用例如`#setCount`或任何数据组件映射方法，`ItemStack`本身将被修改。原版广泛使用`ItemStack`的可变性，许多方法依赖于它。例如，`#split`从调用的堆栈中分离给定数量，同时修改调用者并返回一个新的`ItemStack`。

然而，在同时处理多个`ItemStack`时，这有时会导致问题。最常见的情况是处理物品栏槽位时，因为你必须同时考虑光标当前选择的`ItemStack`以及你试图插入/提取的`ItemStack`。

:::tip
如果不确定，最好安全起见并复制堆栈。
:::

### JSON表示

在许多情况下，例如[配方][recipes]，物品堆需要表示为JSON对象。物品堆的JSON表示如下所示：

```json5
{
    // 物品ID。必需。
    "id": "minecraft:dirt",
    // 物品堆计数[1, 99]。可选，默认为1。
    "count": 4,
    // 数据组件的映射。可选，默认为空映射。
    "components": {
        "minecraft:enchantment_glint_override": true
    }
}
```

## 创造模式标签

默认情况下，你的物品只能通过`/give`获得，而不会出现在创造模式物品栏中。让我们改变这一点！

将物品添加到创造模式菜单的方式取决于你想要将其添加到的标签。

### 现有的创造模式标签

:::note
此方法用于将物品添加到Minecraft的标签或其他MOD的标签中。要将物品添加到你自己的标签中，请参阅下文。
:::

可以通过`BuildCreativeModeTabContentsEvent`将物品添加到现有的`CreativeModeTab`，该事件在[MOD事件总线][modbus]上触发，仅在[逻辑客户端][sides]上。通过调用`event#accept`添加物品。

```java
//MyItemsClass.MY_ITEM 是 Supplier<? extends Item> 的实例，MyBlocksClass.MY_BLOCK 是 Supplier<? extends Block> 的实例
@SubscribeEvent
public static void buildContents(BuildCreativeModeTabContentsEvent event) {
    // 这是我们想要添加的标签吗？
    if (event.getTabKey() == CreativeModeTabs.INGREDIENTS) {
        event.accept(MyItemsClass.MY_ITEM.get());
        // 接受一个ItemLike。这假设MY_BLOCK有一个对应的物品。
        event.accept(MyBlocksClass.MY_BLOCK.get());
    }
}
```

该事件还提供了一些额外信息，例如`getFlags`以获取启用的功能标志列表，或`hasPermissions`以检查玩家是否有权限查看操作员物品标签。

### 自定义创造模式标签

`CreativeModeTab`是一个注册表，这意味着自定义`CreativeModeTab`必须[注册][registering]。创建一个创造模式标签使用一个构建器系统，构建器可通过`CreativeModeTab#builder`获取。构建器提供了设置标题、图标、默认物品和许多其他属性的选项。此外，NeoForge提供了额外的方法来自定义标签的图像、标签和槽位颜色，标签的排序位置等。

```java
//CREATIVE_MODE_TABS 是 DeferredRegister<CreativeModeTab> 的实例
public static final Supplier<CreativeModeTab> EXAMPLE_TAB = CREATIVE_MODE_TABS.register("example", () -> CreativeModeTab.builder()
    //设置标签的标题。不要忘记添加翻译！
    .title(Component.translatable("itemGroup." + MOD_ID + ".example"))
    //设置标签的图标。
    .icon(() -> new ItemStack(MyItemsClass.EXAMPLE_ITEM.get()))
    //将你的物品添加到标签中。
    .displayItems((params, output) -> {
        output.accept(MyItemsClass.MY_ITEM.get());
        // 接受一个ItemLike。这假设MY_BLOCK有一个对应的物品。
        output.accept(MyBlocksClass.MY_BLOCK.get());
    })
    .build()
);
```

## `ItemLike`

`ItemLike`是一个接口，由原版中的`Item`和[`Block`][block]实现。它定义了方法`#asItem`，返回对象实际的物品表示：`Item`只返回自身，而`Block`返回其关联的`BlockItem`（如果可用），否则返回`Blocks.AIR`。`ItemLike`在许多上下文中使用，其中物品的“来源”并不重要，例如在许多[数据生成器][datagen]中。

也可以在自定义对象上实现`ItemLike`。只需重写`#asItem`即可。

[armor]: armor.md
[block]: ../blocks/index.md
[blockstates]: ../blocks/states.md
[breaking]: ../blocks/index.md#breaking-a-block
[citems]: ../resources/client/models/items.md
[creativetabs]: #创造模式标签
[datacomponents]: datacomponents.md
[datagen]: ../resources/index.md#data-generation
[enchantment]: ../resources/server/enchantments/index.md#enchantment-costs-and-levels
[entity]: ../entities/index.md
[food]: consumables.md#food
[hunger]: https://minecraft.wiki/w/Hunger#Mechanics
[interactions]: interactions.md
[loottables]: ../resources/server/loottables/index.md
[modbus]: ../concepts/events.md#event-buses
[recipes]: ../resources/server/recipes/index.md
[registering]: ../concepts/registries.md#methods-for-registering
[sides]: ../concepts/sides.md
[tools]: tools.md
