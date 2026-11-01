# 参与 dsh-superagent

感谢你愿意花时间。这份文件是给**没有参与过原始开发**的人看的仓库地图。

开始之前有两件事值得先知道：

- **这是一个研究原型，不是产品。**[README](README.md) 里写明了这一点，这也是提交贡献时该有的
  心态：代码是公开出来供参考和二次开发的，可能还带着 bug，代码风格和工程习惯也可能不统一。
  那些让这个原型更清楚、更适合被真实环境采用的改动，和新增功能一样受欢迎。
- **这个仓库没有 issue 区，也没有讨论区。**所有沟通都发生在 pull request 里。如果哪里看不懂，
  就开一个提问用的 PR，或者开一个把你发现的东西写下来的 PR。

## 先读设计

[`docs/`](docs) 里的九篇编号文档就是规格说明。它们都不长，而且**按顺序递进**：`000` 讲原则，
`001` 给概念命名，后面每一篇只引入前面已经铺垫过的东西。

| 文档 | 讲清楚什么 |
|---|---|
| [000 Philosophy](docs/000-philosophy.md) | 注意力原则，以及它对 agent 和对运维者分别意味着什么 |
| [001 Domain model](docs/001-domain.md) | 概念与每个概念守住的规则，**包括被否决的概念和否决理由** |
| [002 Authorization](docs/002-authorization.md) | 谁能发起工作、工作能包含什么，以及为什么权限要放在宿主侧 |
| [003 Execution](docs/003-execution.md) | 一个被接受的 Ticket 怎么变成 Session 里的实际工作 |
| [004 Prompts](docs/004-prompts.md) | 团队能改什么、文本放在哪、覆盖时不能破坏什么 |
| [005 Panel](docs/005-panel.md) | 那个「只显示需要人处理的事」的面板 |
| [006 Deployment](docs/006-deployment.md) | 手工安装、配置、升级与排障 |
| [007 Architecture](docs/007-architecture.md) | 概念到类、接口和文件的映射 |
| [008 Cookbook](docs/008-cookbook.md) | 怎么扩展它，附一个完整范例 |

动代码之前请先读 `000` 和 `001`。如果你的改动看起来需要引入一个新概念，**先去 `001` 的
「Rejected concepts」表看一眼**——大部分「新概念」的提案都已经被否决过，而且有理由。

## 前置要求

| 工具 | 版本 | 说明 |
|---|---|---|
| Node.js | `^22.19.0 \|\| >=24.0.0` | `engines` 里声明的 |
| pnpm | `11.7.0` | 由 `packageManager` 字段锁定，pnpm 会自动切到这个版本 |
| Git | 任意 | |

运行时还需要装 DeepSeek Harness（`dsh`）的 `^0.2.1-alpha.2`——真正跑起这个 bundle 的就是它。
但**只做类型检查、测试和构建不需要它**，完整运行时环境见 [SETUP.md](SETUP.md)。

## 装起来

```bash
git clone https://github.com/deepseek-ai/dsh-superagent-prototype.git
cd dsh-superagent-prototype
pnpm install
pnpm check
```

