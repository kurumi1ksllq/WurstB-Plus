# AGENTS.md — WurstB+ Plus

面向在本仓库里干活的人（和 agent）。只写在本仓库里实际读到、验证过的东西；
拿不准的放最后一节「待确认」，不要照着猜。

> 本仓库原本没有 `AGENTS.md`。本文件是从 `PROJECT_INDEX.md`、`docs/RELEASE.md`、
> `docs/PORTING-NEW-VERSIONS.md`、`CHANGELOG.md`、各 `build.gradle` / `scripts/` 里
> 合并出来的，并逐条核对过源码与目录。

---

## 1. 这是什么

`WurstB+ Plus` —— 基于 Wurst 代码结构扩展的 Minecraft 客户端 mod（`mod_id = wurstpenguin`，
GPL-3.0-or-later，作者 Penguin）。**单仓库、多工程**：每个 Minecraft 版本 × 加载器都是
一个**独立的 Gradle 工程**，各自有 `build.gradle` / `settings.gradle` / gradle wrapper 和
**完整复制的源码树**，工程之间不共享 sourceSet。

**67 个独立 Gradle 构建工程**（实测 `build.gradle` 计数 = 67）：

| 路径 | 加载器 / 版本 | 项目版本 |
| --- | --- | --- |
| `.`（根） | Forge 47.4.10 / MC 1.20.1 | **v1.6.0** |
| `fabric/` | Fabric Loader 0.16.14 / MC 1.20.1 | v1.5.0 |
| `neoforge/` | NeoForge 47.1.3 / MC 1.20.1 | v1.5.0 |
| `versions/*`（20 个） | Forge，MC 1.20.2–1.20.4、1.20.6、1.21–1.21.11、26.1–26.1.2、26.2、26.3 | v1.5.0 |
| `fabric/versions/*`（22 个） | Fabric，MC 1.20.2–1.20.6、1.21–1.21.11、26.1–26.1.2、26.2、26.3 | v1.5.0 |
| `neoforge/versions/*`（22 个） | NeoForge，同上 | v1.5.0 |

- Forge **没有**官方 1.20.5 / 1.21.2，所以这两个版本只有 Fabric / NeoForge 目录
  （`versions/1.20.5`、`versions/1.21.2` 不存在，已实测）。
- **只有根目录 Forge 1.20.1 是 v1.6.0**；其余 66 个工程停在 v1.5.0。v1.6 新增的
  子系统只存在于根工程，跨工程改动时不要假设它已移植。
- 详细的逐包/逐文件索引见 `PROJECT_INDEX.md`，发布矩阵与逐版本移植状态见
  `docs/RELEASE.md`、`docs/PORTING-NEW-VERSIONS.md`、`PORTING_TASK.md`、`CHANGELOG.md`。

## 2. 技术栈与版本（根工程实测自 `build.gradle` / `gradle.properties`）

| 项 | 值 |
| --- | --- |
| 语言 | Java（源码 UTF-8，缩进用 **Tab**，见第 6 节） |
| Java 工具链 | **17**（`java.toolchain.languageVersion`，`options.release = 17`，`gradle.properties` 不含 JDK 路径） |
| 各版本 JDK | 1.20.x→17；1.21.1/1.21.11→21；26.x→25（见 `scripts/common.ps1` 的 `$WurstbJdkMajor`） |
| 构建 | Gradle **8.11**（根 wrapper；`gradle/wrapper/gradle-wrapper.properties` 指向腾讯镜像 `mirrors.cloud.tencent.com/gradle/gradle-8.11-bin.zip`） |
| 构建插件 | ForgeGradle `[6.0,6.2)` + `org.spongepowered.mixin` + JarJar（根工程）；其余工程另有 FG7 / ModDevGradle / Loom |
| Minecraft | 1.20.1 |
| Forge | 47.4.10 |
| 映射 | Mojang 官方映射（`mappings channel: "official"`） |
| MixinExtras | 0.5.4（`io.github.llamalad7:mixinextras-forge`，根工程保留兼容配置） |
| 测试 | JUnit 5（`org.junit.jupiter:junit-jupiter:5.10.2`，`useJUnitPlatform()`） |
| mod 版本 | `v1.6.0-Forge-1.20.1` |
| maven group | `net.wurstpenguin` |
| 产物名 | `WurstB+ Plus-v1.6.0-Forge-1.20.1.jar`（根工程 `jarJar` 产物，约 68 MB） |

