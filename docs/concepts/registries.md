---
sidebar_position: 1
---
# 注册表 Registry

注册是将 MOD 的对象（例如 [物品][item]、[方块][block]、实体等）告知游戏的过程。注册非常重要，因为如果没有注册，游戏将不会知道这些对象，这将导致无法解释的行为和崩溃。

简单来说，注册表是一个围绕映射的包装器，它将注册名称（见下文）映射到已注册的对象，通常称为注册条目。注册名称在同一注册表中必须是唯一的，但相同的注册名称可能存在于多个注册表中。最常见的例子是方块（在 `BLOCKS` 注册表中）具有相同注册名称的物品形式（在 `ITEMS` 注册表中）。

每个注册对象都有一个唯一的名称，称为其注册名称。该名称表示为 [`ResourceLocation`][resloc]。例如，泥土方块的注册名称是 `minecraft:dirt`，僵尸的注册名称是 `minecraft:zombie`。MOD 的对象当然不会使用 `minecraft` 命名空间；而是使用它们的 MOD ID。

## 原版与 MOD

为了理解 NeoForge 注册表系统中的一些设计决策，我们首先来看 Minecraft 是如何做到这一点的。我们将以方块注册表为例，因为大多数其他注册表的工作方式相同。

注册表通常注册 [单例][singleton]。这意味着所有注册条目只存在一次。例如，你在游戏中看到的所有石头方块实际上都是同一个石头方块，显示多次。如果你需要石头方块，可以通过引用已注册的方块实例来获取它。

Minecraft 在 `Blocks` 类中注册所有方块。通过 `register` 方法，调用 `Registry#register()`，第一个参数是位于 `BuiltInRegistries.BLOCK` 的方块注册表。在所有方块注册完成后，Minecraft 会根据方块列表执行各种检查，例如自检，以验证所有方块是否已加载模型。

这一切能够正常工作的主要原因是 `Blocks` 类在 Minecraft 启动时足够早地被加载。MOD 不会被 Minecraft 自动加载，因此需要一些变通方法。

## 注册方法

NeoForge 提供了两种注册对象的方法：`DeferredRegister` 类和 `RegisterEvent`。请注意，前者是后者的包装器，推荐使用以防止错误。

### `DeferredRegister`

首先创建 `DeferredRegister`：

```java
public static final DeferredRegister<Block> BLOCKS = DeferredRegister.create(
        // 要使用的注册表。
        // Minecraft 的注册表可以在 BuiltInRegistries 中找到，NeoForge 的注册表可以在 NeoForgeRegistries 中找到
        // MOD 也可能添加自己的注册表，请参考各个 MOD 的文档或源代码以找到它们
        BuiltInRegistries.BLOCKS,
        // MOD ID
        ExampleMod.MOD_ID
);
```

我们可以使用以下方法之一将我们的注册条目添加为 static final 字段（有关 `new Block()` 中应添加哪些参数，请参阅 [关于方块的文档][block]）：

```java
public static final DeferredHolder<Block, Block> EXAMPLE_BLOCK_1 = BLOCKS.register(
        // 注册名称
        "example_block",
        // 要注册的对象的 Supplier
        () -> new Block(...)
);

public static final DeferredHolder<Block, SlabBlock> EXAMPLE_BLOCK_2 = BLOCKS.register(
        // 注册名称
        "example_block",
        // 一个函数，根据其注册名称（作为 ResourceLocation）创建要注册的对象
        registryName -> new SlabBlock(...)
);
```

类 `DeferredHolder<R, T extends R>` 保存我们的对象。类型参数 `R` 是我们注册到的注册表的类型（在我们的例子中是 `Block`）。类型参数 `T` 是 Supplier 的类型。由于我们在第一个示例中直接注册了一个 `Block`，因此我们将 `Block` 作为第二个参数提供。如果我们要注册 `Block` 的子类对象，例如 `SlabBlock`（如第二个示例所示），我们将在此处传入 `SlabBlock`。

