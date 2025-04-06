# 方块

方块是 Minecraft 世界的基础。它们构成了所有的地形、结构和机器。如果你对制作一个 MOD 感兴趣，那么你可能会想要添加一些方块。本页面将指导你创建方块，以及可以用它们做的一些事情。

## 唯一的方块

在开始之前，重要的是要理解游戏中每种方块只有一个实例。一个世界由成千上万个对同一个方块的引用组成，这些引用位于不同的位置。换句话说，同一个方块会被多次显示。

因此，一个方块只能被实例化一次，并且只能在[注册][registration]期间实例化。一旦方块被注册，你就可以根据需要使用已注册的引用。

与大多数其他注册表不同，方块可以使用一个名为 `DeferredRegister.Blocks` 的特殊版本的 `DeferredRegister`。`DeferredRegister.Blocks` 的功能基本上与 `DeferredRegister<Block>` 类似，但有一些细微的差别：

- 它们通过 `DeferredRegister.createBlocks("yourmodid")` 创建，而不是常规的 `DeferredRegister.create(...)` 方法。
- `#register` 返回一个 `DeferredBlock<T extends Block>`，它继承了 `DeferredHolder<Block, T>`。`T` 是我们注册的方块类的类型。
- 有一些用于注册方块的辅助方法。详情见[下文][below]。

现在，注册我们的方块：

```java
//BLOCKS 是一个 DeferredRegister.Blocks 实例
public static final DeferredBlock<Block> MY_BLOCK = BLOCKS.register("my_block", registryName -> new Block(...));
```

在注册方块后，所有对新 `my_block` 的引用都应该使用这个常量。例如，如果你想检查给定位置的方块是否是 `my_block`，代码可能如下所示：

```java
level.getBlockState(position) // 返回放置在给定世界 (level) 中给定位置的方块状态(BlockState)
    //highlight-next-line
    .is(MyBlockRegistrationClass.MY_BLOCK);
```

这种方法还有一个方便的效果，即 `block1 == block2` 可以使用，并且可以代替 Java 的 `equals` 方法（当然，使用 `equals` 仍然有效，但没有意义，因为它本质上是按引用比较的）。

:::danger
不要在注册之外调用 `new Block()`！一旦你这样做，事情可能会出错：

- 方块必须在注册表未冻结时创建。NeoForge 会为你解冻注册表并稍后冻结它们，因此注册是你创建方块的时间窗口。
- 如果你尝试在注册表再次冻结时创建和/或注册一个方块，游戏将崩溃并报告一个 `null` 方块，这可能会非常令人困惑。
- 如果你仍然设法拥有一个未注册的方块实例，游戏在同步和保存时将无法识别它，并将其替换为空气。

:::

## 创建方块

如前所述，我们从创建的 `DeferredRegister.Blocks` 开始：

```java
public static final DeferredRegister.Blocks BLOCKS = DeferredRegister.createBlocks("yourmodid");
```

### 基本方块

对于不需要特殊功能的简单方块（例如圆石、木板等），可以直接使用 `Block` 类。为此，在注册期间，使用 `BlockBehaviour.Properties` 参数实例化 `Block`。可以使用 `BlockBehaviour.Properties#of` 创建此 `BlockBehaviour.Properties` 参数，并可以通过调用其方法进行自定义。以下是最重要的方法：

- `setId` - 设置方块的资源键(resource key)。
  - 每个方块**必须**设置此项，否则会抛出异常。
- `destroyTime` - 确定破坏方块所需的时间。
  - 石头的破坏时间为 1.5，泥土为 0.5，黑曜石为 50，基岩为 -1（不可破坏）。
- `explosionResistance` - 确定方块的爆炸抗性。
  - 石头的爆炸抗性为 6.0，泥土为 0.5，黑曜石为 1,200，基岩为 3,600,000。
- `sound` - 设置方块在被击打、破坏或放置时发出的声音。
  - 默认值为 `SoundType.STONE`。有关详细信息，请参阅[声音页面][sounds]。
- `lightLevel` - 设置方块的光照发射强度。接受一个带有 `BlockState` 参数的函数，该函数返回一个介于 0 和 15 之间的值。
  - 例如，荧石使用 `state -> 15`，火把使用 `state -> 14`。
- `friction` - 设置方块的摩擦（滑溜性）。
  - 默认值为 0.6。冰使用 0.98。

例如，一个简单的实现可能如下所示：

