# 版本控制

本文将分解 Minecraft 和 NeoForge 中的版本控制方式，并为 MOD 版本控制提供一些建议。

## Minecraft

Minecraft 使用 [语义化版本控制][semver]。语义化版本控制，简称 "semver"，格式为 `major.minor.patch`。例如，Minecraft 1.20.2 的主版本号为 1，次版本号为 20，修订版本号为 2。

自 2011 年 Minecraft 1.0 发布以来，Minecraft 一直使用 `1` 作为主版本号。在此之前，版本控制方案经常变化，有像 `a1.1`（Alpha 1.1）、`b1.7.3`（Beta 1.7.3）甚至 `infdev` 版本，这些版本根本没有遵循明确的版本控制方案。由于 `1` 作为主版本号已经持续了十多年，并且由于 Minecraft 2 的内部玩笑，普遍认为这种情况不太可能改变。

### 快照

快照偏离了标准的 semver 方案。它们的标签为 `YYwWWa`，其中 `YY` 表示年份的最后两位数字（例如 `23`），`WW` 表示该年的周数（例如 `01`）。因此，快照 `23w01a` 是 2023 年第一周发布的快照。

`a` 后缀用于同一周内发布两个快照的情况（第二个快照将命名为 `23w01b` 等）。Mojang 过去偶尔使用过这种方式。替代后缀也曾用于像 `20w14infinite` 这样的快照，这是 [2020 年无限维度的愚人节玩笑][infinite]。

### 预发布和候选版本

当快照周期接近完成时，Mojang 开始发布所谓的预发布版本。预发布版本被认为是该版本的功能完整版本，仅专注于修复错误。它们使用 semver 表示法，后缀为 `-preX`。例如，1.20.2 的第一个预发布版本命名为 `1.20.2-pre1`。可以有多个预发布版本，通常也会有多个，分别后缀为 `-pre2`、`-pre3` 等。

类似地，当预发布周期完成时，Mojang 会发布候选版本 1（后缀为 `-rc1`，例如 `1.20.2-rc1`）。Mojang 的目标是发布一个候选版本，如果没有进一步的错误发生，就可以发布它。然而，如果出现意外错误，则可能会有 `-rc2`、`-rc3` 等版本，类似于预发布版本。

## NeoForge

NeoForge 使用了一种改进的 semver 系统：主版本号是 Minecraft 的次版本号，次版本号是 Minecraft 的修订版本号，修订版本号是 NeoForge 的“实际”版本号。例如，NeoForge 20.2.59 是 Minecraft 1.20.2 的第 60 个版本（我们从 0 开始计数）。开头的 `1` 被省略，因为它不太可能改变，原因见 [上文][minecraft]。

NeoForge 中的一些地方也使用 [Maven 版本范围][mvr]，例如 [`neoforge.mods.toml`][neoforgemodstoml] 文件中的 Minecraft 和 NeoForge 版本范围。这些大部分与 semver 兼容，但并不完全兼容（例如，`pre` 标签不被考虑）。

## MOD

没有绝对最佳的版本控制系统。不同的开发风格、项目范围等都会影响选择哪种版本控制系统。有时，版本控制系统也可以组合使用。本节试图概述一些常用的版本控制系统，并提供实际示例。

通常，MOD 的文件名格式为 `modid-<version>.jar`。因此，如果我们的 MOD ID 是 `examplemod`，版本是 `1.2.3`，我们的 MOD 文件将命名为 `examplemod-1.2.3.jar`。

:::note
版本控制系统是建议，而不是严格执行的规则。特别是在版本何时更改（“提升”）以及如何更改方面。如果你想使用不同的版本控制系统，没有人会阻止你。
:::

### 语义化版本控制

语义化版本控制（"semver"）由三部分组成：`major.minor.patch`。主版本号在对代码库进行重大更改时提升，通常与重大新功能和错误修复相关。次版本号在引入次要功能时提升，修订版本号在更新仅包含错误修复时提升。

普遍认为，任何 `0.x.x` 版本都是开发版本，首次（完整）发布时，版本应提升到 `1.0.0`。

在实践中，“次版本号用于功能，修订版本号用于错误修复”的规则经常被忽略。一个流行的例子是 Minecraft 本身，它通过次版本号进行主要功能更新，通过修订版本号进行次要功能更新，并通过快照进行错误修复（见上文）。

根据 MOD 的更新频率，这些数字可以更小或更大。例如，[Supplementaries][supplementaries] 的版本为 `2.6.31`（截至撰写时）。三位数甚至四位数的版本号，尤其是在 `patch` 部分，是完全可能的。

### “简化”和“扩展”语义化版本控制