根工程内嵌（jarJar）19 个依赖，关键几个：`skiko-awt:0.8.19`、`kotlin-stdlib`、
`kotlinx-coroutines-core-jvm`、`baritone-api-forge`、`java-stream-player`、`jlayer`、
`jaudiotagger`、`netty-codec-socks`/`netty-handler-proxy`、`zxing:core`。

> `baritone-maven/`、`fabric/baritone-maven/` 被 `.gitignore` 排除，新克隆的仓库里**没有**它，
> 属预期（`build-all.ps1` 因此传 `-AllowUnresolved`）。

## 3. 目录结构

```text
.
├── build.gradle / settings.gradle / gradle.properties   # 根 Forge 1.20.1 工程（v1.6.0）
├── src/main/java/net/wurstclient/                       # 根工程源码（Mojmap，Forge）
├── src/main/resources/                                  # mixins json / AT / mods.toml / assets / shaders
├── src/test/java/net/wurstclient/                       # 根工程单测
├── versions/<MC>/                                       # 各 MC 版本的独立 Forge 工程
├── fabric/  + fabric/versions/<MC>/                     # Fabric 工程
├── neoforge/ + neoforge/versions/<MC>/                  # NeoForge 工程
├── gradle/init-mirrors.gradle                           # 国内镜像初始化脚本（依赖仓库 / BMCLAPI）
├── gradle/wrapper/                                      # wrapper jar + properties
├── scripts/                                             # 构建、启动、冒烟、审计脚本（见第 4 节）
├── docs/                                                # 架构 / 移植 / 发布文档（见 README 之外的权威来源）
├── PROJECT_INDEX.md                                     # 源码包/文件级索引（权威）
├── PORTING_TASK.md                                      # 移植与修复任务清单（含根因与验证）
├── CHANGELOG.md                                         # 版本变更
└── .github/workflows/sync-version-branches.yml          # 唯一 CI：把 main 同步到 22 个 per-version 分支
```

根工程 `src/main/java/net/wurstclient/` 顶层包（实测 `ls`）：

`addon` `ai` `altmanager` `background` `clickgui2` `command` `commands` `compose` `discord`
`event` `events` `gui` `hack` `hacks` `hud` `hud2` `keybinds` `macros` `mixin` `mixinterface`
`music` `nochatreports` `options` `other_feature` `other_features` `perimeter` `proxy`
`render` `seed` `serverfinder` `settings` `twilight` `update` `util` `waypoints`，
根包下还有 `WurstClient`、`WurstForgeInitializer`、`Feature`、`Category`、`FriendsList`、
`RotationFaker`、`WurstTranslator` 等。

### 入口与关键边界（来自 `PROJECT_INDEX.md`，与源码目录一致）

| 领域 | 文件 |
| --- | --- |
| 客户端生命周期 | `src/main/java/net/wurstclient/WurstClient.java` |
| Forge 入口 | `src/main/java/net/wurstclient/WurstForgeInitializer.java` |
| Fabric 入口 | `WurstInitializer`（`fabric/` 各工程） |
| Hack 基类 / 注册表 | `hack/Hack.java`、`hack/HackList.java` |
| 事件 | `event/EventManager.java`、`event/WurstSubscriber.java` |
| 设置持久化 | `settings/Setting.java`、`settings/SettingsFile.java` |
| Mixin 专用包 | `net.wurstclient.mixin/` —— **普通运行类禁止放进该包** |
| 平台适配 | `util/PlatformUtils.java` |

### 资源 / 生成物边界

- `src/main/resources/wurst.mixins.json`：根工程 **74 条 client mixin**（实测 JSON 解析：`client=74`，
  `mixins=0`，`server=0`），refmap 为 `wurstpenguin-refmap.json`。