`DeferredHolder<R, T extends R>` 是 `Supplier<T>` 的子类。当我们需要获取已注册的对象时，可以调用 `DeferredHolder#get()`。`DeferredHolder` 继承 `Supplier` ，所以也可以用 `Supplier` 作为字段的类型。这样，上面的代码将变为以下内容：

```java
public static final Supplier<Block> EXAMPLE_BLOCK_1 = BLOCKS.register(
        // 注册名称
        "example_block",
        // 要注册的对象的 Supplier
        () -> new Block(...)
);

public static final Supplier<SlabBlock> EXAMPLE_BLOCK_2 = BLOCKS.register(
        // 注册名称。
        "example_block",
        // 一个函数，根据其注册名称（作为 ResourceLocation）创建要注册的对象。
        registryName -> new SlabBlock(...)
);
```

:::note
请注意，某些地方明确要求 `Holder` 或 `DeferredHolder`，而不接受任何 `Supplier`。如果你需要这两种类型中的任何一种，最好根据需要将 `Supplier` 的类型改回 `Holder` 或 `DeferredHolder`。
:::

最后，由于整个系统是围绕注册事件的包装器，我们需要告诉 `DeferredRegister` 根据需要将自己附加到注册事件中：

```java
//构造函数
public ExampleMod(IEventBus modBus) {
    //highlight-next-line
    ExampleBlocksClass.BLOCKS.register(modBus);
    //其他操作
}
```

:::info
对于方块、物品和数据组件，有专门的 `DeferredRegister` 变体，它们提供了辅助方法：分别是 [`DeferredRegister.Blocks`][defregblocks]、[`DeferredRegister.Items`][defregitems] 和 [`DeferredRegister.DataComponents`][defregcomp]。
:::

### `RegisterEvent`

`RegisterEvent` 是注册对象的第二种方式。这个 [事件][event] 会在每个注册表上触发，触发时机在 MOD 构造函数之后（因为 `DeferredRegister` 会在构造函数中注册其内部事件处理程序）和配置加载之前。`RegisterEvent` 在 MOD 事件总线上触发。

```java
@SubscribeEvent
public void register(RegisterEvent event) {
    event.register(
            // 这是注册表的注册键。
            // 对于原版注册表，可以从 BuiltInRegistries 获取；
            // 对于 NeoForge 注册表，可以从 NeoForgeRegistries.Keys 获取。
            BuiltInRegistries.BLOCKS,
            // 在这里注册你的对象。
            registry -> {
                registry.register(ResourceLocation.fromNamespaceAndPath(MODID, "example_block_1"), new Block(...));
                registry.register(ResourceLocation.fromNamespaceAndPath(MODID, "example_block_2"), new Block(...));
                registry.register(ResourceLocation.fromNamespaceAndPath(MODID, "example_block_3"), new Block(...));
            }
    );
}
```

## 查询注册表

有时，你会遇到需要通过给定的 ID 获取已注册对象的情况。或者，你可能想获取某个已注册对象的 ID。由于注册表本质上是 ID（`ResourceLocation`）到唯一对象的映射，即一个可逆的映射，因此这两种操作都是可行的：

```java
BuiltInRegistries.BLOCKS.getValue(ResourceLocation.fromNamespaceAndPath("minecraft", "dirt")); // 返回泥土方块
BuiltInRegistries.BLOCKS.getKey(Blocks.DIRT); // 返回 ResourceLocation "minecraft:dirt"

// 假设 ExampleBlocksClass.EXAMPLE_BLOCK.get() 是一个 ID 为 "yourmodid:example_block" 的 Supplier<Block>
BuiltInRegistries.BLOCKS.getValue(ResourceLocation.fromNamespaceAndPath("yourmodid", "example_block")); // 返回示例方块
BuiltInRegistries.BLOCKS.getKey(ExampleBlocksClass.EXAMPLE_BLOCK.get()); // 返回 ResourceLocation "yourmodid:example_block"
```

