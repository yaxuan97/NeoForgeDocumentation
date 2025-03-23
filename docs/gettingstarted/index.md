# 开始使用 NeoForge

本节介绍如何设置 NeoForge 工作区，以及如何运行和测试您的 MOD。

## 前提条件

- 熟悉 Java 编程语言，特别是其面向对象、多态、泛型和功能特性。
- 安装 Java 21 开发工具包 (JDK) 和 64 位 Java 虚拟机 (JVM)。 NeoForge 推荐并正式支持[OpenJDK 的微软版本][jdk]，但任何其他 JDK 也可正常工作。

:::caution
确保您使用的是 64 位 JVM。 一种检查方法是在终端运行 `java -version`。 使用 32 位 JVM 时可能会出现问题，这是因为 32 位 JVM 已经不再支持很多功能。
:::

- 熟悉您选择的集成开发环境（IDE）。
      - NeoForge 官方支持 [IntelliJ IDEA][intellij] 和 [Eclipse][eclipse], 这两个 IDE 都集成了 Gradle 支持。 不过，也可以使用任何集成开发环境，例如 Netbeans, Visual Studio Code, Vim 或 Emacs。
- 熟悉 [Git][git] 和 [GitHub][github] 。 这在技术上不是必需的，但它会让你的工作轻松很多。

## 设置工作区

- 打开模块开发工具包（MDK）（[ModDevGradle][mdgmdk]或[NeoGradle][ngmdk]）的 GitHub 仓库，点击 "Use this template（使用此模板）"并将新创建的仓库 Clone 到本地计算机。
      - 如果不想使用 GitHub，或者想获取较早提交的模板，也可以下载该版本库的 ZIP 文件（在Code -> Download ZIP 下）并解压缩。
- 打开集成开发环境并导入 Gradle 项目。 Eclipse 和 IntelliJ IDEA 会自动完成这项工作。 如果你的集成开发环境没有这项功能，也可以通过终端命令 `gradlew` 来完成。
      - 首次执行此操作时，Gradle 会下载 NeoForge 的所有依赖项，包括 Minecraft 本身，并对其进行反编译。 这可能需要相当长的时间（可能长达数小时，这取决于你的硬件和网络环境）。
      - 每当你对 Gradle 文件进行修改时，都需要通过集成开发环境中的 "Reload Gradle" 按钮或终端命令 "gradlew" 重新加载 Gradle 的修改。

## 自定义您的 MOD 信息

MOD 的许多基本属性都可以在 `gradle.properties` 文件中更改。 这包括一些基本的东西，如 MOD 名称或 MOD 版本。 更多信息，请参阅`gradle.properties`文件中的注释，或查看[`gradle.properties`文件的文档][properties]。

如果你想在此基础上修改编译过程，可以编辑`build.gradle`文件。 NeoGradle 是 NeoForge 的 Gradle 插件，它提供了几个配置选项，其中几个在`build.gradle`文件的注释中有所解释。 有关完整文档，请参阅[NeoGradle 文档][neogradle]。

:::caution
只有知道自己在做什么，才能编辑 `build.gradle` 和 `settings.gradle` 文件。 所有基本属性都可以通过 `gradle.properties` 设置。
:::

## 构建和测试您的 MOD

要构建您的 MOD，请运行 `gradlew build`。 这将在 `build/libs` 中输出一个文件，文件名为 `<archivesBaseName>-<version>.jar`。 `<存档库名>`和`<版本>`是`build.gradle`设置的属性，默认分别是`gradle.properties`文件中的`mod_id`和`mod_version`值；如果需要，可以在`build.gradle`中更改。 生成的 JAR 文件可以放在支持 NeoForge 的 Minecraft 设置的 `mods` 文件夹中，或者上传到 mod 发布平台。

要在测试环境中运行 MOD，可以使用生成的运行配置或相关任务（如 `gradlew runClient`）。 这将从相应的运行目录（如 `runs/client` 或 `runs/server`）启动 Minecraft，同时还会指定任何源码集。 默认的 MDK 包含 `main` 源代码集，因此在 `src/main/java` 中编写的任何代码都会被应用。

### 服务器测试

如果您通过运行配置或 `gradlew runServer` 运行专用服务器，服务器将立即关闭。 您需要通过编辑运行目录下的 `eula.txt` 文件来接受 Minecraft EULA。

接受后，服务器将加载并在 `localhost`（或默认的 `127.0.0.1`）下可用。 但是，您仍然无法加入，因为服务器将默认进入在线模式，而这需要身份验证（Dev 玩家没有）。 要解决这个问题，请再次停止服务器，并将 `server.properties` 文件中的 `online-mode` 属性设置为 `false`。 现在，启动服务器，就可以连接了。

:::tip
您应该始终在专用服务器环境中也测试您的 MOD。 这包括[纯客户端 MOD][client]，因为这些 MOD 在服务器上加载时不应该做任何事情。
:::

[client]: ../concepts/sides.md
[eclipse]: https://www.eclipse.org/downloads/
[git]: https://www.git-scm.com/
[github]: https://github.com/
[intellij]: https://www.jetbrains.com/idea/
[jdk]: https://learn.microsoft.com/en-us/java/openjdk/download#openjdk-21
[mdgmdk]: https://github.com/NeoForgeMDKs/MDK-1.21.4-ModDevGradle
[ngmdk]: https://github.com/NeoForgeMDKs/MDK-1.21.4-NeoGradle
[neogradle]: https://docs.neoforged.net/neogradle/docs/
[properties]: modfiles.md#gradleproperties
