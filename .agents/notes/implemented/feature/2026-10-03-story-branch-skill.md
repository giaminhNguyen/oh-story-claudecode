# Agent Note: 新增第 14 个 skill `story-branch`（分支续写，只备料不写正文）

Status: implemented

## Problem

作者拿到一本已完成或已连载的书后，只有两条路：接着写同一本（`story-long-write`），或把旧稿反向重建成自己的工程（`story-import`）。两者都不回答「我想让这本书里的某个人重活一次」「换成反派视角重讲」「如果那一章他没死」这类需求——而这恰恰是网文读者和作者最常提的续写方向。

现有工具箱缺的正是这一段：**从一部既有作品抽出正典（人物、关系、时间线、秘密、谁在第几章知道什么），据此提出能独立成书的分支，再把分支的输入准备成写作工程能直接吃下的结构化材料。**

同时，本仓是 `zenstory-ai/oh-story-claudecode` 的 fork，安装与升级入口仍指向上游，作者按 README 装到的是上游而不是本仓，`story` 的「检查更新」也去问上游的 release。

## Decision

- 新增 `skills/story-branch/`，是本仓第 14 个 skill，四个时刻对应四个子命令：
  - `analyze` 读源作品，落 `分支库/{源ID}/正典.md`（人物、关系、时间线、事件、秘密、知识边界、缺失事实）。
  - `explore` 采用正典创意变异引擎（Creative Mutation Engine），从角色立场、身份意识、记忆知识、目标同盟、时间因果、世界规则等多轴变异生成 5–8 个高内在逻辑候选，拒绝固定套路分类与默认套路执行，强调知识衰减与角色自主性，落 `分支库/{源ID}/分支提案.md`。
  - `create` 在作者拍板后落分支简报 `分支库/{源ID}/分支/{源ID}-B0X.md`（主角、前世与正典上下文、知识边界、分歧点、变更事实、约束）。
  - `handoff` 把简报播种成 `{书名}/` 写作工程的 `设定/分支设定.md` 与 `.story/work/分支交接.md`，然后交给既有写作流程。
- **skill 只备料。** 入口写死边界：正文、细纲、卷纲、字数口径、去味与审查方法论一律由 `story-long-write` / `story-short-write` 自带，`story-branch` 不复述、不替代、不改写它们。`handoff` 的产物是写作工程能直接读取的设定材料，不是正文，也不是新流程。
- 落盘目录用 `分支库/`，与既有 `拆文库/` 同级同风格；ID 规则 `{源ID}`=`SRC-001`、`{分支ID}`=`SRC-001-B01`，与既有 `F0xx` 伏笔号、`RAW/REUSE` 批次号不同命名空间，不会撞。
- `story-branch` 是 solo skill：不 spawn agent，因此不动 `SPAWN_CAPABLE_SKILLS`、agent 模板、权限矩阵，也不进 `check-zcode-adapter.sh` 的「必须写 ZCode solo 降级」名单。
- 入口同时给两种写法：`/story branch analyze …`（`story` 路由按子意图分发，参照既有 `/story dashboard`）与真实 slash command `/story-branch …`。后者是必需的：OpenCode/ZCode 的适配守卫断言 command 名与 skill 名集合相等，少一个文件整条 CI 就红。
- `skills/story-setup/references/templates/CLAUDE.md.tmpl` 加路由行后跑 `scripts/sync-opencode.py` 生成 `opencode/AGENTS.md.tmpl`；手改生成物会被下一次 sync 覆盖并丢掉 `story-branch`。
- 安装与升级命令、`story` 的版本检查源、`session-start.sh` 的更新提示统一改指 `giaminhNguyen/oh-story-claudecode`；README 的 Stars / Release 徽章与外链保留指向上游，并新增一段明确的 fork 出处声明（上游出处要看得见）。
- 版本 0.8.4 → **0.9.0**：新增一整个 skill 与四个子命令是 feature，不是 patch。
- `setup_skill_version` 1.3.2 → 1.3.3：`session-start.sh` 的更新提示文案变了，这是被部署出去的产物。`agents_version` 保持 34——agent 模板一个字节都没动。

## Alternatives considered

- **让 `story-branch` 自己写大纲和正文**——最强理由是作者少切一次会话，分支直接从正典长出正文。否：这等于在 `story-long-write` 之外复制一套写作方法论，两边会各自漂移；而且 `story-long-write` 的门禁（缺细纲拦住写正文）、追踪事务与字数口径都不会被继承，分支书会变成没有状态卡、没有追踪的孤儿工程。备料 + 交接是唯一不破坏既有工作流契约的形状。
- **把四个子命令塞进 `story-import`**——最强理由是「读旧书 → 得到可写的东西」本来就是 import 的语义延伸，且不用新增 skill、不用改所有 skill 计数与守卫。否：`story-import` 的交付物是「重建原书」，与「造一本新书」是相反方向；塞进去会让 import 的时刻表和产物目录语义混乱，且 import 已有的篇幅分流、正典/对标边界规则会被新语义污染。`story-import` 只加一行跳转。
- **按固定 5 个套路家族分类生成候选**——最强理由是结构清晰、易于模板化。否：固化分类会导致套路化输出（如单纯的“恶毒女配重生复仇”），抹杀真正有惊奇感的高潜力故事。换用创意变异引擎，以打破/重组正典假设为起点，强调多轴变异与逻辑自洽，产生出乎意料但合乎逻辑的新因果。
- **不落盘、全部留在对话里**——最强理由是省掉 `分支库/`，也不必管 ID 与续跑。否：换上下文就没了，分支提案和简报无法复核，四步链条断在中间；这正是 `story-import` 当初落 `导入记录.md` 的同一个理由。
- **把 fork 的 README 徽章也指向 fork**——最强理由是读者点进去看到的是自己装的东西。否：Stars / Release 数字属于上游仓库，指过去等于把上游的社区资产算作本 fork 的；装的是哪一份由安装命令和 fork 声明说清楚就够了。
- **升 0.8.5 而不是 0.9.0**——最强理由是纯增量、无破坏性变更，语义上更保守。否：新增一整个 skill 与四个新子命令是面向用户的能力扩张，CHANGELOG 与 Release notes 需要让作者一眼看出这一版多了什么。

## Consequences

收益是作者手上多了一条从「读过的书」到「能开的新书」的路径，且这条路径完全落在既有写作流程下游，不新增写作方法论、不新增 agent、不新增运行时依赖。代价是：13 → 14 的 skill 计数散落在守卫脚本、插件清单、部署文档与七份 AGENTS 模板里，每一处都要同步改，漏一处对应的那条 CI 就红；`story-branch` 自己也要占 `doc-budget.json` 的时刻预算，将来加规则同样受 35K 硬上限约束。`.clawhub/publish.json` 的 `packageDigest` 需要发布时用固定中央 `clawhub_release.py lock` 重算，本地无该命令。