`pnpm install` 会拉取这个 bundle 所依赖的 `@deepseek-ai/dsh-*` 包。
`pnpm check` 跑项目所有的检查，见下面的[命令](#命令)一节。

## 命令

| 命令 | 作用 |
|---|---|
| `pnpm check` | 全部：类型检查 → 测试 → 构建。**开 PR 之前必须跑这个。** |
| `pnpm typecheck` | `tsc --noEmit` |
| `pnpm test` | 跑完整的 Vitest 测试套件 |
| `pnpm build` | `tsdown`，产物在 `lib/` |
| `pnpm vitest run tests/assessment.spec.ts` | 跑单个测试文件 |
| `pnpm vitest run tests/assessment.spec.ts -t "keeps nothing for an Eval once it is decided, as for one that timed out"` | 按测试名跑 |

作为参考，在一台 Apple M4 上完整的 `pnpm check` 大约半分钟：测试共 59 个文件、1197 个用例，
跑完约 20 秒，构建几秒。

**这个仓库没有 linter、没有 formatter，也没有 CI workflow。** 没有任何工具会自动检查代码风格，
所以下面[代码风格](#代码风格)一节才是真正的约定，评审的人会照着它看。

## 仓库地图

```
src/          bundle 本体
  app/        Runtime、Engine 装配、passes、outbox、notices、setup
  core/       领域层：source、thread、eval、mandate、ticket、assignment
  executor/   在 Session 里跑一个 Ticket
  infra/      外部边界：store、commands、text、environment
  sources/    Source 实现：GitHub、飞书
  client/     web 面板
docs/         九篇编号文档
prompts/      agent 读到的文本
skills/       agent 按需加载的 Source skill
tests/        Vitest spec，一个概念一个文件
locale/       界面文案（en、zh）
```

`docs/007-architecture.md` 是总索引：它把每个概念映射到实现它的类和文件，
其中的 **Modules** 表和 **Tests** 表是权威的文件清单。

## 哪些东西会被测试拦下来

这些都由 `pnpm test` 强制执行，也是最容易白白损失一轮评审的地方。它们都是 `tests/` 里
真实存在的门禁，不是口头约定。

1. **`src/` 下每个 class 前面都必须紧挨着一段 JSDoc**——`tests/class-docs.spec.ts` 会扫
   `src/`，任何没有 JSDoc 的 class 声明都会让测试失败。
2. **在 `src/` 下新增文件，必须同时在
   [`docs/007-architecture.md`](docs/007-architecture.md) 的 Modules 表里加一行**——
   `tests/docs.spec.ts` 会检查表里没列到的文件，也会检查表里列了但实际不存在的文件。
3. **在 `tests/` 下新增 spec，同样必须在 007 的 Tests 表里加一行**，规则同上。
4. **编号文档必须从 `000` 起连续编号**——加 `009-xxx.md` 没问题，没有 `009` 就写
   `010-xxx.md` 会失败。
5. **编号文档只能链接到编号比它小的编号文档。**这条规则是为了保证这九篇能按顺序读。
6. **`README.md`、`SETUP.md` 和所有编号文档里的本地链接必须都能解析。**
7. **任何文档里都不能留下合并冲突标记。**

如果你加了个模块然后测试报出一串文件，那串文件就是你忘了加的那一行表。

## 代码风格

没有 formatter，所以请**跟随周围的代码**。下面这些是现有代码树里真实在用的约定：

- **两个空格缩进、不写分号、单引号。**
- **相对导入要带 `.ts` 扩展名**——`import { LIMITS } from './limits.ts'`。项目开了
  `allowImportingTsExtensions` 和 `rewriteRelativeImportExtensions`，漏掉扩展名类型检查会挂。
  **这是第一次提 PR 最常见的错误。**
- **纯类型导入用 `import type`**——`verbatimModuleSyntax` 是开着的，用普通 `import` 导入类型会挂。
- **`strict` 和 `noUncheckedIndexedAccess` 都是开的。** 索引数组或对象拿到的是 `T | undefined`，
  请处理它，而不是断言掉。
- **文件开头写一段 JSDoc**，说明这个文件负责什么。导出的函数也要写，需要抛错时加上 `@throws`。
- **领域概念用 PascalCase**——`Policy`、`Mandate`、`Ticket`、`Trigger`、`Subscription`。
- **schema 用 `zod` 并加 `.strict()`。**
- **注释解释「为什么」，不是「是什么」**——代码本身已经说了是什么。

## prompt 文本

[`prompts/`](prompts) 下的文本是 agent 读到的内容，**用英文写**。
`docs/004-prompts.md` 列出了全部覆盖键名，以及覆盖时必须保留的占位符。简单说：

- 团队可以编辑 **Communication**、**Ticket scope**、**Delivery requirements** 和
  **Team skills** 这几节，也可以新增 skill。
- 工具说明、通知和卡片写在代码里，不属于 prompt 设置。
- 改 prompt **不会**改变 gate、Mandate、工具检查或账号包装——那些由代码强制，prompt 放松不了。

如果你改了 prompt，请在 PR 里说明它想改变什么行为、以及你怎么验证的。

## 常见的改动方向

| 你想做的事 | 先读 | 然后 |
|---|---|---|
| 加一种 Source | [008 Cookbook](docs/008-cookbook.md) | 照着 `src/sources/github.ts` 写；Source 是插件，面向团队的 skill 放在 `skills/` |
| 改 agent 说什么 | [004 Prompts](docs/004-prompts.md) | 改 `prompts/`，保留必需的占位符 |
| 改领域规则 | [001 Domain model](docs/001-domain.md) | 文档**和**代码一起改，保持一致 |
| 改工作怎么跑 | [003 Execution](docs/003-execution.md) | `src/executor/` |
| 改面板 | [005 Panel](docs/005-panel.md) | `src/client/` |

## Pull request

- fork 本仓库，从 `main` 切分支，向 `main` 提 PR。
- **一个 PR 只做一件事。** 重命名、行为变更和一个不相关功能的新测试，应该拆成三个 PR。
- **跑完 `pnpm check` 并把结果贴进描述里。**这里没有 CI，评审的人只能看到你贴的结果。
- **commit message 用祈使句**，跟随仓库已有的写法——`feat: add the dsh-superagent research
  prototype`。`feat:`、`fix:`、`docs:`、`refactor:`、`test:`、`chore:` 都在用。
- **解释原因，而不是解释 diff。** 知道原因的评审者，往往会接受跟你不同的实现。
- 如果你不确定某个改动是否被需要，照样开 PR，把不确定的地方写出来。对这个项目来说，
  一个带着清楚问题的开放草稿，比一份私藏的补丁有用得多。

## 边界

- **不要新增依赖。** 运行时依赖只有 `proper-lockfile` 和 `zod`，保持这样。加依赖是一个设计决定，
  不是图方便。
- **不要削弱授权。** gate、Mandate 和工具检查是这个项目最核心的安全属性。如果你认为其中某个
  是错的，请写在 PR 描述里，让评审来决定。
- **不要让 agent 更吵。** 理由见 `docs/000-philosophy.md`：每条消息都在花同事的注意力，
  而一条换不回等价价值的通知就该被删掉。
- **这是原型。** 优先做让它更清楚、更安全、更容易被改造的改动，而不是把它变成一个它本来
  就不该成为的东西。

## 行为准则

参与本项目需遵守 [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)。