- `src/main/resources/mixins.baritone.json`、`META-INF/accesstransformer.cfg`、`META-INF/mods.toml`。
- **生成物/构建目录**：`build/`、`.gradle/`、`run/`、`.test/`、`baritone-maven/`、`_smoke/`、
  `.recon/`、`_porting_stale/`、`_tools/`、`.gradle-compose-cache/` —— 都在 `.gitignore` 里，
  **不要提交、也不要手改**。

## 4. 怎么跑（命令写全）

> 下面凡是标「（未验证）」的，是我这次**没有实际执行**、只从脚本/Gradle 配置里读出来的。
> 本次实际执行并成功的只有：`gradlew.bat --version`（Gradle 8.11）与 4.2 节的
> `gradlew.bat test`（BUILD SUCCESSFUL，187 类 / 1192 项 / 0 失败）。

### 4.1 工具链检查 / 版本

```powershell
# 实测通过（本次以 JDK 21 launcher 运行）：输出 Gradle 8.11
.\gradlew.bat --offline --console=plain --version
```

```powershell
# 检查本机 JDK、wrapper 发行版、v1.6 测试源是否齐全（未验证）
powershell -ExecutionPolicy Bypass -File scripts\doctor.ps1
```

JDK 通过环境变量覆盖：`WURSTBPLUS_JAVA17`、`WURSTBPLUS_JAVA21`、`WURSTBPLUS_JAVA25`
（见 `scripts/common.ps1`）。

### 4.2 编译 / 测试（根工程）

```bash
# 实测通过（2026-10-06，Java 17）：BUILD SUCCESSFUL in 2m 50s
# 结果：187 个测试类 / 1192 项 / 0 失败 / 1 跳过
# 首次运行必须联网（拉 ForgeGradle 插件 + Minecraft 反编译缓存，~3.3GB cache）；之后可加 --offline
JAVA_HOME="E:/JDK/jdk-17.0.8" ./gradlew.bat test --console=plain
```

```powershell
# 封装脚本（未验证；会给 JAVA_HOME 指到 JDK 17，并走 --console=plain）
powershell -ExecutionPolicy Bypass -File scripts\run-unit-tests.ps1
powershell -ExecutionPolicy Bypass -File scripts\run-unit-tests.ps1 -Offline
powershell -ExecutionPolicy Bypass -File scripts\run-unit-tests.ps1 -Class net.wurstclient.music.NeteaseCloudApiTest
```

其它直接 Gradle 调用（未验证）：

```bat
gradlew.bat compileJava
gradlew.bat test --tests "net.wurstclient.music.NeteaseCloudApiTest"
gradlew.bat test --offline
```

根工程测试报告：`build/reports/tests/test/index.html`；XML 在 `build/test-results/test/`
（判失败数读这些 XML 比 grep 控制台输出稳）。

> **两个前置坑（实测）**：
> 1. 跑 `test` 前，仓库根必须有 `baritone-api-forge-1.20.1.jar`（根工程 `flatDir { dirs "." }`）。
>    该 jar 被 `.gitignore` 排除，新 clone 的仓库里**没有**；拿官方的
>    `baritone-api-forge-1.10.3.jar`（`cabaletta/baritone` releases，官方标注 MC 1.20–1.20.1）
>    重命名为 `baritone-api-forge-1.20.1.jar` 放到仓库根即可让根工程编译/测试跑起来
>    （仅用于本地开发；正式产物依赖的是本项目改造过的内嵌 Baritone，发布时需换回）。
> 2. **不要用 `versions/<MC>/gradlew.bat` 跑子工程**——58 个版本工程只有
>    `gradle-wrapper.properties`、没有 `gradle-wrapper.jar`，会报 `GradleWrapperMain` 找不到。
>    用根 wrapper 加 `-p`：`gradlew.bat -p versions/1.21.11 test`（该方式实测可行，但需
>    对应版本的插件能解析；离线时会因 ForgeGradle 7 未缓存而失败）。

### 4.3 打包（根工程）

