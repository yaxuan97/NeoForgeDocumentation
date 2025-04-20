---
sidebar_position: 5
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 盔甲

盔甲是[物品][item]，其主要用途是通过各种抗性和效果保护[`生物实体`][livingentity]免受伤害。许多MOD添加了新的盔甲套装（例如铜盔甲）。

## 自定义盔甲套装

人形实体的盔甲套装通常由四件物品组成：头部的头盔，胸部的胸甲，腿部的护腿，以及脚部的靴子。还有用于狼、马和羊驼的盔甲，这些盔甲专门应用于动物的“身体”盔甲槽。所有这些物品通常分别通过`ArmorItem`和`AnimalArmorItem`实现。

盔甲几乎完全通过七个[数据组件][datacomponents]实现：

- `DataComponents#MAX_DAMAGE` 和 `#DAMAGE` 用于耐久度
- `#MAX_STACK_SIZE` 将堆叠大小设置为 `1`
- `#REPAIRABLE` 用于在铁砧中修复盔甲部件
- `#ENCHANTABLE` 用于最大[附魔][enchantment]值
- `#ATTRIBUTE_MODIFIERS` 用于盔甲、防御韧性和击退抗性
- `#EQUIPPABLE` 用于实体如何装备物品

`ArmorItem` 和 `AnimalArmorItem` 使用 `ArmorMaterial` 结合 `ArmorType` 或 `AnimalArmorItem.BodyType` 分别设置组件。参考值可以在 `ArmorMaterials` 中找到。以下示例使用铜盔甲材质，您可以根据需要调整这些值。

```java
public static final ArmorMaterial COPPER_ARMOR_MATERIAL = new ArmorMaterial(
    // 盔甲材质的耐久倍数。
    // ArmorType（盔甲类型）有不同的单位耐久值，倍数将应用于这些值：
    // - HELMET（头盔）: 11
    // - CHESTPLATE（胸甲）: 16
    // - LEGGINGS（护腿）: 15
    // - BOOTS（靴子）: 13
    // - BODY（身体）: 16
    15,
    // 决定防御值（或护甲栏上的半护甲数量）。
    // 基于 ArmorType（盔甲类型）。
    Util.make(new EnumMap<>(ArmorType.class), map -> {
        map.put(ArmorItem.Type.BOOTS, 2);
        map.put(ArmorItem.Type.LEGGINGS, 4);
        map.put(ArmorItem.Type.CHESTPLATE, 6);
        map.put(ArmorItem.Type.HELMET, 2);
        map.put(ArmorItem.Type.BODY, 4);
    }),
    // 决定盔甲的附魔能力。这表示该盔甲的附魔效果有多好。
    // 金的附魔能力为 25；我们将铜设置得稍低一些。
    20,
    // 决定装备此盔甲时播放的声音。
    // 这是用 Holder 包装的。
    SoundEvents.ARMOR_EQUIP_GENERIC,
    // 返回盔甲的韧性值。韧性值是伤害计算中包含的一个附加值，
    // 更多信息请参考 Minecraft Wiki 上关于盔甲机制的文章：
    // https://minecraft.wiki/w/Armor#Armor_toughness
    // 只有钻石和下界合金的值大于 0，因此我们这里返回 0。
    0,
    // 返回盔甲的击退抗性值。穿戴此盔甲时，玩家对击退有一定程度的免疫。
    // 如果玩家从所有盔甲部件中获得的总击退抗性值为 1 或更高，
    // 则他们将完全免疫击退。
    // 只有下界合金的值大于 0，因此我们这里返回 0。
    0,
    // 一个标签，决定哪些物品可以修复此盔甲。
    Tags.Items.INGOTS_COPPER,
    // EquipmentClientInfo JSON 的资源键，详见下文
    // 指向 assets/examplemod/equipment/copper.json
    ResourceKey.create(EquipmentAssets.ROOT_ID, ResourceLocation.fromNamespaceAndPath("examplemod", "copper"))
);
```