```java
//BLOCKS 是一个 DeferredRegister.Blocks 实例
public static final DeferredBlock<Block> MY_BETTER_BLOCK = BLOCKS.register(
    "my_better_block", 
    registryName -> new Block(BlockBehaviour.Properties.of()
        //highlight-start
        .setId(ResourceKey.create(Registries.BLOCK, registryName))
        .destroyTime(2.0f)
        .explosionResistance(10.0f)
        .sound(SoundType.GRAVEL)
        .lightLevel(state -> 7)
        //highlight-end
    ));
```

有关进一步的文档，请参阅 `BlockBehaviour.Properties` 的源代码。有关更多示例或查看 Minecraft 使用的值，请查看 `Blocks` 类。

:::note
理解世界中的方块与库存中的方块不同是很重要的。在库存中看起来像方块的东西实际上是一个 `BlockItem`，一种特殊类型的[物品][item]，在使用时放置一个方块。这也意味着诸如创造标签或最大堆叠大小之类的东西是由相应的 `BlockItem` 处理的。

`BlockItem` 必须与方块分开注册。这是因为方块不一定需要物品，例如如果它不打算被收集（例如火）。
:::

### 更多功能

直接使用 `Block` 仅允许非常基本的方块。如果你想添加功能，例如玩家交互或不同的碰撞箱，则需要一个继承 `Block` 的自定义类。`Block` 类有许多可以重写的方法来执行不同的操作；有关更多信息，请参阅类 `Block`、`BlockBehaviour` 和 `IBlockExtension`。另请参阅[使用方块][usingblocks]部分，了解方块的一些最常见用例。

如果你想制作一个具有不同变体的方块（例如具有底部、顶部和双变体的台阶），你应该使用[方块状态][blockstates]。最后，如果你想要一个存储额外数据的方块（例如存储物品的箱子），应该使用[方块实体][blockentities]。这里的经验法则是，如果你有有限且相对较少的状态（= 最多几百种状态），使用方块状态；如果你有无限或接近无限的状态，使用方块实体。

#### 方块类型

方块类型(Block types)是用于序列化和反序列化方块对象的[`MapCodec`][codec]。此 `MapCodec` 通过 `BlockBehaviour#codec` 设置并[注册][registration]到方块类型注册表。目前，它的唯一用途是在生成方块列表报告时。如果方块子类仅接受 `BlockBehaviour.Properties`，则可以使用 `BlockBehaviour#simpleCodec` 创建 `MapCodec`。

```java
// 对于某些方块子类
public class SimpleBlock extends Block {
    public SimpleBlock(BlockBehavior.Properties properties) {
        // ...
    }

    @Override
    public MapCodec<SimpleBlock> codec() {
        return SIMPLE_CODEC.get();
    }
}

// 在某些注册类中
public static final DeferredRegister<MapCodec<? extends Block>> REGISTRAR = DeferredRegister.create(BuiltInRegistries.BLOCK_TYPE, "yourmodid");

public static final Supplier<MapCodec<SimpleBlock>> SIMPLE_CODEC = REGISTRAR.register(
    "simple",
    () -> BlockBehaviour.simpleCodec(SimpleBlock::new)
);
```

如果方块子类包含更多参数，则应使用[`RecordCodecBuilder#mapCodec`][codec]创建 `MapCodec`，将 `BlockBehaviour#propertiesCodec` 传递给 `BlockBehaviour.Properties` 作为参数。

```java
// 对于某些方块子类
public class ComplexBlock extends Block {
    public ComplexBlock(int value, BlockBehavior.Properties properties) {
        // ...
    }

    @Override
    public MapCodec<ComplexBlock> codec() {
        return COMPLEX_CODEC.get();
    }

    public int getValue() {
        return this.value;
    }
}

// 在某些注册类中
public static final DeferredRegister<MapCodec<? extends Block>> REGISTRAR = DeferredRegister.create(BuiltInRegistries.BLOCK_TYPE, "yourmodid");

public static final Supplier<MapCodec<ComplexBlock>> COMPLEX_CODEC = REGISTRAR.register(
    "simple",
    () -> RecordCodecBuilder.mapCodec(instance ->
        instance.group(
            Codec.INT.fieldOf("value").forGetter(ComplexBlock::getValue),
            BlockBehaviour.propertiesCodec() // 表示 BlockBehavior.Properties 参数
        ).apply(instance, ComplexBlock::new)
    )
);
```

:::note
尽管方块类型目前基本上没使用，但预计在未来会变得更加重要，因为 Mojang 继续向以编解码器为中心的结构迈进。
:::

### `DeferredRegister.Blocks`

我们已经讨论了如何创建一个 `DeferredRegister.Blocks` [上文][above]，以及它返回 `DeferredBlock`。现在，让我们看看这个专门的 `DeferredRegister` 提供的其他实用工具。让我们从 `#registerBlock` 开始：