如果你只想检查某个对象是否存在，这也是可行的，尽管只能通过键来检查：

```java
BuiltInRegistries.BLOCKS.containsKey(ResourceLocation.fromNamespaceAndPath("minecraft", "dirt")); // true
BuiltInRegistries.BLOCKS.containsKey(ResourceLocation.fromNamespaceAndPath("create", "brass_ingot")); // 只有当 Create 安装时，返回 true
```

如最后一个示例所示，这适用于任何 MOD ID，因此是检查另一个 MOD 中的某个物品是否存在的完美方式。

最后，我们还可以遍历注册表中的所有条目，无论是遍历键还是遍历条目（条目使用 Java 的 `Map.Entry` 类型）：

```java
for (ResourceLocation id : BuiltInRegistries.BLOCKS.keySet()) {
    // ...
}
for (Map.Entry<ResourceKey<Block>, Block> entry : BuiltInRegistries.BLOCKS.entrySet()) {
    // ...
}
```

:::note
查询操作始终使用原版的 `Registry`，而不是 `DeferredRegister`。这是因为 `DeferredRegister` 仅仅是注册工具。
:::

:::danger
查询操作仅在注册完成后使用才是安全的。**在注册仍在进行时，不要查询注册表！**
:::

## 自定义注册表

自定义注册表允许你指定其他 MOD 可能希望插入的附加系统。例如，如果你的 MOD 添加了法术，你可以将法术设为注册表，从而允许其他 MOD 向你的 MOD 添加法术，而无需你做任何其他事情。它还允许你自动执行一些操作，例如同步条目。

让我们从创建 [注册表键 ResourceKey][resourcekey] 和注册表本身开始：

```java
// 我们以法术为例来演示注册表，而不涉及法术的具体细节（因为它并不重要）
// 当然，所有提到法术的地方都可以并且应该替换为你的注册表实际内容
public static final ResourceKey<Registry<Spell>> SPELL_REGISTRY_KEY = ResourceKey.createRegistryKey(ResourceLocation.fromNamespaceAndPath("yourmodid", "spells"));
public static final Registry<YourRegistryContents> SPELL_REGISTRY = new RegistryBuilder<>(SPELL_REGISTRY_KEY)
        // 如果你想启用整数 ID 同步，用于网络通信
        // 这些应仅用于网络通信上下文中，例如数据包或纯网络相关的 NBT 数据
        .sync(true)
        // 默认键。类似于 minecraft:air 对于方块。这是可选的
        .defaultKey(ResourceLocation.fromNamespaceAndPath("yourmodid", "empty"))
        // 有效限制最大数量。通常不推荐使用，但在网络通信等场景中可能有意义
        .maxId(256)
        // 构建注册表
        .create();
```

然后，通过在 `NewRegistryEvent` 中将注册表注册到根注册表来告诉游戏该注册表存在：

```java
@SubscribeEvent
static void registerRegistries(NewRegistryEvent event) {
    event.register(SPELL_REGISTRY);
}
```

现在，你可以像注册其他注册表一样，通过 `DeferredRegister` 和 `RegisterEvent` 注册新的注册表内容：

```java
public static final DeferredRegister<Spell> SPELLS = DeferredRegister.create(SPELL_REGISTRY, "yourmodid");
public static final Supplier<Spell> EXAMPLE_SPELL = SPELLS.register("example_spell", () -> new Spell(...));

// 或者:
@SubscribeEvent
public static void register(RegisterEvent event) {
    event.register(SPELL_REGISTRY_KEY, registry -> {
        registry.register(ResourceLocation.fromNamespaceAndPath("yourmodid", "example_spell"), () -> new Spell(...));
    });
}
```

## 数据包注册表