现在我们有了`ArmorMaterial`，可以用它来[注册][registering]盔甲。`ArmorItem` 接受表示物品可装备位置的 `ArmorMaterial` 和 `ArmorType`。另一方面，`AnimalArmorItem` 接受 `ArmorMaterial` 和 `AnimalArmorItem.BodyType`。它还可以选择性地接受装备声音以及是否应用耐久性和可修复组件。

```java
// ITEMS 是 DeferredRegister.Items 的实例
public static final DeferredItem<ArmorItem> COPPER_HELMET = ITEMS.registerItem(
    "copper_helmet",
    props -> new ArmorItem(
        // 使用的材质。
        COPPER_ARMOR_MATERIAL,
        // 创建的盔甲类型。
        ArmorType.HELMET,
        // 物品属性。
        props
    )
);

public static final DeferredItem<ArmorItem> COPPER_CHESTPLATE =
    ITEMS.registerItem("copper_chestplate", props -> new ArmorItem(...));
public static final DeferredItem<ArmorItem> COPPER_LEGGINGS =
    ITEMS.registerItem("copper_chestplate", props -> new ArmorItem(...));
public static final DeferredItem<ArmorItem> COPPER_BOOTS =
    ITEMS.registerItem("copper_chestplate", props -> new ArmorItem(...));

public static final DeferredItem<AnimalArmorItem> COPPER_WOLF_ARMOR = ITEMS.registerItem(
    "copper_wolf_armor",
    props -> new AnimalArmorItem(
        // 使用的材质。
        COPPER_ARMOR_MATERIAL,
        // 盔甲可穿戴的身体类型。
        AnimalArmorItem.BodyType.CANINE,
        // 物品属性。
        props
    )
);

public static final DeferredItem<AnimalArmorItem> COPPER_HORSE_ARMOR =
    ITEMS.registerItem("copper_horse_armor", props -> new AnimalArmorItem(...));
```

现在，创建盔甲或类似盔甲的物品不需要继承`ArmorItem`或`AnimalArmorItem`。它可以简单地通过以下部分的组合实现：

- 通过`Item.Properties#component`设置`DataComponents#EQUIPPABLE`，为您的物品添加`Equippable`，并设置您自己的要求。
- 通过`Item.Properties#attributes`为物品添加属性（例如盔甲、防御韧性、击退）。
- 通过`Item.Properties#durability`添加物品耐久性。
- 通过`Item.Properties#repariable`允许修复物品。
- 通过`Item.Properties#enchantable`允许物品附魔。
- 将您的盔甲添加到某些`minecraft:enchantable/*` `ItemTags`中，以便可以对您的物品应用某些附魔。

### `Equippable`

`Equippable` 是一个数据组件，包含实体如何装备此物品以及在游戏中如何渲染的内容。这允许任何物品，无论是否被视为“盔甲”，只要此组件可用，就可以装备（例如羊驼上的地毯）。每个具有此组件的物品只能装备到单个`EquipmentSlot`。

可以通过直接调用记录构造函数或通过`Equippable#builder`创建`Equippable`，后者为每个字段设置默认值，完成后调用`build`：

```java
// 假设有一些 DeferredRegister.Items ITEMS
public static final DeferredItem<Item> EQUIPPABLE = ITEMS.registerSimpleItem(
    "equippable",
    new Item.Properties().copmonent(
        DataComponents.EQUIPPABLE,
        // 设置此物品可以装备的槽位。
        Equippable.builder(EquipmentSlot.HELMET)
            // 决定装备此盔甲时播放的声音。
            // 这是用 Holder 包装的。
            // 默认为 SoundEvents#ARMOR_EQUIP_GENERIC。
            .setEquipSound(SoundEvents.ARMOR_EQUIP_GENERIC)
            // EquipmentClientInfo JSON 的资源键，详见下文。
            // 指向 assets/examplemod/equipment/equippable.json
            // 如果未设置，则不渲染装备。
            .setAsset(ResourceKey.create(EquipmentAssets.ROOT_ID, ResourceLocation.fromNamespaceAndPath("examplemod", "equippable")))
            // 穿戴时在玩家屏幕上叠加的纹理的相对位置（例如，南瓜模糊）。
            // 指向 assets/examplemod/textures/equippable.png
            // 如果未设置，则不渲染叠加。
            .setCameraOverlay(ResourceLocation.withDefaultNamespace("examplemod", "equippable"))
            // 可以装备此物品的实体类型的 HolderSet（直接或标签）。
            // 如果未设置，则任何实体都可以装备此物品。
            .setAllowedEntities(EntityType.ZOMBIE)
            // 是否可以通过发射器装备此物品。
            // 默认为 true。
            .setDispensable(true),
            // 是否可以在快速装备期间从玩家身上交换此物品。
            // 默认为 true。
            .setSwappable(false),
            // 是否在被攻击时损坏物品（通常用于装备）。
            // 还必须是可损坏的物品。
            // 默认为 true。
            .setDamageOnHurt(false)
            .build()
    )
);
```