```bat
gradlew.bat jarJar            & rem 根工程正式产物（jarJar + reobfJarJar + 复制到测试实例）
```
- `jarJar` 完成后 `finalizedBy reobfJarJar`、`copyJarToTestMods`，并跑内嵌库重定位
  （`relocateNestedJars`）。产物落在 `build/libs/`。
- 整合包兼容构建（会加 `-packCompat` 后缀并跳过 `copyJarToTestMods`）（未验证）：
  ```bat
  gradlew.bat jarJar -PpackCompat       & rem 只跳过被崩溃日志点名的三个库
  gradlew.bat jarJar -PpackCompatFull   & rem 只内嵌 baritone + mixinextras
  gradlew.bat jarJar -PnoRelocate       & rem 关闭内嵌库重定位
  ```
- 各版本工程的产出任务不同：Forge 1.20.2–1.21.1 用 `jarJar`；Forge 1.21.3+ 用 `allJar`；
  NeoForge 用 `jar` / `build`；Fabric 用 `remapJar`（26.x 用 `jar`）。权威表见
  `scripts/build-all.ps1` 里的 `$projects`。

### 4.4 启动客户端 / 服务端（真实启动，需先有对应实例）

```powershell
# 指定 MC 版本 + 加载器（自动带 gradle/init-mirrors.gradle）（未验证）
.\scripts\run-client.ps1 26.2
.\scripts\run-client.ps1 1.21.9 fabric
.\scripts\run-client.ps1 1.20.1 neoforge -Offline
.\scripts\run-client.ps1 26.2 -Server
```
```bash
# Git Bash / Linux / macOS
./scripts/run-client.sh 26.2
./scripts/run-client.sh 1.21.9 fabric
./scripts/run-client.sh 1.20.1 neoforge --offline
```
- 工程定位规则：MC=1.20.1 时 Forge→`.`, Fabric→`fabric/`, NeoForge→`neoforge/`；
  其余 Forge→`versions/<ver>/`，Fabric/NeoForge→`<loader>/versions/<ver>/`。
- 直接调用（未验证）：`./gradlew runClient --init-script <repo>/gradle/init-mirrors.gradle`
- 镜像开关：`-PnoMirror` 跳过全部镜像注入；把 `gradle/init-mirrors.gradle` 复制到
  `~/.gradle/init.d/` 可对所有构建生效。

### 4.5 冒烟 / 批量验证

```powershell
# 准备真实客户端测试实例（安装到 .test/versions，走 BMCLAPI）（未验证）
py -3 scripts/provision-smoke-clients.py --all --list
py -3 scripts/provision-smoke-clients.py --all
py -3 scripts/provision-smoke-clients.py --all --verify
py -3 scripts/provision-smoke-clients.py 1.21.6 --loaders fabric neoforge
```
```powershell
# 批量启动 .test 实例做「起不起来」判定（未验证）
powershell -ExecutionPolicy Bypass -File scripts/run-version-tests.ps1
powershell -ExecutionPolicy Bypass -File scripts/run-version-tests.ps1 -Version 1.21.6 -Loader Fabric -BuildOutputs -QuickPlayWorld WurstSmokeFresh
```
```bash
# 逐工程冒烟启动，自动分类 LAUNCHED / CRASHED / TIMEOUT（未验证）
python scripts/smoke-launch.py 26.2
python scripts/smoke-launch.py --all
python scripts/smoke-launch.py 26.1.2 --loaders fabric neoforge
```
- 判据：见到 `Sound engine started`（门槛）后**进程还需存活 `--settle` 秒且日志无致命行**才算
  `LAUNCHED`；主菜单级判据查不出只在进世界时才加载的类，需配 `-QuickPlayWorld`。
- 日志在 `_smoke/<工程>.log`，结果 TSV 可 `--results _smoke/results.tsv --skip-done` 续跑。

### 4.6 批量构建 / 发布（未验证）

