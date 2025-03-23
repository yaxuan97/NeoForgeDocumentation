# Mod 配置文件

MOD 配置文件负责决定如何将 MOD 打包到您的 JAR 中，在 "MOD" 菜单中显示哪些信息，以及如何在游戏中加载您的 MOD。

## `gradle.properties`

`gradle.properties` 文件保存了 mod 的各种常用属性，比如 mod id 或 mod 版本在构建过程中，Gradle 会读取这些文件中的值，并将它们内嵌到不同的地方，比如 [neoforge.mods.toml][neoforgemodstoml] 文件这样，你只需要在这个地方修改它，然后它们就会被应用到其他地方。

大多数值在 [MDK 的 `gradle.properties` 文件][mdkgradleproperties] 中也有注释说明。

| 属性                  | 描述                                                                                                                                                                                                                             | 示例                                    |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------|
| `org.gradle.jvmargs`      | 允许您向 Gradle 传递额外的 JVM 参数。通常情况下，这是用来分配更多/更少的内存给 Gradle 的。请注意，这仅针对 Gradle 本身，而不是 Minecraft                                                                 | `org.gradle.jvmargs=-Xmx3G`                |
| `org.gradle.daemon`       | 是否在构建时 Gradle 应该使用守护进程                                                                                                                                                                                     | `org.gradle.daemon=false`                  |
| `org.gradle.parallel`     | Gradle 是否应 fork JVM 以并行执行项目                                                                                                                                                                                     | `org.gradle.parallel=false`                  |
| `org.gradle.caching`      | Gradle 是否应重复使用之前构建的任务输出                                                                                                                                                                                     | `org.gradle.caching=false`                  |
| `org.gradle.configuration-cache`  | Gradle 是否应重复使用以前的构建配置                                                                                                                                                                                     | `org.gradle.configuration-cache=false`                  |
| `org.gradle.debug`        | Gradle 是否设置为调试模式调试模式主要意味着更多的 Gradle 日志输出请注意，这是针对 Gradle 本身，而不是 Minecraft                                                                                                | `org.gradle.debug=false`                   |
| `minecraft_version`       | 您正在修改的 Minecraft 版本必须与 `neo_version`匹配                                                                                                                                                                | `minecraft_version=1.20.6`                 |
| `minecraft_version_range` | 该 MOD 可以使用的 Minecraft 版本范围，作为[Maven 版本范围][mvr]请注意，[快照、预发布版本和候选发布版本][mcversioning] 不保证排序正确，因为它们不遵循 maven 版本控制    | `minecraft_version_range=[1.20.6,1.21)`    |
| `neo_version`             | 您正在修改的 NeoForge 版本必须与 `minecraft_version` 匹配有关 NeoForge 版本控制的更多信息，请参阅 [NeoForge Versioning][neoversioning]                                                           | `neo_version=20.6.62`                      |
| `neo_version_range`       | 该 MOD 可使用的 NeoForge 版本范围，作为[Maven 版本范围][mvr]                                                                                                                                                           | `neo_version_range=[20.6.62,20.7)`         |
| `loader_version_range`    | 该 MOD 可使用的 MOD 加载器的版本范围，以[Maven 版本范围][mvr]表示。请注意，加载器版本与 NeoForge 版本是解耦的                                                                           | `loader_version_range=[1,)`                |
| `mod_id`                  | 查看 [The Mod ID][modid].                                                                                                                                                                                                                | `mod_id=examplemod`                        |
| `mod_name`                | MOD 的可读显示名称。默认情况下，这只能在 MOD 列表中看到，但 [JEI][jei] 等 MOD 也会在物品工具提示中显示该名称                                                | `mod_name=Example Mod`                     |
| `mod_license`             | 您的 MOD 所使用的许可证。建议将其设置为您使用的[SPDX 标识符][spdx]和/或许可证链接您可以访问 <https://choosealicense.com/> 帮助选择要使用的许可证 | `mod_license=MIT`                          |
| `mod_version`             | MOD 列表中显示的 MOD 版本更多信息请参阅[版本控制][versioning]页面                                                                                                                         | `mod_version=1.0`                          |
| `mod_group_id`            | 查看 [The Group ID][group].                                                                                                                                                                                                              | `mod_group_id=com.example.examplemod`      |
| `mod_authors`             | MOD 的作者，显示在 MOD 列表里                                                                                                                                                                                          | `mod_authors=ExampleModder`                |
| `mod_description`         | 多行字符串形式的 MOD 描述，显示在 MOD 列表中可以使用换行符 (`\n`) 并能正确识别                                                                                         | `mod_description=Example mod description.` |

