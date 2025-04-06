# 方块状态 (Blockstates)

通常，你会遇到需要为一个方块定义不同状态的情况。例如，小麦作物有八个生长阶段，为每个阶段创建一个单独的方块显然不合适。或者你有一个台阶或类似台阶的方块——一个底部状态，一个顶部状态，以及一个同时包含两者的状态。

这就是方块状态 (blockstates) 的用武之地。方块状态是一种简单的方法，用于表示方块可以具有的不同状态，例如生长阶段或台阶的放置类型。

## 方块状态属性 (Blockstate Properties)

方块状态使用属性系统。一个方块可以有多个不同类型的属性。例如，一个末地传送门框架有两个属性：是否有眼睛 (`eye`, 2 个选项) 和它的放置方向 (`facing`, 4 个选项)。因此，末地传送门框架总共有 8 (2 * 4) 种不同的方块状态：

```plaintext
minecraft:end_portal_frame[facing=north,eye=false]
minecraft:end_portal_frame[facing=east,eye=false]
minecraft:end_portal_frame[facing=south,eye=false]
minecraft:end_portal_frame[facing=west,eye=false]
minecraft:end_portal_frame[facing=north,eye=true]
minecraft:end_portal_frame[facing=east,eye=true]
minecraft:end_portal_frame[facing=south,eye=true]
minecraft:end_portal_frame[facing=west,eye=true]
```

表示方块状态的标准化文本形式为 `blockid[property1=value1,property2=value,...]`，并在原版的某些地方使用，例如命令中。

如果你的方块没有定义任何方块状态属性，它仍然只有一个方块状态——即没有任何属性的那个状态，因为没有属性需要指定。这可以表示为 `minecraft:oak_planks[]` 或简单地 `minecraft:oak_planks`。

与方块一样，每个 `BlockState` 在内存中只存在一次。这意味着可以且应该使用 `==` 来比较 `BlockState`。`BlockState` 也是一个最终类 (final class)，这意味着它不能被继承。**任何功能都应该放在对应的 [方块 (Block)][block] 类中！**

## 何时使用方块状态

### 方块状态 vs. 单独的方块

一个好的经验法则是：**如果它有不同的名字，它应该是一个单独的方块**。例如制作椅子方块：椅子的方向应该是一个属性，而不同类型的木材应该分成不同的方块。因此，你会为每种木材类型创建一个椅子方块，并且每个椅子方块有四种方块状态（每个方向一种）。

### 方块状态 vs. [方块实体 (block entity)][blockentity]

这里的经验法则是：**如果状态是有限的，使用方块状态；如果状态是无限的或接近无限的，使用方块实体 (block entity)**。方块实体 (block entity) 可以存储任意数量的数据，但比方块状态慢。

方块状态和方块实体 (block entity) 可以结合使用。例如，箱子使用方块状态属性来表示方向、是否被水淹没或是否成为双箱，而存储物品、是否打开或与漏斗交互则由方块实体 (block entity) 处理。

对于“多少状态对于方块状态来说太多了”这个问题，没有明确的答案，但建议如果需要超过 8-9 位数据（即几百种状态以上），应该改用方块实体 (block entity)。

## 实现方块状态

要实现一个方块状态属性，在你的方块类中创建或引用一个 `public static final Property<?>` 常量。虽然你可以自由创建自己的 `Property<?>` 实现，但原版代码提供了几个便捷实现，应该能满足大多数用例：

- `IntegerProperty`
  - 实现 `Property<Integer>`。定义一个保存整数值的属性。注意不支持负值。
  - 通过调用 `IntegerProperty#create(String propertyName, int minimum, int maximum)` 创建。
- `BooleanProperty`
  - 实现 `Property<Boolean>`。定义一个保存 `true` 或 `false` 值的属性。
  - 通过调用 `BooleanProperty#create(String propertyName)` 创建。
- `EnumProperty<E extends Enum<E>>`
  - 实现 `Property<E>`。定义一个可以接受枚举类值的属性。
  - 通过调用 `EnumProperty#create(String propertyName, Class<E> enumClass)` 创建。
  - 也可以只使用枚举值的子集（例如 16 种 `DyeColor` 中的 4 种），参见 `EnumProperty#create` 的重载方法。

`BlockStateProperties` 类包含了共享的原版属性，应尽可能使用或引用这些属性，而不是创建自己的属性。

一旦你有了属性常量，在你的方块类中重写 `Block#createBlockStateDefinition(StateDefinition$Builder)` 方法。在该方法中调用 `StateDefinition.Builder#add(YOUR_PROPERTY);`。`StateDefinition.Builder#add` 有一个可变参数，所以如果你有多个属性，可以一次性添加它们。

每个方块也会有一个默认状态。如果没有特别指定，默认状态会使用每个属性的默认值。你可以通过在构造函数中调用 `Block#registerDefaultState(BlockState)` 方法来改变默认状态。

如果你想改变放置方块时使用的 `BlockState`，重写 `Block#getStateForPlacement(BlockPlaceContext)` 方法。这可以用来，例如，根据玩家站立或注视的位置来设置方块的方向。

为了进一步说明这一点，以下是 `EndPortalFrameBlock` 类的相关部分：

