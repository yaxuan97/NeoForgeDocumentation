---
sidebar_position: 3
---
# 消耗品

消耗品是[物品][item]，可以在一段时间内使用，并在此过程中“消耗”它们。在Minecraft中，任何可以吃或喝的东西都是某种类型的消耗品。

## `Consumable` 数据组件

任何可以被消耗的物品都具有[`DataComponents#CONSUMABLE`组件][datacomponent]。支持的记录`Consumable`定义了物品如何被消耗以及消耗后应用的效果。

可以通过直接调用记录构造函数或通过`Consumable#builder`创建`Consumable`，后者为每个字段设置默认值，完成后调用`build`：

- `consumeSeconds` - 一个`float`值，表示完全消耗物品所需的秒数。当经过指定时间后调用`Item#finishUsingItem`。默认为1.6秒或32刻（ticks）。
- `animation` - 设置物品使用时播放的[`ItemUseAnimation`][animation]。默认为`ItemUseAnimation#EAT`。
- `sound` - 设置消耗物品时播放的[`SoundEvent`][sound]。必须是一个`Holder`实例。默认为`SoundEvents#GENERIC_EAT`。
    - 如果原版实例不是`Holder<SoundEvent>`，可以通过调用`BuiltInRegistries.SOUND_EVENT.wrapAsHolder(soundEvent)`获取一个`Holder`包装版本。
- `soundAfterConsume` - 设置物品完全消耗后播放的[`SoundEvent`][sound]。委托给[`PlaySoundConsumeEffect`][consumeeffect]。
- `hasConsumeParticles` - 当为`true`时，每四刻生成一次物品[粒子][particles]，并在物品完全消耗后生成一次。默认为`true`。
- `onConsume` - 添加一个[`ConsumeEffect`][consumeeffect]，在物品通过`Item#finishUsingItem`完全消耗后应用。

原版在其`Consumables`类中提供了一些消耗品，例如用于[食物][food]物品的`#defaultFood`和用于[药水][potions]及牛奶桶的`#defaultDrink`。

可以通过调用`Item.Properties#component`添加`Consumable`组件：

```java
// 假设有一个 DeferredRegister.Items ITEMS
public static final DeferredItem<Item> CONSUMABLE = ITEMS.registerSimpleItem(
    "consumable",
    new Item.Properties().component(
        DataComponents.CONSUMABLE,
        Consumable.builder()
            // 消耗2秒（40刻）
            .consumeSeconds(2f)
            // 设置消耗时播放的动画
            .animation(ItemUseAnimation.BLOCK)
            // 每刻播放消耗时的声音
            .sound(SoundEvents.ARMOR_EQUIP_CHAIN)
            // 消耗完成后播放的声音
            .soundAfterConsume(SoundEvents.BREEZE_WIND_CHARGE_BURST)
            // 不显示消耗时的粒子效果
            .hasConsumeParticles(false)
            .onConsume(
                // 消耗完成后，以30%的概率应用效果
                new ApplyStatusEffectsConsumeEffect(new MobEffectInstance(MobEffects.HUNGER, 600, 0), 0.3F)
            )
            // 可以有多个效果
            .onConsume(
                // 随机将实体传送到50格范围内
                new TeleportRandomlyConsumeEffect(100f)
            )
            .build()
    )
);
```

### `ConsumeEffect`

当消耗品使用完成后，您可能希望触发某种逻辑执行，例如添加药水效果。这些由`ConsumeEffect`处理，可以通过调用`Consumable.Builder#onConsume`添加到`Consumable`中。

原版效果列表可以在`ConsumeEffect`中找到。

每个`ConsumeEffect`都有两个方法：`getType`，指定注册对象`ConsumeEffect.Type`；以及`apply`，在物品完全消耗时调用。`apply`接受三个参数：消耗实体所在的`Level`，调用消耗品的`ItemStack`，以及消耗对象的`LivingEntity`。当效果成功应用时，方法返回`true`，如果失败则返回`false`。