数据包注册表（也称为动态注册表，或根据其主要用例称为世界生成注册表）是一种特殊的注册表，它在世界加载时从 [数据包][datapack] JSON 文件加载数据，而不是在游戏启动时加载。默认的数据包注册表最显著地包括大多数世界生成注册表，以及其他一些注册表。

数据包注册表允许其内容在 JSON 文件中指定。这意味着不需要编写代码（除非你不想自己编写 JSON 文件，而是使用 [数据生成][datagen]）。每个数据包注册表都有一个与之关联的 [`Codec`][codec]，用于序列化，每个注册表的 ID 决定了其数据包路径：

- Minecraft 的数据包注册表使用格式 `data/yourmodid/registrypath`（例如 `data/yourmodid/worldgen/biome`，其中 `worldgen/biome` 是注册表路径）。
- 所有其他数据包注册表（NeoForge 或 MOD 的）使用格式 `data/yourmodid/registrynamespace/registrypath`（例如 `data/yourmodid/neoforge/biome_modifier`，其中 `neoforge` 是注册表命名空间，`biome_modifier` 是注册表路径）。

数据包注册表可以从 `RegistryAccess` 中获取。如果在服务器上，可以通过调用 `ServerLevel#registryAccess()` 获取 `RegistryAccess`；如果在客户端上，可以通过调用 `Minecraft.getInstance().getConnection()#registryAccess()` 获取（后者仅在连接到世界时有效，否则连接将为 null）。这些调用的结果可以像任何其他注册表一样使用，以获取特定元素或遍历内容。

### 自定义数据包注册表

自定义数据包注册表不需要构建 `Registry`。相反，它们只需要一个注册表键和至少一个 [`Codec`][codec] 来（反）序列化其内容。继续之前的法术示例，将我们的法术注册表注册为数据包注册表的过程如下：

```java
public static final ResourceKey<Registry<Spell>> SPELL_REGISTRY_KEY = ResourceKey.createRegistryKey(ResourceLocation.fromNamespaceAndPath("yourmodid", "spells"));

@SubscribeEvent
public static void registerDatapackRegistries(DataPackRegistryEvent.NewRegistry event) {
    event.dataPackRegistry(
            // 注册表键。
            SPELL_REGISTRY_KEY,
            // 注册表内容的 codec 。
            Spell.CODEC,
            // 注册表内容的网络 codec 。通常与普通 codec 相同。
            // 可能是普通 codec 的简化版本，省略了客户端不需要的数据。
            // 可以为 null。如果为 null，注册表条目将不会同步到客户端。
            // 可以省略，这在功能上等同于传递 null（调用带有两个参数的方法重载，该方法重载会将 null 传递给正常的三个参数方法）。
            Spell.CODEC,
            // 一个 consumer，通过 RegistryBuilder 配置构建的注册表。
            // 可以省略，这在功能上等同于传递 builder -> {}。
            builder -> builder.maxId(256)
    );
}
```

### 数据包注册表的数据生成

由于手动编写所有 JSON 文件既繁琐又容易出错，NeoForge 提供了一个 [数据提供器][datagenindex] 来为你生成 JSON 文件。这适用于内置和你自己的数据包注册表。

首先，我们创建一个 `RegistrySetBuilder` 并将我们的条目添加到其中（一个 `RegistrySetBuilder` 可以包含多个注册表的条目）：

```java
new RegistrySetBuilder()
    .add(Registries.CONFIGURED_FEATURE, bootstrap -> {
        // 通过引导上下文注册 ConfiguredFeature（见下文）
    })
    .add(Registries.PLACED_FEATURE, bootstrap -> {
        // 通过引导上下文注册 PlacedFeature（见下文）
    });
```

`bootstrap` lambda 参数是我们实际用来注册对象的。它的类型是 `BootstrapContext`。要注册一个对象，我们调用它的 `#register` 方法，如下所示：