## 装备资源

现在我们在游戏中有了一些盔甲，但如果我们尝试穿戴它，由于我们从未指定如何渲染装备，因此不会渲染任何内容。为此，我们需要在`Equippable#assetId`指定的位置创建一个`EquipmentClientInfo` JSON，相对于[资源包][respack]的`equipment`文件夹（`assets`文件夹）。`EquipmentClientInfo`指定要使用的每个层的相关纹理以进行渲染。

`EquipmentClientInfo`在功能上是`EquipmentClientInfo.LayerType`到要应用的`EquipmentClientInfo.Layer`列表的映射。

`LayerType`可以被认为是某些实例的纹理组。例如，`LayerType#HUMANOID`由`HumanoidArmorLayer`用于渲染人形实体的头部、胸部和脚部；`LayerType#WOLF_BODY`由`WolfArmorLayer`用于渲染身体盔甲。这些可以组合到一个装备信息 JSON 中，如果它们是同一类型的可装备物品，例如铜盔甲。

`LayerType`映射到要应用和按提供顺序渲染的`Layer`列表。`Layer`实际上表示要渲染的单个纹理。第一个参数表示纹理的位置，相对于`textures/entity/equipment`。

第二个参数是一个可选项，指示纹理是否可以作为`EquipmentClientInfo.Dyeable`进行[染色][tinting]。`Dyeable`对象包含一个整数，当存在时，指示默认的 RGB 颜色以对纹理进行染色。如果此可选项不存在，则使用纯白色。

第三个参数是一个布尔值，指示在渲染期间是否应使用提供的纹理而不是`Layer`中定义的纹理。一个示例是玩家的自定义披风或自定义鞘翅纹理。

让我们为铜盔甲材质创建一个装备信息。我们还假设每个层都有两个纹理：一个用于实际盔甲，一个用于叠加和染色。对于动物盔甲，我们会说有一些可以传入的动态纹理。

<Tabs>
<TabItem value="json" label="JSON" default>

```json5
// 在 assets/examplemod/equipment/copper.json
{
    // 层映射
    "layers": {
        // 要应用的 EquipmentClientInfo.LayerType 的序列化名称。
        // 对于人形头部、胸部和脚部
        "humanoid": [
            // 按提供顺序渲染的层列表
            {
                // 盔甲的相对纹理
                // 指向 assets/examplemod/textures/entity/equipment/copper/outer.png
                "texture": "examplemod:copper/outer"
            },
            {
                // 叠加纹理
                // 指向 assets/examplemod/textures/entity/equipment/copper/outer_overlay.png
                "texture": "examplemod:copper/outer_overlay",
                // 如果指定，允许纹理染色为 DataComponents#DYED_COLOR 中的颜色
                // 否则，无法染色
                "dyeable": {
                    // RGB 值（始终为不透明颜色）
                    // 0x7683DE 转为十进制
                    // 如果未指定，则设置为 0（表示透明或不可见）
                    "color_when_undyed": 7767006
                }
            }
        ],
        // 对于人形护腿
        "humanoid_leggings": [
            {
                // 指向 assets/examplemod/textures/entity/equipment/copper/inner.png
                "texture": "examplemod:copper/inner"
            },
            {
                // 指向 assets/examplemod/textures/entity/equipment/copper/inner_overlay.png
                "texture": "examplemod:copper/inner_overlay",
                "dyeable": {
                    "color_when_undyed": 7767006
                }
            }
        ],
        // 对于狼盔甲
        "wolf_body": [
            {
                // 指向 assets/examplemod/textures/entity/equipment/copper/wolf.png
                "texture": "examplemod:copper/wolf",
                // 如果为 true，则改用传递到层渲染器的纹理
                "use_player_texture": true
            }
        ],
        // 对于马盔甲
        "horse_body": [
            {
                // 指向 assets/examplemod/textures/entity/equipment/copper/horse.png
                "texture": "examplemod:copper/horse",
                "use_player_texture": true
            }
        ]
    }
}
```