### The Mod ID

The mod ID is the main way your mod is distinguished from others. It is used in a wide variety of places, including as the namespace for your mod's [registries][registration], and as your [resource and data pack][resource] namespaces. Having two mods with the same id will prevent the game from loading.

As such, your mod ID should be something unique and memorable. Usually, it will be your mod's display name (but lower case), or some variation thereof. Mod IDs may only contain lowercase letters, digits and underscores, and must be between 2 and 64 characters long (both inclusive).
MOD ID 是将你的 MOD 与其他 MOD 区分开来的主要方式。它被用在很多地方，包括作为 mod 的[注册表][registration]命名空间，以及[资源和数据包][resource]命名空间。如果两个 mod 的 ID 相同，游戏将无法加载。

因此，MOD ID 应该是独一无二且易于记忆的。通常，它将是 MOD 的显示名称（但要小写），或其中的一些变体。MOD ID 只能包含小写字母、数字和下划线，长度必须在 2 到 64 个字符之间（包括这两个字符）。
:::info
在 `gradle.properties` 文件中更改此属性将自动在所有地方应用更改，除了主 MOD 类中的 [`@Mod` 注解][javafml]。在那里，您需要手动更改它以匹配 `gradle.properties` 文件中的值。
:::

### The Group ID

虽然 `build.gradle` 中的 `group` 属性仅在您计划将 MOD 发布到 Maven 时才需要，但始终正确设置此属性被认为是一种良好的实践。这是通过 `gradle.properties` 文件中的 `mod_group_id` 属性为您完成的。

Group ID 应设置为您的顶级包名。有关更多信息，请参阅 [Packaging][packaging]。

```properties
# 在您的 gradle.properties 文件中
mod_group_id=com.example
```

您的 Java 源代码 (`src/main/java`) 中的包结构也应符合此结构，其中内部包表示 MOD ID：

```text
com
- example (在 group 属性中指定的顶级包)
    - mymod (MOD ID)
        - MyMod.java (重命名后的 ExampleMod.java)
```

## `neoforge.mods.toml`

`neoforge.mods.toml` 文件位于 `src/main/resources/META-INF/neoforge.mods.toml`，是一个 [TOML][toml] 格式的文件，用于定义你的 MOD 的元数据。它还包含有关你的 MOD 应如何加载到游戏中的附加信息，以及在“Mods”菜单中显示的展示信息。[MDK 提供的 `neoforge.mods.toml` 文件][mdkneoforgemodstoml] 包含了注释，解释了每个条目的作用，这里将更详细地说明这些内容。

`neoforge.mods.toml` 可以分为三个部分：非 MOD 特定的属性，这些属性与 MOD 配置文件相关联；MOD 属性，每个 MOD 都有一个对应的部分；以及依赖配置，每个 MOD 或 MOD 依赖都有一个对应的部分。与 `neoforge.mods.toml` 文件相关的一些属性是强制性的；强制性属性需要指定一个值，否则会抛出异常。
:::note
在默认的 MDK 中，Gradle 会使用 `gradle.properties` 文件中指定的值替换此文件中的各种属性。例如，行 `license="${mod_license}"` 表示 `license` 字段会被 `gradle.properties` 中的 `mod_license` 属性替换。像这样被替换的值应该在 `gradle.properties` 中更改，而不是在这里直接修改。
:::

### 非 MOD 特定属性

非 MOD 特定属性是与 JAR 文件本身相关的属性，指示如何加载 MOD 以及任何额外的全局元数据。