```powershell
powershell -ExecutionPolicy Bypass -File scripts\build-all.ps1
powershell -ExecutionPolicy Bypass -File scripts\build-all.ps1 -Version 1.21.11
powershell -ExecutionPolicy Bypass -File scripts\build-all.ps1 -Loader NeoForge
powershell -ExecutionPolicy Bypass -File scripts\build-all.ps1 -Skip 26.2
powershell -ExecutionPolicy Bypass -File scripts\build-all.ps1 -Offline
```
```bash
python scripts/build-v1.5-release.py                 # 重建 v1.5 矩阵到 build/release-v1.5/
python scripts/validate-release-jars.py              # 校验发布 jar（zip 完好/元数据/Mixin/主类）
python scripts/upload-release-assets.py --dry-run    # 只报告会做什么
```

> `docs/RELEASE.md` 里提到的 `D:\WurstB\tmp-recon\build-v1.5-release.py`、`validate-jars.py`
> 是**仓库外**的旧路径，已丢失；仓库内的对应脚本是 `scripts/build-v1.5-release.py` 与
> `scripts/validate-release-jars.py`。以仓库内脚本为准。

### 4.7 Mixin 静态审计（改 mixin 前必跑）

本仓库的注解处理器**不生成 refmap**，`method = "..."` 与 `@At(target = "...")` 只在运行期解析 ——
编译能过，客户端一启动就崩。改 mixin 后先静态审计：

```bash
python scripts/audit-mixin-injections.py 26.3
python scripts/audit-mixin-injections.py 26.3 --tree neoforge
python scripts/audit-mixin-targets.py
```

### 4.8 其它脚本

```bash
python scripts/fetch-assets.py 26.2 --jobs 16     # 预取某版本 assets（走国内镜像）
python scripts/patch-baritone-mixin-level.py      # 修 26.2 内嵌 Baritone 的 compatibilityLevel
python scripts/sync-version-branches.py --dry-run # 同步 per-version 分支（CI 里会跑）
```

## 5. 不能碰的东西

- **生成物与构建目录**（`.gitignore` 覆盖，别提交、别手改）：`build/`、`.gradle/`、`run/`、
  `logs/`、`.test/`、`baritone-maven/`、`fabric/baritone-maven/`、`_smoke/`、`.recon/`、
  `_porting_stale/`、`_tools/`、`_artifacts/`、`download/`、`source/`、`config/`、
  `scripts/__pycache__/`、`.gradle-compose-cache/`。
- **密钥**：`.env` 永不进仓库（团队红线）。本仓库未发现 `.env` 或密钥文件；
  `altmanager/` 下出现 `token`/`credential` 字样的是微软登录流程的业务代码，不是硬编码密钥。
- **构建本地真需要的两个缺口（实测 2026-10-06）**：
  1. 根工程 `baritone-api-forge-1.20.1.jar` 缺失，不动它连 `compileJava`/`test` 都跑不了
     （4.2 节给了临时补齐办法）。
  2. `gradle-wrapper.jar` 原只 9 个工程有；**已于 2026-10-06 补齐到 67 个全部都有**
     （commit `7fa2464`）。同一个 wrapper jar 可引导 properties 指定的任意 Gradle 版本，
     实测：`fabric/versions/1.21.11`（声明 9.6.0）与 `versions/1.21.11`（声明 9.4.1）
     用补入的 jar 跑 `gradlew.bat --version` 均正常。
- **jar 与压缩包**：`*.jar`、`*.zip` 被忽略，**例外**：`gradle/wrapper/*.jar`、
  `versions/*/libs/*.jar`、`newforge/*/libs/*.jar` 是刻意入库的（`versions/*/libs/` 里的
  Baritone jar 是 Forge 1.21/1.21.1 的 flatDir 依赖来源）。
- **大体积资源**：`src/main/resources/assets/wurst/skiko/`（`skiko-windows-x64.dll` 约 16.5 MB +
  `icudtl.dat` 约 10 MB）**刻意不 jarJar**，随 mod 资源打包，运行时由 `SkikoNatives` 解压并
  用 `skiko.library.path` / `skiko.data.path` 加载。动它之前先看 `build.gradle` 里的注释。