可以通过实现接口并将关联的`MapCodec`和`StreamCodec`的`ConsumeEffect.Type`[注册]到`BuiltInRegistries#CONSUME_EFFECT_TYPE`来创建`ConsumeEffect`：

```java
public record UsePortalConsumeEffect(ResourceKey<Level> level)
    implements ConsumeEffect, Portal {

    @Override
    public boolean apply(Level level, ItemStack stack, LivingEntity entity) {
        if (entity.canUsePortal(false)) {
            entity.setAsInsidePortal(this, entity.blockPosition());

            // 成功使用传送门
            return true;
        }

        // 无法使用传送门
        return false;
    }

    @Override
    public ConsumeEffect.Type<? extends ConsumeEffect> getType() {
        // 设置为注册对象
        return USE_PORTAL.get();
    }

    @Override
    @Nullable
    public TeleportTransition getPortalDestination(ServerLevel level, Entity entity, BlockPos pos) {
        // 设置传送位置
    }
}

// 在某个注册类中
// 假设有一个 DeferredRegister<ConsumeEffect.Type<?>> CONSUME_EFFECT_TYPES
public static final Supplier<ConsumeEffect.Type<UsePortalConsumeEffect>> USE_PORTAL =
    CONSUME_EFFECT_TYPES.register("use_portal", () -> new ConsumeEffect.Type<>(
        ResourceKey.codec(Registries.DIMENSION).optionalFieldOf("dimension")
            .xmap(UsePortalConsumeEffect::new, UsePortalConsumeEffect::level),
        ResourceKey.streamCodec(Registries.DIMENSION)
            .map(UsePortalConsumeEffect::new, UsePortalConsumeEffect::level)
    ));

// 对于某些添加了CONSUMABLE组件的Item.Properties
Consumable.builder()
    .onConsume(
        new UsePortalConsumeEffect(Level.END)
    )
    .build();
```

### `ItemUseAnimation`

`ItemUseAnimation`实际上是一个枚举，它除了定义其id和名称外没有其他内容。它的用途在`ItemHandRenderer#renderArmWithItem`中硬编码用于第一人称，在`PlayerRenderer#getArmPose`中用于第三人称。因此，简单地创建一个新的`ItemUseAnimation`将仅类似于`ItemUseAnimation#NONE`。

要应用某些动画，您需要为第一人称实现`IClientItemExtensions#applyForgeHandTransform`，为第三人称实现`IClientItemExtensions#getArmPose`。

#### 创建`ItemUseAnimation`

首先，让我们创建一个新的`ItemUseAnimation`。这是使用[可扩展枚举][extensibleenum]系统完成的：

```json5
{
    "entries": [
        {
            "enum": "net/minecraft/world/item/ItemUseAnimation",
            "name": "EXAMPLEMOD_ITEM_USE_ANIMATION",
            "constructor": "(ILjava/lang/String;)V",
            "parameters": [
                // id，应该始终为-1
                -1,
                // 名称，应该是唯一标识符
                "examplemod:item_use_animation"
            ]
        }
    ]
}
```

然后我们可以通过`valueOf`获取枚举常量：

```java
public static final ItemUseAnimation EXAMPLE_ANIMATION = ItemUseAnimation.valueOf("EXAMPLEMOD_ITEM_USE_ANIMATION");
```

从那里，我们可以开始应用变换。为此，我们必须创建一个新的`IClientItemExtensions`，实现我们所需的方法，并通过[**mod事件总线**][modbus]上的`RegisterClientExtensionsEvent`注册它：

```java
public class ConsumableClientItemExtensions implements IClientItemExtensions {
    // 在此实现方法
}

// 在某个事件监听类中
@SubscribeEvent
public static void registerClientExtensions(RegisterClientExtensionsEvent event) {
    event.registerItem(
        // 扩展实例
        new ConsumableClientItemExtensions(),
        // 使用此扩展的物品
        CONSUMABLE
    )
}
```

#### 第一人称

所有消耗品都有的第一人称变换通过`IClientItemExtensions#applyForgeHandTransform`实现：