| 属性                 | 类型     | 默认值         | 描述                                                                                                                                                                                                                                                                                                                                         | 示例                                                                        |
|----------------------|----------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| `modLoader`          | string   | **必填**       | MOD 使用的语言加载器。可以用于支持替代语言结构，例如主文件的 Kotlin 对象，或不同的入口点确定方法，例如接口或方法。NeoForge 提供了 Java 加载器 [`"javafml"`][javafml] 和低代码/无代码加载器 [`"lowcodefml"`][lowcodefml]                                                                                                                          | `modLoader="javafml"`                                                          |
| `loaderVersion`      | string   | **必填**       | 语言加载器的可接受版本范围，以 [Maven 版本范围][mvr] 表示。对于 `javafml` 和 `lowcodefml`，当前版本为 `1`                                                                                                                                                                                      | `loaderVersion="[1,)"`                                                         |
| `license`            | string   | **必填**       | 此 JAR 中 MOD 的许可证。建议将其设置为使用的 [SPDX 标识符][spdx] 和/或许可证的链接。你可以访问 <https://choosealicense.com/> 来帮助你选择所需的许可证                                                                                              | `license="MIT"`                                                                |
| `showAsResourcePack` | boolean  | `false`        | 当为 `true` 时，MOD 的资源将在“资源包”菜单中显示为单独的资源包，而不是与“MOD 资源”包合并                                                                                                                                                                           | `showAsResourcePack=true`                                                      |
| `showAsDataPack`     | boolean  | `false`        | 当为 `true` 时，MOD 的数据文件将在“数据包”菜单中显示为单独的数据包，而不是与“MOD 数据”包合并                                                                                                                                                                           | `showAsDataPack=true`                                                          |
| `services`           | array    | `[]`           | 你的 MOD 使用的服务数组。这是作为 NeoForge 实现的 Java 平台模块系统中 MOD 创建模块的一部分使用的                                                                                                                                                                                   | `services=["net.neoforged.neoforgespi.language.IModLanguageProvider"]`         |
| `properties`         | table    | `{}`           | 替换属性的表。这由 `StringSubstitutor` 使用，用于将 `${file.<key>}` 替换为其对应的值                                                                                                                                                                                                                    | `properties={"example"="1.2.3"}`（然后可以通过 `${file.example}` 引用）         |
| `issueTrackerURL`    | string   | _无_           | 表示报告和跟踪 MOD 问题的 URL                                                                                                                                                                                                                                                                            | `"https://github.com/neoforged/NeoForge/issues"`                               |

:::note
`services` 属性在功能上等同于在模块中指定 [`uses` 指令][uses]，这允许 [加载给定类型的服务][serviceload]。

或者，它可以在 `src/main/resources/META-INF/services` 文件夹中的服务文件中定义，其中文件名是服务的完全限定名称，文件内容是要加载的服务的名称（参见 [AtlasViewer MOD 的这个示例][atlasviewer]）。
:::

### MOD 特定属性

MOD 特定属性使用 `[[mods]]` 标头与指定的 MOD 绑定。这是一个 [表数组][array]；所有键/值属性将附加到该 MOD，直到下一个标头。

```toml
# examplemod1 的属性
[[mods]]
modId = "examplemod1"

# examplemod2 的属性
[[mods]]
modId = "examplemod2"
```