```java
// 对象的 ResourceKey
public static final ResourceKey<ConfiguredFeature<?, ?>> EXAMPLE_CONFIGURED_FEATURE = ResourceKey.create(
    Registries.CONFIGURED_FEATURE,
    ResourceLocation.fromNamespaceAndPath(MOD_ID, "example_configured_feature")
);

new RegistrySetBuilder()
    .add(Registries.CONFIGURED_FEATURE, bootstrap -> {
        bootstrap.register(
            // ConfiguredFeature 的 ResourceKey
            EXAMPLE_CONFIGURED_FEATURE,
            // 实际的 ConfiguredFeature
            new ConfiguredFeature<>(Feature.ORE, new OreConfiguration(...))
        );
    })
    .add(Registries.PLACED_FEATURE, bootstrap -> {
        // ...
    });
```

如果需要，`BootstrapContext` 还可以用于从另一个注册表中查找条目：

```java
public static final ResourceKey<ConfiguredFeature<?, ?>> EXAMPLE_CONFIGURED_FEATURE = ResourceKey.create(
    Registries.CONFIGURED_FEATURE,
    ResourceLocation.fromNamespaceAndPath(MOD_ID, "example_configured_feature")
);
public static final ResourceKey<PlacedFeature> EXAMPLE_PLACED_FEATURE = ResourceKey.create(
    Registries.PLACED_FEATURE,
    ResourceLocation.fromNamespaceAndPath(MOD_ID, "example_placed_feature")
);

new RegistrySetBuilder()
    .add(Registries.CONFIGURED_FEATURE, bootstrap -> {
        bootstrap.register(EXAMPLE_CONFIGURED_FEATURE, ...);
    })
    .add(Registries.PLACED_FEATURE, bootstrap -> {
        HolderGetter<ConfiguredFeature<?, ?>> otherRegistry = bootstrap.lookup(Registries.CONFIGURED_FEATURE);
        bootstrap.register(EXAMPLE_PLACED_FEATURE, new PlacedFeature(
            otherRegistry.getOrThrow(EXAMPLE_CONFIGURED_FEATURE), // 获取 ConfiguredFeature
            List.of() // 放置时无操作 - 替换为你的放置参数
        ));
    });
```

最后，我们在实际的数据提供器中使用 `RegistrySetBuilder`，并将该数据提供器注册到事件中：

```java
@SubscribeEvent
public static void onGatherData(GatherDataEvent.Client event) {
    // 将生成的注册表 (Registry) 对象添加到当前的查找提供器中，以便在其他数据生成中使用。
    this.createDatapackRegistryObjects(
        // 用于生成数据的 RegistrySetBuilder。
        new RegistrySetBuilder().add(...),
        // （可选）biconsumer，用于加载与资源键关联的对象的条件。
        conditions -> {
            conditions.accept(resourceKey, condition);
        },
        // （可选）我们为其生成条目的 MOD ID 集合。
        // 默认情况下，提供当前 MOD 容器的 MOD ID。
        Set.of("yourmodid")
    );

    // 你可以通过调用 `#create*` 方法之一或通过 `#getLookupProvider` 获取实际的查找提供器来使用生成的条目。
    // ...
}
```

[block]: ../blocks/index.md
[blockentity]: ../blockentities/index.md
[codec]: ../datastorage/codecs.md
[datagen]: #data-generation-for-datapack-registries
[datagenindex]: ../resources/index.md#data-generation
[datapack]: ../resources/index.md#data
[defregblocks]: ../blocks/index.md#deferredregisterblocks-helpers
[defregcomp]: ../items/datacomponents.md#creating-custom-data-components
[defregitems]: ../items/index.md#deferredregisteritems
[event]: events.md
[item]: ../items/index.md
[resloc]: ../misc/resourcelocation.md
[resourcekey]: ../misc/resourcelocation.md#resourcekeys
[singleton]: https://en.wikipedia.org/wiki/Singleton_pattern