```java
public static final DeferredRegister.Blocks BLOCKS = DeferredRegister.createBlocks("yourmodid");

public static final DeferredBlock<Block> EXAMPLE_BLOCK = BLOCKS.register(
    "example_block", registryName -> new Block(
        BlockBehaviour.Properties.of()
            // 必须在方块上设置 ID
            .setId(ResourceKey.create(Registries.BLOCK, registryName))
    )
);

// 与上面相同，只是方块属性(block properties)是提前构造的。
// setId 也在属性对象上内部调用。
public static final DeferredBlock<Block> EXAMPLE_BLOCK = BLOCKS.registerBlock(
    "example_block",
    Block::new, // 属性将传递给的工厂类。
    BlockBehaviour.Properties.of() // 要使用的属性。
);
```

如果你想使用 `Block::new`，你可以完全省略它：

```java
public static final DeferredBlock<Block> EXAMPLE_BLOCK = BLOCKS.registerSimpleBlock(
    "example_block",
    BlockBehaviour.Properties.of() // 要使用的属性。
);
```

这与前面的示例完全相同，但稍微简短一些。当然，如果你想使用 `Block` 的子类而不是 `Block` 本身，你将不得不使用前一种方法。

### 资源

如果你注册了你的方块并将其放置在世界中，你会发现它缺少诸如纹理之类的东西。这是因为[纹理][textures]等是由 Minecraft 的资源系统处理的。要将纹理应用到方块，你必须提供一个[模型][model]和一个[方块状态文件][bsfile]，将方块与纹理和形状关联起来。请阅读链接的文章以获取更多信息。

## 使用方块

方块很少直接用于执行操作。实际上，Minecraft 中可能最常见的两个操作 - 获取某个位置的方块和在某个位置设置方块 - 使用的是方块状态(blockstates)，而不是方块。一般的设计方法是让方块定义行为，然后通过方块状态运行行为。因此，`BlockState` 通常作为参数传递给 `Block` 的方法。有关如何使用方块状态以及如何从方块获取方块状态的更多信息，请参阅[使用方块状态][usingblockstates]。

在几种情况下，会在不同时间使用 `Block` 的多个方法。以下小节列出了最常见的与方块相关的检查。除非另有说明，否则所有方法都在逻辑两侧调用，并且应该在两侧返回相同的结果。

### 放置方块

方块放置逻辑从 `BlockItem#useOn`（或其某些子类的实现，例如用于睡莲的 `PlaceOnWaterBlockItem`）调用。有关游戏如何到达此处的更多信息，请参阅[右键单击物品][rightclick]。实际上，这意味着一旦右键单击了一个 `BlockItem`（例如鹅卵石物品），就会调用此行为。

- 检查几个先决条件，例如你是否不在旁观者模式中，是否启用了方块的所有必需功能标志，或者目标位置是否不在世界边界之外。如果至少有一个检查失败，检查结束。
- 对当前在尝试放置方块的位置的方块调用 `BlockBehaviour#canBeReplaced`。如果返回 `false`，检查结束。这里返回 `true` 的显著情况是高草或雪层。
- 调用 `Block#getStateForPlacement`。这是根据上下文（包括位置、旋转和放置方块的侧面）可以返回不同方块状态的地方。这对于可以以不同方向放置的方块非常有用。
- 使用在上一步中获得的方块状态调用 `BlockBehaviour#canSurvive`。如果返回 `false`，检查结束。
- 通过 `Level#setBlock` 调用将方块状态设置到关卡中。
  - 在该 `Level#setBlock` 调用中，调用 `BlockBehaviour#onPlace`。
- 调用 `Block#setPlacedBy`。

### 破坏方块

破坏方块稍微复杂一些，因为它需要时间。该过程可以大致分为三个阶段：“启动”、“挖掘”和“实际破坏”。

- 当单击左键时，进入“启动”阶段。
- 现在需要按住左键，进入“挖掘”阶段。**此阶段的方法每个 tick 都会被调用。**
- 如果“继续”阶段未被中断（通过释放左键）并且方块被破坏，则进入“实际破坏”阶段。

或者对于那些更喜欢伪代码的人：

```java
leftClick();
initiatingStage();
while (leftClickIsBeingHeld()) {
    miningStage();
    if (blockIsBroken()) {
        actuallyBreakingStage();
        break;
    }
}
```

以下小节进一步分解这些阶段为实际方法调用。有关游戏如何从左键单击到此检查的更多信息，请参阅[左键单击物品][leftclick]。

#### “启动”阶段