| 属性             | 类型     | 默认值                      | 描述                                                                                                                                                                                                                                                                    | 示例                                                         |
|------------------|----------|------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| `modId`          | string   | **必填**                    | 参见 [MOD ID][modid]                                                                                                                                                                                                                                                       | `modId="examplemod"`                                            |
| `namespace`      | string   | `modId` 的值                | MOD 的命名空间覆盖。必须是一个有效的 [MOD ID][modid]，但可以额外包含点或破折号。目前未使用                                                                                                                                        | `namespace="example"`                                           |
| `version`        | string   | `"1"`                        | MOD 的版本，最好采用 [Maven 版本控制的变体][versioning]。当设置为 `${file.jarVersion}` 时，它将被替换为 JAR 清单中 `Implementation-Version` 属性的值（在开发环境中显示为 `0.0NONE`） | `version="1.20.2-1.0.0"`                                        |
| `displayName`    | string   | `modId` 的值                | MOD 的显示名称。用于在屏幕上表示 MOD（例如，MOD 列表、MOD 不匹配）                                                                                                                                                                        | `displayName="Example Mod"`                                     |
| `description`    | string   | `'''MISSING DESCRIPTION'''`  | 在 MOD 列表屏幕上显示的 MOD 描述。建议使用 [多行字面字符串][multiline]。此值也可翻译，更多信息请参见 [翻译 MOD 元数据][i18n]                                                                | `description='''This is an example.'''`                         |
| `logoFile`       | string   | _无_                        | 用于 MOD 列表屏幕的图像文件的名称和扩展名。位置必须是从 JAR 或源集根目录开始的绝对路径（例如，主源集的 `src/main/resources`）。有效的文件名字符包括小写字母（`a-z`）、数字（`0-9`）、斜杠（`/`）、下划线（`_`）、点（`.`）和连字符（`-`）。完整字符集为 `[a-z0-9_-.]`                                                                  | `logoFile="test/example_logo.png"`                              |
| `logoBlur`       | boolean  | `true`                       | 是否使用 `GL_LINEAR*`（true）或 `GL_NEAREST*`（false）来渲染 `logoFile`。简单来说，这意味着在尝试缩放徽标时是否应模糊徽标                                                                                    | `logoBlur=false`                                                |
| `updateJSONURL`  | string   | _无_                        | 用于 [更新检查器][update] 的 JSON 的 URL，以确保你正在玩的 MOD 是最新版本                                                                                                                                                               | `updateJSONURL="https://example.github.io/update_checker.json"` |
| `features`       | table    | `{}`                         | 参见 [features]                                                                                                                                                                                                                                                                | `features={java_version="[17,)"}`                               |
| `modproperties`  | table    | `{}`                         | 与此 MOD 关联的键/值表。NeoForge 未使用，但主要用于 MOD 使用                                                                                                                                                                             | `modproperties={example="value"}`                               |
| `modUrl`         | string   | _无_                        | MOD 下载页面的 URL。目前未使用                                                                                                                                                                                                                       | `modUrl="https://neoforged.net/"`                               |
| `credits`        | string   | _无_                        | 在 MOD 列表屏幕上显示的 MOD 致谢和鸣谢                                                                                                                                                                                                             | `credits="The person over here and there."`                     |
| `authors`        | string   | _无_                        | 在 MOD 列表屏幕上显示的 MOD 作者                                                                                                                                                                                                                          | `authors="Example Person"`                                      |
| `displayURL`     | string   | _无_                        | 在 MOD 列表屏幕上显示的 MOD 展示页面的 URL                                                                                                                                                                                                             | `displayURL="https://neoforged.net/"`                           |
| `enumExtensions` | string   | _无_                        | 用于 [枚举扩展][enumextension] 的 JSON 文件路径                                                                                                                                                                                                          | `enumExtensions="META_INF/enumextensions.json"`                 |
| `featureFlags`   | string   | _无_                        | 用于 [功能标志][featureflags] 的 JSON 文件路径                                                                                                                                                                                                            | `featureFlags="META-INF/feature_flags.json"`                    |

#### Features

Features 系统允许 MOD 在加载时要求某些设置、软件或硬件可用。当某个 feature 不满足时，MOD 加载将失败，并通知用户相关要求。目前，NeoForge 提供以下 features：

| Feature          | 描述                                                                                                                                                                                                | 示例                             |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------|
| `javaVersion`   | Java 版本的可接受范围，以 [Maven 版本范围][mvr] 表示。这应该是 Minecraft 使用的支持版本                                                                                       | `features={javaVersion="[17,)"}`  |
| `openGLVersion` | OpenGL 版本的可接受范围，以 [Maven 版本范围][mvr] 表示。Minecraft 需要 OpenGL 3.2 或更高版本。如果你想要求更新的 OpenGL 版本，可以在这里设置  | `features={openGLVersion="[4.6,)"}` |

### Access Transformer 特定属性

[Access Transformer 特定属性][accesstransformer] 使用 `[[accessTransformers]]` 标头与指定的 Access Transformer 绑定。这是一个 [表数组][array]；所有键/值属性将附加到该 Access Transformer，直到下一个标头。Access Transformer 标头是可选的；但如果指定，则所有元素都是必填的。