</TabItem>

<TabItem value="datagen" label="Datagen">

```java
public class MyEquipmentInfoProvider implements DataProvider {

    private final PackOutput.PathProvider path;

    public MyEquipmentInfoProvider(PackOutput output) {
        this.path = output.createPathProvider(PackOutput.Target.RESOURCE_PACK, "equipment");
    }

    private void add(BiConsumer<ResourceLocation, EquipmentClientInfo> registrar) {
        registrar.accept(
            // 必须与 Equippable#assetId 匹配
            ResourceLocation.fromNamespaceAndPath("examplemod", "copper"),
            EquipmentClientInfo.builder()
                // 对于人形头部、胸部和脚部
                .addLayers(
                    EquipmentClientInfo.LayerType.HUMANOID,
                    // 基础纹理
                    new EquipmentClientInfo.Layer(
                        // 盔甲的相对纹理
                        // 指向 assets/examplemod/textures/entity/equipment/copper/outer.png
                        ResourceLocation.fromNamespaceAndPath("examplemod", "copper/outer"),
                        Optional.empty(),
                        false
                    ),
                    // 叠加纹理
                    new EquipmentClientInfo.Layer(
                        // 叠加纹理
                        // 指向 assets/examplemod/textures/entity/equipment/copper/outer_overlay.png
                        ResourceLocation.fromNamespaceAndPath("examplemod", "copper/outer_overlay"),
                        // RGB 值（始终为不透明颜色）
                        // 如果未指定，则设置为 0（表示透明或不可见）
                        Optional.of(new EquipmentClientInfo.Dyeable(Optional.of(0x7683DE))),
                        false
                    )
                )
                // 对于人形护腿
                .addLayers(
                    EquipmentClientInfo.LayerType.HUMANOID_LEGGINGS,
                    new EquipmentClientInfo.Layer(
                        // 指向 assets/examplemod/textures/entity/equipment/copper/inner.png
                        ResourceLocation.fromNamespaceAndPath("examplemod", "copper/inner"),
                        Optional.empty(),
                        false
                    ),
                    new EquipmentClientInfo.Layer(
                        // 指向 assets/examplemod/textures/entity/equipment/copper/inner_overlay.png
                        ResourceLocation.fromNamespaceAndPath("examplemod", "copper/inner_overlay"),
                        Optional.of(new EquipmentClientInfo.Dyeable(Optional.of(0x7683DE))),
                        false
                    )
                )
                // 对于狼盔甲
                .addLayers(
                    EquipmentClientInfo.LayerType.WOLF_BODY,
                    // 基础纹理
                    new EquipmentClientInfo.Layer(
                        // 指向 assets/examplemod/textures/entity/equipment/copper/wolf.png
                        ResourceLocation.fromNamespaceAndPath("examplemod", "copper/wolf"),
                        Optional.empty(),
                        // 如果为 true，则改用传递到层渲染器的纹理
                        true
                    )
                )
                // 对于马盔甲
                .addLayers(
                    EquipmentClientInfo.LayerType.HORSE_BODY,
                    // 基础纹理
                    new EquipmentClientInfo.Layer(
                        // 指向 assets/examplemod/textures/entity/equipment/copper/horse.png
                        ResourceLocation.fromNamespaceAndPath("examplemod", "copper/horse"),
                        Optional.empty(),
                        true
                    )
                )
                .build()
        );
    }

    @Override
    public CompletableFuture<?> run(CachedOutput cache) {
        Map<ResourceLocation, EquipmentClientInfo> map = new HashMap<>();
        this.add((name, info) -> {
            if (map.putIfAbsent(name, info) != null) {
                throw new IllegalStateException("尝试为 ID 注册两次装备客户端信息：" + name);
            }
        });
        return DataProvider.saveAll(cache, EquipmentClientInfo.CODEC, this.pathProvider, map);
    }

    @Override
    public String getName() {
        return "装备客户端信息：" + MOD_ID;
    }
}

// 监听 mod 事件总线
@SubscribeEvent
public static void gatherData(GatherDataEvent.Client event) {
    event.createProvider(MyEquipmentInfoProvider::new);
}
```

