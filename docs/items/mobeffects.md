---
sidebar_position: 6
---
# 生物效果与药水

状态效果，有时称为药水效果，在代码中被称为`MobEffect`，是每刻（tick）影响[`生物实体`][livingentity]的效果。本文解释了如何使用它们、效果与药水之间的区别，以及如何添加自定义效果。

## 术语

- `MobEffect` 每刻影响一个实体。像[方块][block]或[物品][item]一样，`MobEffect`是注册对象，这意味着它们必须被[注册][registration]并且是单例。
    - **即时生物效果**是一种特殊的生物效果，设计为仅应用于一个刻。原版有两个即时效果：即时治疗和即时伤害。
- `MobEffectInstance` 是 `MobEffect` 的一个实例，具有持续时间、放大器和一些其他属性（见下文）。`MobEffectInstance` 对于 `MobEffect` 就像[`物品堆`][itemstack] 对于 `Item`。
- `Potion` 是 `MobEffectInstance` 的集合。原版主要将药水用于四种药水物品（继续阅读），然而，它们可以随意应用于任何物品。物品是否以及如何使用设置在其上的药水取决于物品本身。
- **药水物品**是指旨在设置药水的物品。这是一个非正式术语，原版的 `PotionItem` 类与此无关（它指的是“普通”药水物品）。Minecraft 目前有四种药水物品：药水、喷溅药水、滞留药水和附魔箭；然而，MOD可能会添加更多。

## `MobEffect`

要创建自定义的 `MobEffect`，扩展 `MobEffect` 类：

```java
public class MyMobEffect extends MobEffect {
    public MyMobEffect(MobEffectCategory category, int color) {
        super(category, color);
    }
    
    @Override
    public boolean applyEffectTick(ServerLevel level, LivingEntity entity, int amplifier) {
        // 在这里应用你的效果逻辑。

        // 如果当 shouldApplyEffectTickThisTick 返回 true 时此方法返回 false，效果将立即被移除。
        return true;
    }
    
    // 效果是否应该在此刻应用。例如，恢复效果根据刻计数和放大器每 x 刻应用一次。
    @Override
    public boolean shouldApplyEffectTickThisTick(int tickCount, int amplifier) {
        return tickCount % 2 == 0; // 用你想要的检查替换此处
    }
    
    // 当效果首次添加到实体时调用的实用方法。
    // 在此效果的所有实例从实体中移除之前，不会再次调用此方法。
    @Override
    public void onEffectAdded(LivingEntity entity, int amplifier) {
        super.onEffectAdded(entity, amplifier);
    }

    // 当效果添加到实体时调用的实用方法。
    // 每次将此效果添加到实体时都会调用此方法。
    @Override
    public void onEffectStarted(LivingEntity entity, int amplifier) {
    }
}
```

像所有注册对象一样，`MobEffect` 必须被[注册][registration]，如下所示：

```java
// MOB_EFFECTS 是 DeferredRegister<MobEffect> 的实例
public static final Holder<MobEffect> MY_MOB_EFFECT = MOB_EFFECTS.register("my_mob_effect", () -> new MyMobEffect(
        // 可以是 BENEFICIAL, NEUTRAL 或 HARMFUL。用于确定此效果的药水提示颜色。
        MobEffectCategory.BENEFICIAL,
        // 效果粒子的颜色，RGB 格式。
        0xffffff
));
```

`MobEffect` 类还为受影响的实体添加[属性修饰符][attributemodifier]提供了默认功能，并在效果到期或通过其他方式移除时将其移除。例如，速度效果为移动速度添加了一个属性修饰符。效果属性修饰符的添加方式如下：

```java
public static final Holder<MobEffect> MY_MOB_EFFECT = MOB_EFFECTS.register("my_mob_effect", () -> new MyMobEffect(...)
        .addAttributeModifier(Attributes.ATTACK_DAMAGE, ResourceLocation.fromNamespaceAndPath("examplemod", "effect.strength"), 2.0, AttributeModifier.Operation.ADD_VALUE)
);
```

### `InstantenousMobEffect`

如果你想创建一个即时效果，可以使用辅助类 `InstantenousMobEffect` 而不是常规的 `MobEffect` 类，如下所示：

```java
public class MyMobEffect extends InstantenousMobEffect {
    public MyMobEffect(MobEffectCategory category, int color) {
        super(category, color);
    }

    @Override
    public void applyEffectTick(ServerLevel level, LivingEntity entity, int amplifier) {
        // 在这里应用你的效果逻辑。
    }
}
```

然后，像正常一样[注册][registration]你的效果。

### 事件

许多效果的逻辑在其他地方应用。例如，漂浮效果在生物实体移动处理器中应用。对于模组的 `MobEffect`，通常在[事件处理器][events]中应用它们是有意义的。NeoForge 还提供了一些与效果相关的事件：