| 属性   | 类型    | 默认值         | 描述                          | 示例          |
|:------:|:-------:|:-------------:|:-----------------------------:|:-------------:|
| `file` | string  | **必填**      | 参见 [添加 AT][accesstransformer] | `file="at.cfg"` |

### Mixin 配置属性

[Mixin 配置属性][mixinconfig] 使用 `[[mixins]]` 标头与指定的 Mixin 配置绑定。这是一个 [表数组][array]；所有键/值属性将附加到该 Mixin 块，直到下一个标头。Mixin 标头是可选的；但如果指定，则所有元素都是必填的。

| 属性     | 类型    | 默认值         | 描述                          | 示例                       |
|:--------:|:-------:|:-------------:|:-----------------------------:|:--------------------------:|
| `config` | string  | **必填**      | Mixin 配置文件的位置         | `config="examplemod.mixins.json"` |

### 依赖配置

MOD 可以指定其依赖项，NeoForge 在加载 MOD 之前会检查这些依赖项。这些配置使用 [表数组][array] `[[dependencies.<modid>]]` 创建，其中 `modid` 是使用依赖项的 MOD 的标识符。

| 属性           | 类型    | 默认值         | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                | 示例                                      |
|----------------|---------|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------|
| `modId`        | string  | **必填**       | 作为依赖项添加的 MOD 的标识符                                                                                                                                                                                                                                                                                                                                                                                                                           | `modId="jei"`                                |
| `type`         | string  | `"required"`   | 指定此依赖项的性质：`"required"` 是默认值，如果缺少此依赖项，则阻止 MOD 加载；`"optional"` 不会阻止 MOD 加载，但仍会验证依赖项是否兼容；`"incompatible"` 如果存在此依赖项，则阻止 MOD 加载；`"discouraged"` 允许 MOD 加载，但如果存在依赖项，则向用户显示警告| `type="incompatible"`                        |
| `reason`       | string  | _无_           | 可选的面向用户的消息，用于描述为什么需要此依赖项，或为什么它不兼容                                                                                                                                                                                                                                                                                                                                                                   | `reason="integration"`                       |
| `versionRange` | string  | `""`           | 语言加载器的可接受版本范围，以 [Maven 版本范围][mvr] 表示。空字符串匹配任何版本                                                                                                                                                                                                                                                                                                                                      | `versionRange="[1, 2)"`                      |
| `ordering`     | string  | `"NONE"`       | 定义 MOD 必须在此依赖项之前（`"BEFORE"`）或之后（`"AFTER"`）加载。如果顺序无关紧要，则返回 `"NONE"`                                                                                                                                                                                                                                                                                                                                   | `ordering="AFTER"`                           |
| `side`         | string  | `"BOTH"`       | 依赖项必须存在的 [物理端][sides]：`"CLIENT"`、`"SERVER"` 或 `"BOTH"`                                                                                                                                                                                                                                                                                                                                                                        | `side="CLIENT"`                              |
| `referralUrl`  | string  | _无_           | 依赖项下载页面的 URL。目前未使用                                                                                                                                                                                                                                                                                                                                                                                                           | `referralUrl="https://library.example.com/"` |

:::danger
两个 MOD 的 `ordering` 可能会导致循环依赖而崩溃，例如，如果 MOD A 必须 `"BEFORE"` MOD B 加载，同时 MOD B 必须 `"BEFORE"` MOD A 加载。
:::

## MOD 入口点

现在 `neoforge.mods.toml` 已经填写完毕，我们需要为 MOD 提供一个入口点。入口点本质上是执行 MOD 的起点。入口点本身由 `neoforge.mods.toml` 中使用的语言加载器决定。

### `javafml` 和 `@Mod`

`javafml` 是 NeoForge 为 Java 编程语言提供的语言加载器。入口点使用带有 `@Mod` 注解的公共类定义。`@Mod` 的值必须包含 `neoforge.mods.toml` 中指定的一个 MOD ID。然后，所有初始化逻辑（例如 [注册事件][events] 或 [添加 `DeferredRegister`s][registration]）可以在类的构造函数中指定。

主 MOD 类只能有一个公共构造函数；否则会抛出 `RuntimeException`。构造函数可以按**任意**顺序包含以下**任何**参数；没有明确要求必须包含哪些参数。但是，不允许有重复的参数。