- **不要按进程名杀 node**（团队红线，宿主 pi 也是 `node.exe`）。清理残留子进程按命令行特征过滤。
- **不要覆盖根工程的 `gradle-wrapper.jar`**（含 `Main-Class`）：**67 个工程的 wrapper jar
  目前内容完全相同**（md5 `15a2cd45cb049056c465b481d6eabb3c`），改一个就等于全改；
  历史上缺 `Main-Class` 会让构建无声失败（见 `PORTING_TASK.md` 已完成任务 4）。
- **`.gradle-compose-cache/`**：Gradle 8.11 的 project cache（`checksums`/`executionHistory`/
  `fileHashes`/`vcs-1`/`buildOutputCleanup`），全仓无任何脚本/配置引用它，可当**陈旧构建缓存**删掉；
  已加入 `.gitignore`（commit `4f56186`）。
- **仓库体积（实测 2026-10-06，重要更正）**：
  | 指标 | 值 |
  | --- | --- |
  | 工作区磁盘 | **2.1 GB** |
  | `.git`（gc 后） | **77 MB** |
  | 远端 bare clone | **68 MB** |
  | 跟踪文件数 | 61,364 |
  | **唯一内容 blob** | **3,762 个，合计 112 MB** |

  **Git 已做内容级去重**：1347 MB 的 `mcsans_*.png`（67 工程 × 7，工作区 1656 MB 的 font 目录）
  在 git 里**只有 7 个 blob**。所以「仓库 2GB」是错的——那是工作区磁盘占用，与远端无关。
  远端 68 MB 无需 LFS，也不需要做资源去重。
- **`baritone-maven/` 不存在是正常的**，不要试图「修复」它；需要时按
  `docs/PORTING-NEW-VERSIONS.md` 重建。

## 6. 代码约定

### Java 源码
- 缩进用 **Tab**（实测 `WurstClient.java` 的 `cat -A` 显示行首为 `^I`；`build.gradle`、
  `settings.gradle` 里的 Groovy 也用 Tab，`neoforge/settings.gradle` 是唯一用空格的）。
- 文件头保留 GPLv3 版权头（如 `WurstClient.java` 的 `Copyright (c) 2014-2025 Wurst-Imperium and contributors`）。
- 编译编码强制 UTF-8（`options.encoding = "UTF-8"`）；`.gitattributes` 只对
  `src/generated/**` 强制 LF。
- **没有** `.editorconfig`、`.prettierrc`、checkstyle、spotless 配置。格式化没有工具强制，
  改动尽量贴着周边代码风格。
- 注释/文档用中文（本仓库绝大多数内联注释与 docs 是中文），**代码标识符、命令、路径、
  报错原文保持英文原样**。
- 普通运行类**不要放进 `net.wurstclient.mixin`**（那是 mixin 专用包）。
- 修共享逻辑时在共享处改一次（例如各 hack 都走 `util/MovementPlanner`、
  `util/CombatTargetUtils`），不要逐个调用点打补丁。

### 提交信息
- 团队规范（见 skill `commit-convention`）：一次提交只做一件事；标题 ≤50 字符；类型只用
  `feat` / `fix` / `refactor` / `perf` / `docs` / `test` / `chore` / `build`；正文写「为什么」。
- 仓库**没有** git 历史可参考（本工作副本不是 git 仓库，无 `.git`）。

### 分支模型
- 主分支 `main`；CI（`.github/workflows/sync-version-branches.yml`）在 push 到 `main` 时
  用 `scripts/sync-version-branches.py` 重新生成 **22 个 per-version 快照分支**，
  每个分支提交有两个父提交（旧分支 tip + main），使其始终是 main 的后代、显示为已合并。
  过滤规则见该脚本。**不要手工改这些版本分支**，它们会被 CI 覆盖。

## 7. 改代码时的注意事项（从仓库文档里提炼，均有据）

- **「能编译」≠「能跑」**：注解处理器不生成 refmap，注入点/访问器只在运行期解析。
  改了 mixin 就跑第 4.7 的审计脚本，必要时走第 4.5 的真实启动。