```java
public class EndPortalFrameBlock extends Block {
    // 注意：可以直接使用 BlockStateProperties 中的值，而不是在这里再次引用它们
    // 但为了简单和可读性，建议像这样添加常量
    public static final EnumProperty<Direction> FACING = BlockStateProperties.FACING;
    public static final BooleanProperty EYE = BlockStateProperties.EYE;

    public EndPortalFrameBlock(BlockBehaviour.Properties pProperties) {
        super(pProperties);
        // stateDefinition.any() 从内部集合返回一个随机的 BlockState
        // 我们并不关心，因为无论如何我们都会自己设置所有值
        registerDefaultState(stateDefinition.any()
                .setValue(FACING, Direction.NORTH)
                .setValue(EYE, false)
        );
    }

    @Override
    protected void createBlockStateDefinition(StateDefinition.Builder<Block, BlockState> pBuilder) {
        // 这里是属性实际被添加到状态的地方
        pBuilder.add(FACING, EYE);
    }

    @Override
    @Nullable
    public BlockState getStateForPlacement(BlockPlaceContext pContext) {
        // 根据 BlockPlaceContext 确定放置此方块时将使用的状态的代码
    }
}
```

## 使用方块状态

要从 `Block` 获取 `BlockState`，调用 `Block#defaultBlockState()`。默认方块状态可以通过 `Block#registerDefaultState` 更改，如上所述。

你可以通过调用 `BlockState#getValue(Property<?>)` 并传入你想要获取值的属性来获取属性的值。以我们的末地传送门框架为例，代码看起来会是这样：

```java
// `EndPortalFrameBlock.FACING` 是一个 `EnumProperty<Direction>`实例，因此可以用来从 `BlockState` 中获取 `Direction` 值。
Direction direction = endPortalFrameBlockState.getValue(EndPortalFrameBlock.FACING);
```

如果你想获取具有不同属性值的 `BlockState`，只需在现有的方块状态上调用 `BlockState#setValue(Property<T>, T)` 并传入属性和其值。以我们的末地传送门框架为例，代码类似这样：

```java
endPortalFrameBlockState = endPortalFrameBlockState.setValue(EndPortalFrameBlock.FACING, Direction.SOUTH);
```

:::note
`BlockState` 是不可变的。这意味着当你调用 `#setValue(Property<T>, T)` 时，实际上并没有修改方块状态。相反，内部会执行查找操作，并返回你请求的方块状态对象，这是唯一具有这些确切属性值的对象。这也意味着仅仅调用 `state#setValue` 而不将其保存到变量中（例如保存回 `state`）是没有任何效果的。
:::

要从世界中获取 `BlockState`，使用 `Level#getBlockState(BlockPos)`。

### `Level#setBlock`

要在世界中设置 `BlockState`，使用 `Level#setBlock(BlockPos, BlockState, int)`。

`int` 参数需要额外解释，因为它的含义并不直观。它表示所谓的更新标志。

为了帮助正确设置更新标志，`Block` 类中有许多以 `UPDATE_` 为前缀的 `int` 常量。这些常量可以通过按位或运算（例如 `Block.UPDATE_NEIGHBORS | Block.UPDATE_CLIENTS`）组合使用。

- `Block.UPDATE_NEIGHBORS` 向相邻方块发送更新。更具体地说，它会调用 `Block#neighborChanged`，该方法会调用许多方法，其中大多数与红石相关。
- `Block.UPDATE_CLIENTS` 将方块更新同步到客户端。
- `Block.UPDATE_INVISIBLE` 明确表示不在客户端更新。这会覆盖 `Block.UPDATE_CLIENTS`，导致更新不会被同步。方块在服务器上始终会更新。
- `Block.UPDATE_IMMEDIATE` 强制在客户端的主线程上重新渲染。
- `Block.UPDATE_KNOWN_SHAPE` 停止邻居更新的递归。
- `Block.UPDATE_SUPPRESS_DROPS` 禁用该位置旧方块的掉落。
- `Block.UPDATE_MOVE_BY_PISTON` 仅由活塞代码使用，表示方块被活塞移动。这主要用于延迟光照引擎的更新。
- `Block.UPDATE_SKIP_SHAPE_UPDATE_ON_WIRE` 由 `ExperimentalRedstoneWireEvaluator` 使用，表示是否跳过形状更新。仅在电源强度不是由放置引起或信号的原始来源不是当前电线时设置。
- `Block.UPDATE_ALL` 是 `Block.UPDATE_NEIGHBORS | Block.UPDATE_CLIENTS` 的别名。
- `Block.UPDATE_ALL_IMMEDIATE` 是 `Block.UPDATE_NEIGHBORS | Block.UPDATE_CLIENTS | Block.UPDATE_IMMEDIATE` 的别名。
- `Block.UPDATE_NONE` 是 `Block.UPDATE_INVISIBLE` 的别名。

还有一个便捷方法 `Level#setBlockAndUpdate(BlockPos pos, BlockState state)`，它内部调用 `setBlock(pos, state, Block.UPDATE_ALL)`。

[block]: index.md
[blockentity]: ../blockentities/index.md
