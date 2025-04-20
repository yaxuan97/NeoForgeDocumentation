---
sidebar_position: 2
---

# 数据组件

数据组件是用于在`ItemStack`上存储数据的映射中的键值对。每个数据片段，例如烟花爆炸或工具，作为实际对象存储在堆栈上，使这些值可见且可操作，而无需动态转换通用编码实例（例如，`CompoundTag`，`JsonElement`）。

## `DataComponentType`

每个数据组件都有一个关联的`DataComponentType<T>`，其中`T`是组件值的类型。`DataComponentType`表示一个键，用于引用存储的组件值，并包含一些编解码器以处理磁盘和网络的读写操作（如果需要）。

现有组件的列表可以在`DataComponents`中找到。

### 创建自定义数据组件

与`DataComponentType`关联的组件值必须实现`hashCode`和`equals`，并且在存储时应被视为**不可变**。

:::note
组件值可以非常容易地通过记录（record）实现。记录字段是不可变的，并实现了`hashCode`和`equals`。
:::

```java
// 一个记录的示例
public record ExampleRecord(int value1, boolean value2) {}

// 一个类的示例
public class ExampleClass {

    private final int value1;
    // 可以是可变的，但使用时需要小心
    private boolean value2;

    public ExampleClass(int value1, boolean value2) {
        this.value1 = value1;
        this.value2 = value2;
    }

    @Override
    public int hashCode() {
        return Objects.hash(this.value1, this.value2);
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == this) {
            return true;
        } else {
            return obj instanceof ExampleClass ex
                && this.value1 == ex.value1
                && this.value2 == ex.value2;
        }
    }
}
```

标准的`DataComponentType`可以通过`DataComponentType#builder`创建，并使用`DataComponentType.Builder#build`构建。构建器包含三个设置：`persistent`，`networkSynchronized`，`cacheEncoding`。

`persistent`指定了[`Codec`][codec]，用于将组件值读写到磁盘。`networkSynchronized`指定了`StreamCodec`，用于在网络上传输组件值。如果未指定`networkSynchronized`，则`persistent`中提供的`Codec`将被包装并用作[`StreamCodec`][streamcodec]。

:::warning
构建器中必须提供`persistent`或`networkSynchronized`之一；否则将抛出`NullPointerException`。如果不需要通过网络发送数据，则将`networkSynchronized`设置为`StreamCodec#unit`，提供默认的组件值。
:::

`cacheEncoding`缓存了`Codec`的编码结果，以便在组件值未更改的情况下，后续的编码使用缓存值。仅当组件值预计很少或从不更改时才应使用此选项。

`DataComponentType`是注册对象，必须[注册]。

```java
// 使用 ExampleRecord(int, boolean)
// 下面只应使用一个 Codec 和/或 StreamCodec
// 提供多个是为了示例

// 基本编解码器
public static final Codec<ExampleRecord> BASIC_CODEC = RecordCodecBuilder.create(instance ->
    instance.group(
        Codec.INT.fieldOf("value1").forGetter(ExampleRecord::value1),
        Codec.BOOL.fieldOf("value2").forGetter(ExampleRecord::value2)
    ).apply(instance, ExampleRecord::new)
);
public static final StreamCodec<ByteBuf, ExampleRecord> BASIC_STREAM_CODEC = StreamCodec.composite(
    ByteBufCodecs.INT, ExampleRecord::value1,
    ByteBufCodecs.BOOL, ExampleRecord::value2,
    ExampleRecord::new
);

// 如果不需要通过网络发送任何数据的单元流编解码器
public static final StreamCodec<ByteBuf, ExampleRecord> UNIT_STREAM_CODEC = StreamCodec.unit(new ExampleRecord(0, false));


// 在另一个类中
// 专门的 DeferredRegister.DataComponents 简化了数据组件的注册，并避免了在`DataComponentType.Builder`中的一些泛型推断问题
public static final DeferredRegister.DataComponents REGISTRAR = DeferredRegister.createDataComponents(Registries.DATA_COMPONENT_TYPE, "examplemod");

public static final Supplier<DataComponentType<ExampleRecord>> BASIC_EXAMPLE = REGISTRAR.registerComponentType(
    "basic",
    builder -> builder
        // 用于将数据读写到磁盘的编解码器
        .persistent(BASIC_CODEC)
        // 用于通过网络读写数据的编解码器
        .networkSynchronized(BASIC_STREAM_CODEC)
);

/// 组件不会保存到磁盘
public static final Supplier<DataComponentType<ExampleRecord>> TRANSIENT_EXAMPLE = REGISTRAR.registerComponentType(
    "transient",
    builder -> builder.networkSynchronized(BASIC_STREAM_CODEC)
);

// 不会通过网络同步任何数据
public static final Supplier<DataComponentType<ExampleRecord>> NO_NETWORK_EXAMPLE = REGISTRAR.registerComponentType(
   "no_network",
   builder -> builder
        .persistent(BASIC_CODEC)
        // 注意我们在这里使用了一个单元流编解码器
        .networkSynchronized(UNIT_STREAM_CODEC)
);
```