- 检查几个先决条件，例如你是否不在旁观者模式中，是否启用了主手中的 `ItemStack` 的所有必需功能标志，或者相关方块是否不在世界边界之外。如果至少有一个检查失败，检查结束。
- 触发 `PlayerInteractEvent.LeftClickBlock`。如果事件被取消，检查结束。
  - 请注意，当事件在客户端上被取消时，不会向服务器发送任何数据包，因此服务器上不会运行任何逻辑。
  - 但是，在服务器上取消此事件仍会导致客户端代码运行，这可能导致不同步！
- 调用 `Block#attack`。

#### “挖掘”阶段

- 触发 `PlayerInteractEvent.LeftClickBlock`。如果事件被取消，检查步骤移动到“完成”阶段。
  - 请注意，当事件在客户端上被取消时，不会向服务器发送任何数据包，因此服务器上不会运行任何逻辑。
  - 但是，在服务器上取消此事件仍会导致客户端代码运行，这可能导致不同步！
- 调用 `Block#getDestroyProgress` 并将其添加到内部破坏进度计数器。
  - `Block#getDestroyProgress` 返回一个介于 0 和 1 之间的浮点值，表示每个 tick 应增加多少破坏进度计数器。
- 根据需要更新进度覆盖（裂纹纹理）。
- 如果破坏进度大于 1.0（即完成，即方块应该被破坏），则退出“挖掘”阶段并进入“实际破坏”阶段。

#### “实际破坏”阶段

- 调用 `Item#canAttackBlock`。如果返回 `false`（确定方块不应被破坏），检查步骤移动到“完成”阶段。
- 如果方块是 `GameMasterBlock` 的实例，则调用 `Player#canUseGameMasterBlocks`。这确定玩家是否有能力破坏仅限创造的方块。如果 `false`，检查步骤移动到“完成”阶段。
- 仅限服务器：调用 `Player#blockActionRestricted`。这确定当前玩家是否不能破坏方块。如果 `true`，检查步骤移动到“完成”阶段。
- 仅限服务器：触发 `BlockEvent.BreakEvent`。如果被取消或 `getExpToDrop` 返回 -1，检查步骤移动到“完成”阶段。初始取消状态由上述三个方法确定。
  - 仅限服务器：触发 `PlayerEvent.HarvestCheck`。如果 `canHarvest` 返回 `false` 或传递给破坏事件的 `BlockState` 为 null，则事件的初始经验值为 0。
  - 仅限服务器：如果 `PlayerEvent.HarvestCheck#canHarvest` 返回 `true`，调用 `IBlockExtension#getExpDrop`。此值传递给 `BlockEvent.BreakEvent#getExpToDrop` 以供检查步骤后续使用。
- 仅限服务器：调用 `IBlockExtension#canHarvestBlock`。这确定方块是否可以被收获，即破坏时是否有掉落物。
- 调用 `IBlockExtension#onDestroyedByPlayer`。如果返回 `false`，检查步骤移动到“完成”阶段。在该 `IBlockExtension#onDestroyedByPlayer` 调用中：
  - 调用 `Block#playerWillDestroy`。
  - 通过 `Level#setBlock` 调用将方块状态从关卡中移除，`Blocks.AIR.defaultBlockState()` 作为方块状态参数。
    - 在该 `Level#setBlock` 调用中，调用 `Block#onRemove`。
- 调用 `Block#destroy`。
- 仅限服务器：如果之前对 `IBlockExtension#canHarvestBlock` 的调用返回 `true`，调用 `Block#playerDestroy`。
  - 仅限服务器：调用 `Block#dropResources`。这确定方块在挖掘时的掉落物。
    - 仅限服务器：触发 `BlockDropsEvent`。如果事件被取消，则方块破坏时不会掉落任何物品。否则，`BlockDropsEvent#getDrops` 中的每个 `ItemEntity` 都会被添加到当前关卡。
- 仅限服务器：调用 `Block#popExperience`，使用之前 `IBlockExtension#getExpDrop` 调用的结果，如果该调用返回的值大于 0。

#### 挖掘速度

挖掘速度根据方块的硬度、使用的[工具][tool]的速度以及几个实体[属性][attributes]按照以下规则计算：