```java
public class ConsumableClientItemExtensions implements IClientItemExtensions {

    // ...

    @Override
    public boolean applyForgeHandTransform(
        PoseStack poseStack, LocalPlayer player, HumanoidArm arm, ItemStack itemInHand,
        float partialTick, float equipProcess, float swingProcess
    ) {
        // 首先需要检查物品是否正在使用并具有我们的动画
        HumanoidArm usingArm = entity.getUsedItemHand() == InteractionHand.MAIN_HAND
            ? entity.getMainArm()
            : entity.getMainArm().getOpposite();
        if (
            entity.isUsingItem() && entity.getUseItemRemainingTicks() > 0
            && usingArm == arm && itemInHand.getUseAnimation() == EXAMPLE_ANIMATION
        ) {
            // 对姿势堆栈应用变换（平移、缩放、旋转）
            // ...
            return true;
        }

        // 不执行任何操作
        return false;
    }
}
```

#### 第三人称

所有但`EAT`和`DRINK`有特殊逻辑的第三人称变换通过`IClientItemExtensions#getArmPose`实现，其中`HumanoidModel.ArmPose`也可以扩展以实现自定义变换。

由于`ArmPose`需要一个lambda作为其构造函数的一部分，因此必须使用`EnumProxy`引用：

```json5
{
    "entries": [
        {
            "name": "EXAMPLEMOD_ITEM_USE_ANIMATION",
            // ...
        },
        {
            "enum": "net/minecraft/client/model/HumanoidModel$ArmPose",
            "name": "EXAMPLEMOD_ARM_POSE",
            "constructor": "(ZLnet/neoforged/neoforge/client/IArmPoseTransformer;)V",
            "parameters": {
                // 指向代理所在的类
                // 应该是单独的，因为这是一个仅客户端的类
                "class": "example/examplemod/client/MyClientEnumParams",
                // 枚举代理的字段名称
                "field": "CUSTOM_ARM_POSE"
            }
        }
    ]
}
```

```java
// 创建枚举参数
public class MyClientEnumParams {
    public static final EnumProxy<HumanoidModel.ArmPose> CUSTOM_ARM_POSE = new EnumProxy<>(
        HumanoidModel.ArmPose.class,
        // 姿势是否使用双臂
        false,
        // 姿势变换器
        (IArmPoseTransformer) MyClientEnumParams::applyCustomModelPose
    );

    private static void applyCustomModelPose(
        HumanoidModel<?> model, HumanoidRenderState state, HumanoidArm arm
    ) {
        // 在此应用模型变换
        // ...
    }
}

// 在某个仅客户端的类中
public static final HumanoidModel.ArmPose EXAMPLE_POSE = HumanoidModel.ArmPose.valueOf("EXAMPLEMOD_ARM_POSE");
```

然后，通过`IClientItemExtensions#getArmPose`设置手臂姿势：

```java
public class ConsumableClientItemExtensions implements IClientItemExtensions {

    // ...

    @Override
    public HumanoidModel.ArmPose getArmPose(
        LivingEntity entity, InteractionHand hand, ItemStack stack
    ) {
        // 首先需要检查物品是否正在使用并具有我们的动画
        if (
            entity.isUsingItem() && entity.getUseItemRemainingTicks() > 0
            && entity.getUsedItemHand() == hand
            && itemInHand.getUseAnimation() == EXAMPLE_ANIMATION
        ) {
            // 返回要应用的姿势
            return EXAMPLE_POSE;
        }

        // 否则返回null
        return null;
    }
}
```

### 覆盖实体的声音

有时，实体可能希望在消耗物品时播放不同的声音。在这些情况下，[`LivingEntity`][livingentity]实例可以实现`Consumable.OverrideConsumeSound`并让`getConsumeSound`返回他们希望实体播放的`SoundEvent`。

```java
public class MyEntity extends LivingEntity implements Consumable.OverrideConsumeSound {
    
    // ...

    @Override
    public SoundEvent getConsumeSound(ItemStack stack) {
        // 返回要播放的声音
    }
}
```

## `ConsumableListener`