## 组件映射

所有数据组件都存储在`DataComponentMap`中，使用`DataComponentType`作为键，使用对象作为值。`DataComponentMap`的功能类似于只读的`Map`。因此，有方法可以通过`#get`给定的`DataComponentType`获取条目，或者在不存在时提供默认值（通过`#getOrDefault`）。

```java
// 对于某些 DataComponentMap map

// 如果组件存在，将获取染料颜色
// 否则为 null
@Nullable
DyeColor color = map.get(DataComponents.BASE_COLOR);
```

### `PatchedDataComponentMap`

由于默认的`DataComponentMap`仅提供基于读取的操作方法，写入操作通过子类`PatchedDataComponentMap`支持。这包括`#set`设置组件的值或完全`#remove`它。

`PatchedDataComponentMap`使用原型和补丁映射存储更改。原型是一个`DataComponentMap`，包含此映射应具有的默认组件及其值。补丁映射是一个从`DataComponentType`到包含更改的`Optional`值的映射。

```java
// 对于某些 PatchedDataComponentMap map

// 将基础颜色设置为白色
map.set(DataComponents.BASE_COLOR, DyeColor.WHITE);

// 通过以下方式移除基础颜色
// - 如果未提供默认值，则移除补丁
// - 如果有默认值，则设置为空的 optional
map.remove(DataComponents.BASE_COLOR);
```

:::danger
原型和补丁映射都是`PatchedDataComponentMap`哈希码的一部分。因此，映射中的任何组件值都应被视为**不可变**。在修改数据组件的值后，请始终调用`#set`或其下述引用方法之一。
:::

## 组件持有者

所有可以持有数据组件的实例都实现了`DataComponentHolder`。`DataComponentHolder`实际上是`DataComponentMap`中只读方法的委托。

```java
// 对于某些 ItemStack stack

// 委托给 'DataComponentMap#get'
@Nullable
DyeColor color = stack.get(DataComponents.BASE_COLOR);
```

### `MutableDataComponentHolder`

`MutableDataComponentHolder`是由 NeoForge 提供的一个接口，用于支持组件映射的写入方法。Vanilla 和 NeoForge 中的所有实现都使用`PatchedDataComponentMap`存储数据组件，因此`#set`和`#remove`方法也有相同名称的委托。

此外，`MutableDataComponentHolder`还提供了一个`#update`方法，该方法处理获取组件值或提供的默认值（如果未设置），对值进行操作，然后将其设置回映射。操作符可以是`UnaryOperator`，它接受组件值并返回组件值，或者是`BiFunction`，它接受组件值和另一个对象并返回组件值。

```java
// 对于某些 ItemStack stack

FireworkExplosion explosion = stack.get(DataComponents.FIREWORK_EXPLOSION);

// 修改组件值
explosion = explosion.withFadeColors(new IntArrayList(new int[] {1, 2, 3}));

// 由于我们修改了组件值，因此应随后调用'set'
stack.set(DataComponents.FIREWORK_EXPLOSION, explosion);

// 更新组件值（内部调用'set'）
stack.update(
    DataComponents.FIREWORK_EXPLOSION,
    // 如果没有组件值，则为默认值
    FireworkExplosion.DEFAULT,
    // 返回一个新的 FireworkExplosion 以设置
    explosion -> explosion.withFadeColors(new IntArrayList(new int[] {4, 5, 6}))
);

stack.update(
    DataComponents.FIREWORK_EXPLOSION,
    // 如果没有组件值，则为默认值
    FireworkExplosion.DEFAULT,
    // 提供给函数的对象
    new IntArrayList(new int[] {7, 8, 9}),
    // 返回一个新的 FireworkExplosion 以设置
    FireworkExplosion::withFadeColors
);
```

## 向物品添加默认数据组件

尽管数据组件存储在`ItemStack`上，但可以在`Item`上设置默认组件映射，以在构造时传递给`ItemStack`作为原型。可以通过`Item.Properties#component`将组件添加到`Item`。