```java
// 这将返回工具的挖掘速度，如果持有的物品为空、不是工具或不适用于正在破坏的方块，则返回 1。
float destroySpeed = item.getDestroySpeed(blockState);
// 如果我们有一个适用的工具，添加 minecraft:mining_efficiency 属性作为加法修饰符。
if (destroySpeed > 1) {
    destroySpeed += player.getAttributeValue(Attributes.MINING_EFFICIENCY);
}
// 应用急迫或潮涌能量的效果。
if (player.hasEffect(MobEffects.DIG_SPEED) || player.hasEffect(MobEffects.CONDUIT_POWER)) {
    int haste = player.hasEffect(MobEffects.DIG_SPEED)
        ? player.getEffect(MobEffects.DIG_SPEED).getAmplifier()
        : 0;
    int conduitPower = player.hasEffect(MobEffects.CONDUIT_POWER)
        ? player.getEffect(MobEffects.CONDUIT_POWER).getAmplifier()
        : 0;
    int amplifier = Math.max(haste, conduitPower);
    destroySpeed *= 1 + (amplifier + 1) * 0.2f;
}
// 应用挖掘疲劳效果。
if (player.hasEffect(MobEffects.DIG_SLOWDOWN)) {
    destroySpeed *= switch (player.getEffect(MobEffects.DIG_SLOWDOWN).getAmplifier()) {
        case 0 -> 0.3F;
        case 1 -> 0.09F;
        case 2 -> 0.0027F;
        default -> 8.1E-4F;
    };
}
// 添加 minecraft:block_break_speed 属性作为乘法修饰符。
destroySpeed *= player.getAttributeValue(Attributes.BLOCK_BREAK_SPEED);
// 如果玩家在水下，应用水下挖掘速度惩罚(乘算)。
if (player.isEyeInFluid(FluidTags.WATER)) {
    destroySpeed *= player.getAttributeValue(Attributes.SUBMERGED_MINING_SPEED);
}
// 如果玩家试图在空中破坏方块，使玩家挖掘速度降低 5 倍。
if (!player.onGround()) {
    destroySpeed /= 5;
}
destroySpeed = /* 在此处触发 PlayerEvent.BreakSpeed 事件，允许模组开发者进一步修改此值。 */;
return destroySpeed;
```

可以在 `Player#getDestroySpeed` 中找到此代码的确切参考。

### 刻(Tick)

刻(Tick)是一种机制，每 1 / 20 秒或 50 毫秒（“一个 tick”）更新游戏的部分内容。方块提供了不同的 tick 方法，这些方法以不同的方式调用。

#### 服务器 Tick 和 Tick 调度

`BlockBehaviour#tick` 在两种情况下调用：默认的[随机刻][randomtick]（见下文），或调度刻。调度刻可以通过 `Level#scheduleTick(BlockPos, Block, int)` 创建，其中 `int` 表示延迟。这在许多地方被原版使用，例如大型垂滴叶的倾斜机制严重依赖于此系统。其他显著的用户是各种红石组件。

#### 客户端 Tick

`Block#animateTick` 仅在客户端上调用，每帧调用一次。这是客户端专属行为发生的地方，例如火把粒子生成。

#### 天气 Tick

天气 Tick 由 `Block#handlePrecipitation` 处理，并独立于常规 Tick 运行。它仅在服务器上调用，仅在某种形式下雨时调用，概率为 1/16。这用于例如在雨或雪中填充的炼药锅。

#### 随机刻

随机刻系统独立于常规 Tick 运行。随机刻必须通过调用 `BlockBehaviour.Properties#randomTicks()` 方法在方块的 `BlockBehaviour.Properties` 中启用。这使方块成为随机 Tick 机制的一部分。

随机刻每个 Tick 对区块中的一组方块发生。该组方块的数量由 `randomTickSpeed` 游戏规则定义。其默认值为 3，每个 Tick 从区块中随机选择 3 个方块。如果这些方块启用了随机刻，则调用它们各自的 `BlockBehaviour#randomTick` 方法。

随机刻被 Minecraft 中的广泛机制使用，例如植物生长、冰和雪融化或铜氧化。

[above]: #唯一的方块
[attributes]: ../entities/attributes.md
[below]: #deferredregisterblocks
[blockentities]: ../blockentities/index.md
[blockstates]: states.md
[bsfile]: ../resources/client/models/index.md#blockstate-files
[codec]: ../datastorage/codecs.md#records
[item]: ../items/index.md
[leftclick]: ../items/interactions.md#left-clicking-an-item
[model]: ../resources/client/models/index.md
[randomtick]: #随机刻
[registration]: ../concepts/registries.md#methods-for-registering
[resources]: ../resources/index.md#assets
[rightclick]: ../items/interactions.md#right-clicking-an-item
[sounds]: ../resources/client/sounds.md
[textures]: ../resources/client/textures.md
[tool]: ../items/tools.md
[usingblocks]: #使用方块
[usingblockstates]: states.md#使用方块状态