虽然消耗品和消耗后应用的效果很有用，但有时效果的属性需要作为其他[数据组件][datacomponents]外部可用。例如，猫和狼也吃[食物][food]并查询其营养，或者带有药水内容的物品查询其颜色以进行渲染。在这些情况下，数据组件实现`ConsumableListener`以提供消耗逻辑。

`ConsumableListener`只有一个方法：`#onConsume`，它接受当前的等级、消耗物品的实体、被消耗的物品以及物品上的`Consumable`实例。当物品完全消耗时，在`Item#finishUsingItem`期间调用`onConsume`。

添加您自己的`ConsumableListener`只是[注册一个新的数据组件][datacompreg]并实现`ConsumableListener`。

```java
public record MyConsumableListener() implements ConsumableListener {

    @Override
    public void onConsume(
        Level level, LivingEntity entity, ItemStack stack, Consumable consumable
    ) {
        // 在此执行操作
    }
}
```

### 食物

食物是饥饿系统的一种`ConsumableListener`类型。所有食物物品的功能都已在`Item`类中处理，因此只需将`FoodProperties`添加到`DataComponents#FOOD`以及一个消耗品即可。如果未指定消耗品，可以使用`Consumables#DEFAULT_FOOD`。

可以通过直接调用记录构造函数或通过`new FoodProperties.Builder()`创建`FoodProperties`，完成后调用`build`：

- `nutrition` - 设置恢复的饥饿点数。以半饥饿点计数，例如，Minecraft的牛排恢复8个饥饿点。
- `saturationModifier` - 饱和度修正值，用于计算食用此食物时恢复的[饱和值][hunger]。计算公式为`min(2 * nutrition * saturationModifier, playerNutrition)`，这意味着使用`0.5`将使有效饱和值与营养值相同。
- `alwaysEdible` - 此物品是否可以始终食用，即使饥饿条已满。默认为`false`，对于金苹果和其他提供超出填充饥饿条的奖励的物品为`true`。

```java
// 假设有一个 DeferredRegister.Items ITEMS
public static final DeferredItem<Item> FOOD = ITEMS.registerSimpleItem(
    "food",
    new Item.Properties().food(
        new FoodProperties.Builder()
            // 恢复1.5颗心
            .nutrition(3)
            // 胡萝卜是0.3
            // 生鳕鱼是0.1
            // 熟鸡肉是0.6
            // 熟牛肉是0.8
            // 金苹果是1.2
            .saturationModifier(0.3f)
            // 设置后，即使饥饿条已满，食物也可以始终食用。
            .alwaysEdible()
    )
);
```

有关示例或查看Minecraft使用的各种值，请查看`Foods`类。

要获取物品的`FoodProperties`，请调用`ItemStack.get(DataComponents.FOOD)`。这可能返回null，因为并非每个物品都是可食用的。要确定物品是否可食用，请对`getFoodProperties`调用的结果进行空检查。

### 药水内容

通过`PotionContents`的[药水][potions]内容是另一个`ConsumableListener`，其效果在消耗时应用。它们包含一个可选的药水以应用，一个可选的药水颜色，一个自定义[`MobEffectInstance`][mobeffectinstance]列表以与药水一起应用，以及一个可选的翻译键以在获取堆栈名称时使用。如果不是`PotionItem`的子类型，模组开发者需要覆盖`Item#getName`。

[animation]: #itemuseanimation
[consumeeffect]: #consumeeffect
[datacomponent]: datacomponents.md
[datacompreg]: datacomponents.md#creating-custom-data-components
[extensibleenum]: ../advanced/extensibleenums.md
[food]: #食物
[hunger]: https://minecraft.wiki/w/Hunger#Mechanics
[item]: index.md
[livingentity]: ../entities/livingentity.md
[modbus]: ../concepts/events.md#event-buses
[mobeffectinstance]: mobeffects.md#mobeffectinstances
[particles]: ../resources/client/particles.md
[potions]: mobeffects.md#potions
[sound]: ../resources/client/sounds.md#creating-soundevents
[registering]: ../concepts/registries.md#注册方法