- `MobEffectEvent.Applicable` 在游戏检查 `MobEffectInstance` 是否可以应用于实体时触发。此事件可用于拒绝或强制将效果实例添加到目标。
- `MobEffectEvent.Added` 在 `MobEffectInstance` 添加到目标时触发。此事件包含有关可能已存在于目标上的先前 `MobEffectInstance` 的信息。
- `MobEffectEvent.Expired` 在 `MobEffectInstance` 到期时触发，即计时器归零。
- `MobEffectEvent.Remove` 在效果通过非到期方式从实体中移除时触发，例如通过喝牛奶或通过命令。

## `MobEffectInstance`

`MobEffectInstance` 简单来说，就是应用于实体的效果。通过调用构造函数创建 `MobEffectInstance`：

```java
MobEffectInstance instance = new MobEffectInstance(
        // 要使用的生物效果。
        MobEffects.REGENERATION,
        // 要使用的持续时间，以刻为单位。如果未指定，默认为 0。
        500,
        // 要使用的放大器。这是效果的“强度”，即力量 I，力量 II 等。
        // 必须在 0 到 255（含）之间。如果未指定，默认为 0。
        0,
        // 效果是否为“环境”效果，意味着它是由环境源应用的，
        // Minecraft 目前有信标和潮涌核心。如果未指定，默认为 false。
        false,
        // 效果是否在物品栏中可见。如果未指定，默认为 true。
        true,
        // 效果图标是否在右上角可见。如果未指定，默认为 true。
        true
);
```

有几个构造函数重载，分别省略最后 1-5 个参数。

:::info
`MobEffectInstance` 是可变的。如果需要副本，请调用 `new MobEffectInstance(oldInstance)`。
:::

### 使用 `MobEffectInstance`

可以像这样将 `MobEffectInstance` 添加到 `LivingEntity`：

```java
MobEffectInstance instance = new MobEffectInstance(...);
livingEntity.addEffect(instance);
```

同样，也可以从 `LivingEntity` 中移除 `MobEffectInstance`。由于 `MobEffectInstance` 会覆盖实体上同一 `MobEffect` 的预先存在的 `MobEffectInstance`，因此每个 `MobEffect` 和实体只能有一个 `MobEffectInstance`。因此，移除时只需指定 `MobEffect` 即可：

```java
livingEntity.removeEffect(MobEffects.REGENERATION);
```

:::info
`MobEffect` 只能应用于 `LivingEntity` 或其子类，即玩家和生物。物品或投掷的雪球等不能受到 `MobEffect` 的影响。
:::

## `Potion`

通过调用 `Potion` 的构造函数并传入你希望药水具有的 `MobEffectInstance` 来创建 `Potion`。例如：

```java
// POTIONS 是 DeferredRegister<Potion> 的实例
public static final Holder<Potion> MY_POTION = POTIONS.register("my_potion", registryName -> new Potion(
    // 应用于药水的后缀
    registryName.getPath(),
    // 药水使用的效果
    new MobEffectInstance(MY_MOB_EFFECT, 3600)
));
```

药水的名称是第一个构造函数参数。它用作翻译键的后缀；例如，原版中的长效和强效药水变体使用此名称与其基础变体具有相同的名称。

`new Potion` 的 `MobEffectInstance` 参数是一个可变参数。这意味着你可以为药水添加任意多的效果。这也意味着可以创建空药水，即没有任何效果的药水。只需调用 `new Potion()` 即可！（顺便说一句，这就是原版添加 `awkward` 药水的方式。）

`PotionContents` 类提供了与药水物品相关的各种辅助方法。药水物品通过 `DataComponent#POTION_CONTENTS` 存储其 `PotionContents`。

### 酿造

现在你的药水已经添加，药水物品已经可用。然而，在生存模式中没有办法获得你的药水，所以让我们改变这一点！

传统上，药水是在酿造台中制作的。不幸的是，Mojang 不提供[数据包][datapack]支持的酿造配方，因此我们必须通过 `RegisterBrewingRecipesEvent` 事件以稍显传统的方式通过代码添加配方。如下所示：

```java
// 使用某种方法监听事件
@SubscribeEvent
public static void registerBrewingRecipes(RegisterBrewingRecipesEvent event) {
    // 获取用于添加配方的构建器
    PotionBrewing.Builder builder = event.getBuilder();

    // 将为所有容器药水（例如药水、喷溅药水、滞留药水）添加酿造配方
    builder.addMix(
        // 要应用的初始药水
        Potions.AWKWARD,
        // 酿造成分。这是酿造台顶部的物品。
        Items.FEATHER,
        // 结果药水
        MY_POTION
    );
}
```

[attributemodifier]: ../entities/attributes.md#attribute-modifiers
[block]: ../blocks/index.md
[commonsetup]: ../concepts/events.md#事件总线
[datapack]: ../resources/index.md#data
[events]: ../concepts/events.md
[item]: index.md
[itemstack]: index.md#itemstacks
[livingentity]: ../entities/livingentity.md
[registration]: ../concepts/registries.md#methods-for-registering
[uuidgen]: https://www.uuidgenerator.net/version4