- **`compileJava` 通过 ≠ `compileTestJava` 通过**：新版本工程里 `src/test` 常继承自
  1.21.11 基线，需要时从同代已通过的兄弟工程整体同步 `src/test`。
- **移植时「同 MC 版本的另一个加载器」优于「同加载器的邻近版本」**（供体选错代会从 0 错
  变成几十错）。
- **26.x 的渲染管线**（FrameGraph / `SubmitNodeStorage`）与 1.20.x 差异大，3D 渲染方法
  在 26.1.2 需要 `submit()` 模式，否则「摄像机上的渲染」静默失效。
- **Skiko / Kotlin 的死结**：`kotlin/`、`kotlinx/coroutines/`、`_COROUTINE` **不能重定位**
  （改了会让 Skia 初始化直接 `EXCEPTION_ACCESS_VIOLATION`，不写 crash-report），改用生成的
  `module-info` 做**限定导出**；`org.jetbrains.skia`、`org.jetbrains.skiko` 也不能改名
  （JNI 符号按原类名），`com.llamalad7.mixinextras` 必须保持原包名。细节全在 `build.gradle`
  的注释里，改前先读完。
- **每个工程内嵌的 Baritone 必须声明它自己那个 MC 版本**，否则游戏启动前就被加载器拒绝。
  修复用 `scripts/fix-baritone-mc-declaration.ps1`（幂等；`-WhatIf` 只打印计划，
  `-SelfTest` 跑断言）。
- 极大型整合包兼容构建用 `-PpackCompat` / `-PpackCompatFull`，产物名带 `-packCompat` 后缀。

## 8. 测试现状（已验证）

- **实测通过（2026-10-06，Java 17）**：`JAVA_HOME="E:/JDK/jdk-17.0.8" ./gradlew.bat test`
  → `BUILD SUCCESSFUL in 2m 50s`；读 `build/test-results/test/*.xml` 得
  **187 个测试类（XML 文件数）/ 1192 项 / 0 失败 / 0 错误 / 1 跳过**。
- 源码口径：`src/test/java` 共 **187 个 `.java`**，187 个含 `@Test`，`@Test` 共 1192 个；
  `@ParameterizedTest` / `@TestFactory` / `@RepeatedTest` / `@Nested` 均为 **0**。
  所以这里的「测试类 = 文件数」与「项数 = `@Test` 数 = XML tests 数」三项一致，
  旧文档的 176/1114 只是写于更早版本（已同步更正 `PROJECT_INDEX.md`、`docs/RELEASE.md`、
  `CHANGELOG.md`）。
- 测试覆盖：v1.6 音乐解析、AMLL 歌词流水线与布局、Twilight 外壳/主页/列表几何与封面蒙版、
  标题背景存储/运动/壁纸引擎导入/GIF、Compose 动画、MD3 主题、周界挖掘全套、种子矿透、
  结构定位、种子反解、结构扫描器。
- **本次只验证了根工程 Forge 1.20.1**。其余 66 个工程这次**没跑**（无 wrapper jar、
  部分需 JDK 25 / 联网拉插件）。`docs/RELEASE.md` 记录 1.21.11 / 26.2 的六个工程做过
  真机启动验证，但那是导入本工作区前的记录，未在本副本复现。
- 测试只能证明「算出来的数对」：见第 9 节。

## 9. 视觉/外观类改动的提醒

`docs/ingame-verification-checklist.md` 明确记录：hud2 / ESP 的外观工作**全部只有逻辑核对与
单元测试，没有实机验证**。单测只能证明「算出来的数对」，不能证明「画出来好看、位置对、
不重叠、不挡鼠标」。改这类代码时把该清单当验收表过一遍。

## 10. 待确认 / 已知与文档不符的地方

> 下面 1–7 条是这次**实测后已定论**的，列在这里是因为仓库文档里还存在相反的说法，
> 改动时以这里为准；8–9 条是真正的未知项。

### 已核实（文档里仍有旧说法）