有时，semver 可能只有两个数字。这是一种“简化”的 semver，或“两部分”semver。它们的版本号只有 `major.minor` 格式。这通常用于小型 MOD，这些 MOD 只添加了几个简单的对象，因此很少需要更新（除了 Minecraft 版本更新），通常永远停留在 `1.0` 版本。

“扩展”semver，或“四部分”semver，有四个数字（例如 `1.0.0.0`）。根据 MOD 的不同，格式可以是 `major.api.minor.patch`，或 `major.minor.patch.hotfix`，或完全不同的格式——没有标准的方式。

对于 `major.api.minor.patch`，`major` 版本与 `api` 版本解耦。这意味着 `major`（功能）部分和 `api` 部分可以独立提升。这通常用于暴露 API 供其他 MOD 开发者使用的 MOD。例如，[Mekanism][mekanism] 目前的版本是 10.4.5.19（截至撰写时）。

对于 `major.minor.patch.hotfix`，修订版本号分为两部分。这是 [Create][create] MOD 使用的方法，目前的版本是 0.5.1f（截至撰写时）。请注意，Create 将热修复表示为字母而不是第四个数字，以保持与常规 semver 的兼容性。

:::info
简化 semver、扩展 semver、两部分 semver 和四部分 semver 并不是官方术语或标准化格式。
:::

### Alpha、Beta、Release

与 Minecraft 本身一样，MOD 开发通常遵循软件工程中经典的 `alpha`/`beta`/`release` 阶段，其中 `alpha` 表示不稳定/实验性版本（有时也称为 `experimental` 或 `snapshot`），`beta` 表示半稳定版本，`release` 表示稳定版本（有时称为 `stable` 而不是 `release`）。

一些 MOD 使用主版本号来表示 Minecraft 版本更新。例如，[JEI][jei] 使用 `13.x.x.x` 表示 Minecraft 1.19.2，`14.x.x.x` 表示 1.19.4，`15.x.x.x` 表示 1.20.1（没有 1.19.3 和 1.20.0 的版本）。其他 MOD 将标签附加到 MOD 名称中，例如 [Minecolonies][minecolonies] MOD，截至撰写时的版本为 `1.1.328-BETA`。

### 包含 Minecraft 版本

通常会在文件名中包含 MOD 所支持的 Minecraft 版本。这使得最终用户更容易找到 MOD 所支持的 Minecraft 版本。常见的位置是在 MOD 版本之前或之后，前者比后者更常见。例如，JEI 版本 `16.0.0.28`（截至撰写时的最新版本）对于 1.20.2 将变为 `jei-1.20.2-16.0.0.28` 或 `jei-16.0.0.28-1.20.2`。

### 包含 MOD 加载器

你可能知道，NeoForge 并不是唯一的 MOD 加载器，许多 MOD 开发者在多个平台上开发。因此，需要一种方法来区分同一 MOD 同一版本但用于不同 MOD 加载器的两个文件。

通常，这是通过在名称中包含 MOD 加载器来实现的。`jei-neoforge-1.20.2-16.0.0.28`、`jei-1.20.2-neoforge-16.0.0.28` 或 `jei-1.20.2-16.0.0.28-neoforge` 都是有效的方式。对于其他 MOD 加载器，`neoforge` 部分将替换为 `forge`、`fabric`、`quilt` 或任何其他你可能与 NeoForge 一起开发的 MOD 加载器。

### 关于 Maven 的说明

Maven 是用于依赖项托管的系统，其版本控制系统在某些细节上与 semver 不同（尽管一般的 `major.minor.patch` 模式保持不变）。相关的 [Maven 版本范围（MVR）][mvr] 系统在 NeoForge 中的一些地方使用（见 [上文][neoforge]）。在选择版本控制方案时，你应该确保它与 MVR 兼容，否则 MOD 将无法依赖于你的 MOD 的特定版本！

[create]: https://www.curseforge.com/minecraft/mc-mods/create
[infinite]: https://minecraft.wiki/w/Java_Edition_20w14∞
[jei]: https://www.curseforge.com/minecraft/mc-mods/jei
[mekanism]: https://www.curseforge.com/minecraft/mc-mods/mekanism
[minecolonies]: https://www.curseforge.com/minecraft/mc-mods/minecolonies
[minecraft]: #minecraft
[neoforgemodstoml]: modfiles.md#neoforgemodstoml
[mvr]: https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html
[mvr]: https://maven.apache.org/ref/3.5.2/maven-artifact/apidocs/org/apache/maven/artifact/versioning/ComparableVersion.html
[neoforge]: #neoforge
[pre]: #pre-releases
[rc]: #release-candidates
[semver]: https://semver.org/
[supplementaries]: https://www.curseforge.com/minecraft/mc-mods/supplementaries