</TabItem>
</Tabs>

## 装备渲染

装备信息通过`EquipmentLayerRenderer`在`EntityRenderer`或其`RenderLayer`之一的渲染函数中渲染。`EquipmentLayerRenderer`作为渲染上下文的一部分通过`EntityRendererProvider.Context#getEquipmentRenderer`获取。如果需要`EquipmentClientInfo`，也可以通过`EntityRendererProvider.Context#getEquipmentAssets`获取。

默认情况下，以下层渲染相关的`EquipmentClientInfo.LayerType`：

| `LayerType`         | `RenderLayer`        | 使用者                                                        |
|:-------------------:|:--------------------:|:---------------------------------------------------------------|
| `HUMANOID`          | `HumanoidArmorLayer` | 玩家、人形生物（例如僵尸、骷髅）、盔甲架                     |
| `HUMANOID_LEGGINGS` | `HumanoidArmorLayer` | 玩家、人形生物（例如僵尸、骷髅）、盔甲架                     |
| `WINGS`             | `WingsLayer`         | 玩家、人形生物（例如僵尸、骷髅）、盔甲架                     |
| `WOLF_BODY`         | `WolfArmorLayer`     | 狼                                                           |
| `HORSE_BODY`        | `HorseArmorLayer`    | 马                                                           |
| `LLAMA_BODY`        | `LlamaDecorLayer`    | 羊驼、商人羊驼                                               |

`EquipmentLayerRenderer`只有一个方法来渲染装备层，恰当地命名为`renderLayers`：

```java
// 在某个渲染方法中，其中 EquipmentLayerRenderer equipmentLayerRenderer 是一个字段
this.equipmentLayerRenderer.renderLayers(
    // 要渲染的层类型
    EquipmentClientInfo.LayerType.HUMANOID,
    // 表示 EquipmentClientInfo JSON 的资源键
    // 这将在 EQUIPPABLE 数据组件中通过 assetId 设置
    stack.get(DataComponents.EQUIPPABLE).assetId().orElseThrow(),
    // 要应用装备信息的模型
    // 这些通常是与实体模型分开的模型
    // 并且是链接到 LayerDefinition 的单独 ModelLayers
    model,
    // 表示作为模型渲染的物品的物品堆
    // 这仅用于获取可染色、闪烁和盔甲装饰信息
    stack,
    // 用于在正确位置渲染模型的姿态堆
    poseStack,
    // 用于获取渲染类型的顶点消费者的缓冲区源
    bufferSource,
    // 打包的光照纹理
    lighting,
    // 如果不为 null，则表示在 use_player_texture 为 true 时渲染的纹理的绝对路径
    // 表示 assets 文件夹中的绝对位置
    ResourceLocation.fromNamespaceAndPath("examplemod", "textures/other_texture.png")
);
```

[item]: index.md
[datacomponents]: datacomponents.md
[enchantment]: ../resources/server/enchantments/index.md#enchantment-costs-and-levels
[livingentity]: ../entities/livingentity.md
[registering]: ../concepts/registries.md#methods-for-registering
[rendering]: #装备渲染
[respack]: ../resources/index.md#assets
[tag]: ../resources/server/tags.md
[tinting]: ../resources/client/models/index.md#tinting