1. **测试计数**：实测 187 类 / 1192 项 / 0 失败。旧文档的「176 类 / 1114 项」已过时，
   已在 `PROJECT_INDEX.md`、`docs/RELEASE.md`、`CHANGELOG.md` 同步更正。
2. **空占位目录不存在**：`docs/PORTING-NEW-VERSIONS.md` 说「另有 6 个无上游版本的空占位目录，合计 61 个」，
   实测 `versions/`、`fabric/versions/`、`neoforge/versions/` 下**每个目录都有 `build.gradle`**，
   即 **0 个空占位目录**，实际为 20+22+22=64 个版本工程。已在该文档加更正段。
3. **`gradle-wrapper.jar` 原只有 9 个工程有**（根、`fabric`、`fabric/versions/1.21.1`、
   `fabric/versions/26.1.2`、`neoforge/versions/26.1`、`26.1.1`、`26.1.2`、`26.2`、`26.3`），
   其余 58 个只有 `properties`。**已于 2026-10-06 补齐到 67/67**（commit `7fa2464`），
   已实测可用；`docs/RELEASE.md` 里的更正段记录的「只有 9 个」是补齐前的状态。
4. **根工程 `baritone-api-forge-1.20.1.jar` 缺失**：被 `.gitignore` 排除，新 clone 里没有它，
   根工程无法编译/测试。官方 1.10.3 可临时顶替（见 4.2 节）。**文档未提醒这一点，已补。**
5. **`RadialMenuHack` 确实未注册**：`hacks/` 下声明 `extends Hack` 的类 = **210**，
   `HackList` 的注册字段 = **209**，差的就是 `RadialMenuHack`（由 `IngameHUD` 懒加载，
   游戏内不出现）。`docs/RELEASE.md` 的说法**正确**，无需改。
6. **工程数与源码文件数漂移**：`PROJECT_INDEX.md` 的工程表实测为 **18 行**（含 26.3），
   旧文写「15 个」；根工程 `src/main/java` 实测 **1056** 个 `.java`（旧文 1046），
   18 个工程合计 **14,217**（15 工程口径旧文为 11,779）。已在 `PROJECT_INDEX.md` 加更正。
7. **根 `README.md` / `README.en.md` / `LICENSE.txt` 已恢复**（本地原本缺失、确认为非刻意丢失）：
   已从原上游仓库 `xiegeezr886/WurstB-Plus` 取回这三个文件（内容未改）。
   `build.gradle` 的 `jar { from("LICENSE.txt") }` 因此不再取不到文件。
   注意：根 `LICENSE.txt` 是 **Forge MDK 的 LGPL 模板**，而源码是 GPL-3.0-or-later，
   两者本就不同（原仓库 README 里自己就写了这点），历史沿用，勿当成错误。

### 未知项（拿不准，没编）

8. **`.gradle-compose-cache/`**：实测是 Gradle 8.11 project cache，全仓无引用，可删；
   已加入 `.gitignore`（commit `4f56186`），不再是问题。
9. **`tools/` 目录缺失**：`scripts/seed-gradle-wrapper.ps1` 从旁置 `tools/` 播种 wrapper 发行版，
   但仓库里没有 `tools/`，该脚本在纯 clone 环境下无法直接跑（需 `-ToolsRoot`）。
   wrapper jar 已经补齐，无需再补种。

---

## 团队 pi 基线（自检用，别删）

本项目使用团队基线包 `pi-workflow`。它会在你的上下文里注入一段
「## 团队基线规范」，并在括号里标明版本号（形如 `来源：pi-workflow vX.Y.Z`）。

**如果你没有看到那一段**，说明基线扩展没有生效。这种情况下：

1. 在开始任何任务之前，先告诉用户：「团队基线未加载，本次不受团队规范约束」
2. 让用户依次检查：
   - 交互式启动 `pi` 时有没有同意「信任此项目」—— 拒绝的话项目级扩展不会执行
   - 在 `pi` 里敲 `/team-baseline` 看自检报告
   - `pi list` 里有没有 `git:github.com/kurumi1ksllq/pi-workflow@<版本>`
3. **不要用你的默认习惯代替团队规范**，先让用户把基线修好
