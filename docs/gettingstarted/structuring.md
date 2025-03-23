# 构建你的 MOD

结构化的 MOD 有利于维护、贡献以及更清晰地理解代码库。以下列出了来自 Java、Minecraft 和 NeoForge 的一些建议。

:::note
你不必遵循以下建议；你可以按照你认为合适的方式构建你的 MOD。然而，仍然强烈建议这样做。
:::

## 包结构

在构建你的 MOD 时，选择一个唯一的顶级包结构。许多程序员会为不同的类、接口等使用相同的名称。Java 允许类具有相同的名称，只要它们位于不同的包中。因此，如果两个类具有相同的包和相同的名称，则只会加载其中一个，很可能会导致游戏崩溃。

```plaintext
a.jar
    - com.example.ExampleClass
b.jar
    - com.example.ExampleClass // 通常不会加载该类
```

在加载模块时，这一点更加重要。如果在不同模块中存在同名包下的类文件，这将导致 MOD 加载器在启动时崩溃，因为 MOD 模块会导出到游戏和其他 MOD 中。

```plaintext
module A
    - package X
        - class I
        - class J
module B
    - package X // 这个包会导致 MOD 加载器崩溃，因为已经有一个导出包 X 的模块存在
        - class R
        - class S
        - class T
```

因此，你的顶级包应该是你拥有的内容：域名、电子邮件地址、网站（或子域名）等。它甚至可以是你的名字或用户名，只要你能保证它在预期目标中是唯一可识别的。此外，顶级包还应与你的 [group id][group] 匹配。

|   类型    |       值       | 顶级包           |
|:---------:|:--------------:|:------------------|
|  域名     |   example.com  | `com.example`     |
| 子域名    | example.github.io | `io.github.example` |
| 电子邮件  | `example@gmail.com` | `com.gmail.example` |

接下来的包层级应该是你的 MOD ID（例如 `com.example.examplemod`，其中 `examplemod` 是 MOD ID）。这将确保，除非你有两个相同 ID 的 MOD（这种情况不应该发生），否则你的包在加载时不会出现任何问题。

你可以在 [Oracle 的教程页面][naming] 上找到一些额外的命名约定。

### 子包组织

除了顶级包之外，强烈建议将 MOD 的类按子包进行划分。有两种主要的方法可以实现这一点：

- **按功能分组**：为具有共同用途的类创建子包。例如，方块可以放在 `block` 下，物品放在 `item` 下，实体放在 `entity` 下，等等。Minecraft 本身也使用了类似的结构（有一些例外）。
- **按逻辑分组**：为具有共同逻辑的类创建子包。例如，如果你正在创建一个新的工作台类型，你可以将其方块、菜单、物品等放在 `feature.crafting_table` 下。

#### 客户端、服务器和数据包

通常，仅用于特定端或运行时的代码应与其它类隔离，放在单独的子包中。例如，与 [数据生成][datagen] 相关的代码应放在 `data` 包中，而仅在专用服务器上运行的代码应放在 `server` 包中。

强烈建议将 [仅客户端代码][sides] 隔离在 `client` 子包中。这是因为专用服务器无法访问 Minecraft 中的任何仅客户端包，如果你的 MOD 尝试访问它们，服务器将崩溃。因此，拥有一个专门的包可以提供一个良好的检查机制，以确保你不会在 MOD 中跨端访问。

## 类命名规范

常见的类命名规范可以更容易地解读类的用途或轻松定位特定类。

类通常以其类型作为后缀，例如：

- 一个名为 `PowerRing` 的 `Item` -> `PowerRingItem`。
- 一个名为 `NotDirt` 的 `Block` -> `NotDirtBlock`。
- 一个 `Oven` 的菜单 -> `OvenMenu`。

:::tip
Mojang 通常对所有类使用类似的结构，除了实体。实体类仅由其名称表示（例如 `Pig`、`Zombie` 等）。
:::

## 从多种方法中选择一种

执行某些任务的方法有很多：注册对象、监听事件等。通常建议通过使用单一方法来完成给定任务，以保持一致性。这提高了可读性，并避免了可能发生的奇怪交互或冗余（例如，你的事件监听器运行两次）。

[group]: index.md#the-group-id
[naming]: https://docs.oracle.com/javase/tutorial/java/package/namingpkgs.html
[datagen]: ../resources/index.md#data-generation
[sides]: ../concepts/sides.md