```java
// 对于某些 DeferredRegister.Items REGISTRAR
public static final Item COMPONENT_EXAMPLE = REGISTRAR.register("component",
    // register 用于其他重载，因为 DataComponentType 尚未注册
    registryName -> new Item(
        new Item.Properties()
        .setId(ResourceKey.create(Registries.ITEM, registryName))
        .component(BASIC_EXAMPLE.get(), new ExampleRecord(24, true))
    )
);
```

如果数据组件应添加到属于 Vanilla 或其他模组的现有物品中，则应在[**MOD事件总线**][modbus]上监听`ModifyDefaultComponentEvent`。该事件提供了`modify`和`modifyMatching`方法，允许修改与相关物品关联的`DataComponentPatch.Builder`。构建器可以`#set`组件或`#remove`现有组件。

```java
// 在MOD事件总线上监听
@SubscribeEvent
public void modifyComponents(ModifyDefaultComponentsEvent event) {
    // 设置组件到甜瓜种子
    event.modify(Items.MELON_SEEDS, builder ->
        builder.set(BASIC_EXAMPLE.get(), new ExampleRecord(10, false))
    );

    // 移除任何具有制作剩余物的物品的组件
    event.modifyMatching(
        item -> !item.getCraftingRemainder().isEmpty(),
        builder -> builder.remove(DataComponents.BUCKET_ENTITY_DATA)
    );
}
```

## 使用自定义组件持有者

要创建自定义数据组件持有者，持有者对象只需实现`MutableDataComponentHolder`并实现缺失的方法。持有者对象必须包含一个表示`PatchedDataComponentMap`的字段，以实现相关方法。

```java
public class ExampleHolder implements MutableDataComponentHolder {

    private int data;
    private final PatchedDataComponentMap components;

    // 可以提供重载以提供映射本身
    public ExampleHolder() {
        this.data = 0;
        this.components = new PatchedDataComponentMap(DataComponentMap.EMPTY);
    }

    @Override
    public DataComponentMap getComponents() {
        return this.components;
    }

    @Nullable
    @Override
    public <T> T set(DataComponentType<? super T> componentType, @Nullable T value) {
        return this.components.set(componentType, value);
    }

    @Nullable
    @Override
    public <T> T remove(DataComponentType<? extends T> componentType) {
        return this.components.remove(componentType);
    }

    @Override
    public void applyComponents(DataComponentPatch patch) {
        this.components.applyPatch(patch);
    }

    @Override
    public void applyComponents(DataComponentMap components) {
        this.components.setAll(components);
    }

    // 其他方法
}
```

### `DataComponentPatch` 和编解码器

为了将组件持久化到磁盘或通过网络发送信息，持有者可以发送整个`DataComponentMap`。然而，这通常是信息的浪费，因为任何默认值将在数据发送到的地方已经存在。因此，我们使用`DataComponentPatch`发送相关数据。`DataComponentPatch`仅包含组件映射的补丁信息，而不包含任何默认值。然后，这些补丁应用于接收方位置的原型。

可以通过`#patch`从`PatchedDataComponentMap`创建`DataComponentPatch`。同样，`PatchedDataComponentMap#fromPatch`可以根据原型`DataComponentMap`和`DataComponentPatch`构造`PatchedDataComponentMap`。

```java
public class ExampleHolder implements MutableDataComponentHolder {

    public static final Codec<ExampleHolder> CODEC = RecordCodecBuilder.create(instance ->
        instance.group(
            Codec.INT.fieldOf("data").forGetter(ExampleHolder::getData),
            DataCopmonentPatch.CODEC.optionalFieldOf("components", DataComponentPatch.EMPTY).forGetter(holder -> holder.components.asPatch())
        ).apply(instance, ExampleHolder::new)
    );

    public static final StreamCodec<RegistryFriendlyByteBuf, ExampleHolder> STREAM_CODEC = StreamCodec.composite(
        ByteBufCodecs.INT, ExampleHolder::getData,
        DataComponentPatch.STREAM_CODEC, holder -> holder.components.asPatch(),
        ExampleHolder::new
    );

    // ...

    public ExampleHolder(int data, DataComponentPatch patch) {
        this.data = data;
        this.components = PatchedDataComponentMap.fromPatch(
            // 应用的原型映射
            DataComponentMap.EMPTY,
            // 相关补丁
            patch
        );
    }

    // ...
}
```

[同步持有者数据到网络][network]以及将数据读写到磁盘必须手动完成。

[registered]: ../concepts/registries.md
[codec]: ../datastorage/codecs.md
[modbus]: ../concepts/events.md#event-buses
[network]: ../networking/payload.md
[streamcodec]: ../networking/streamcodecs.md