| 参数类型       | 描述                                                                                              |
|----------------|----------------------------------------------------------------------------------------------------------|
| `IEventBus`    | [MOD 特定事件总线][modbus]（用于注册、事件等）                             |
| `ModContainer` | 包含此 MOD 元数据的抽象容器                                                       |
| `FMLModContainer` | 由 `javafml` 定义的实际容器，包含此 MOD 的元数据；是 `ModContainer` 的扩展 |
| `Dist`         | 此 MOD 正在加载的 [物理端][sides]                                                        |

```java
@Mod("examplemod") // Must match a mod id in the neoforge.mods.toml
public class ExampleMod {
    // Valid constructor, only uses two of the available argument types
    public ExampleMod(IEventBus modBus, ModContainer container) {
        // Initialize logic here
    }
}
```

默认情况下，`@Mod` 注解会在 所有 [端][sides] 加载。可以通过指定 `dist` 参数来更改此行为：

```java
// 必须与 neoforge.mods.toml 中的 mod id 匹配
// 此 MOD 类仅在物理客户端加载
@Mod(value = "examplemod", dist = Dist.CLIENT) 
public class ExampleModClient {
    // 有效的构造函数
    public ExampleModClient(FMLModContainer container, IEventBus modBus, Dist dist) {
        // 在这里初始化仅客户端的逻辑
    }
}
```

:::note
`neoforge.mods.toml` 中的条目不需要对应的 `@Mod` 注解。同样，`neoforge.mods.toml` 中的一个条目可以有多个 `@Mod` 注解，例如，如果你想分离通用逻辑和仅客户端逻辑。
:::

### `lowcodefml`

`lowcodefml` 是一种语言加载器，用于将数据包和资源包作为 MOD 分发，而无需代码中的入口点。它被指定为 `lowcodefml` 而不是 `nocodefml`，以便未来可能需要的少量代码扩展。

[accesstransformer]: ../advanced/accesstransformers.md#adding-ats
[array]: https://toml.io/en/v1.0.0#array-of-tables
[atlasviewer]: https://github.com/XFactHD/AtlasViewer/blob/1.20.2/neoforge/src/main/resources/META-INF/services/xfacthd.atlasviewer.platform.services.IPlatformHelper
[events]: ../concepts/events.md
[features]: #features
[group]: #the-group-id
[i18n]: ../resources/client/i18n.md#translating-mod-metadata
[javafml]: #javafml-and-mod
[jei]: https://www.curseforge.com/minecraft/mc-mods/jei
[lowcodefml]: #lowcodefml
[mcversioning]: versioning.md#minecraft
[mdkgradleproperties]: https://github.com/NeoForgeMDKs/MDK-1.21-NeoGradle/blob/main/gradle.properties
[mdkneoforgemodstoml]: https://github.com/NeoForgeMDKs/MDK-1.21-NeoGradle/blob/main/src/main/resources/META-INF/neoforge.mods.toml
[neoforgemodstoml]: #neoforgemodstoml
[mixinconfig]: https://github.com/SpongePowered/Mixin/wiki/Introduction-to-Mixins---The-Mixin-Environment#mixin-configuration-files
[modbus]: ../concepts/events.md#event-buses
[modid]: #the-mod-id
[mojmaps]: https://github.com/neoforged/NeoForm/blob/main/Mojang.md
[multiline]: https://toml.io/en/v1.0.0#string
[mvr]: https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html
[neoversioning]: versioning.md#neoforge
[packaging]: structuring.md#packaging
[registration]: ../concepts/registries.md#deferredregister
[resource]: ../resources/index.md
[serviceload]: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ServiceLoader.html#load(java.lang.Class)
[sides]: ../concepts/sides.md
[spdx]: https://spdx.org/licenses/
[toml]: https://toml.io/
[update]: ../misc/updatechecker.md
[uses]: https://docs.oracle.com/javase/specs/jls/se21/html/jls-7.html#jls-7.7.3
[versioning]: versioning.md
[enumextension]: ../advanced/extensibleenums.md
[featureflags]: ../advanced/featureflags.md
