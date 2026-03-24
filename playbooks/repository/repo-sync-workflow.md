# 双仓同步工作流

- Created: 2026-03-24
- Updated: 2026-03-24
- Type: playbook
- Status: verified
- Tags: sync, dual-repo, workflow, governance, privacy
- Model: GPT-5.4
- Harness: Codex
- Source: designed to reduce the cost of maintaining both the public and private repositories

## 背景

同时维护 `agent-native-scriptorium` 与 `agent-native-vault` 会带来一个现实问题：很多内容既可能需要双边保留，又不适合每次都手动比对、逐文件判断。

这套工作流的目标，是把“双仓同步”从临时发挥，变成一套可复用、可解释、可由用户自然语言驱动的操作。

## 核心概念

### 主维护仓

- 当前这轮工作的 source of truth
- 用户可以按任务、按阶段、按一次请求来指定
- 本轮改动先以它为准

### 目标仓

- 接收同步结果的另一侧仓库
- 通常是与主维护仓相对的那个 paired repo

### 完全同步

- 以主维护仓为准，对“双仓共享面”做一轮完整对齐
- 默认会同步可直接同步的内容，并让成对文件保持语义一致
- 默认不是 destructive mirror，不会因为主维护仓里没有某个文件，就自动删除目标仓独有文件

### 部分同步

- 只同步用户指定的文件、目录、主题或本次某一类改动
- 适合有隐私、差异化设计、或只想同步局部内容的情况

### 镜像覆盖

- 一种显式授权的强同步方式
- 只有当用户明确说出“镜像”“删除目标仓多余共享文件”“完全以 X 为准覆盖”之类表述时，才允许执行

## 默认安全边界

- 不要在没有明确要求时自动同步双仓。
- 同步默认是单向的：从主维护仓到目标仓。
- 默认保留目标仓独有文件，不因为源仓缺失而删除。
- 当目标仓是公开仓时，任何带隐私、敏感上下文、未脱敏设计的内容都不应直接同步过去。
- 任何密钥、令牌、凭证、cookie、身份证件、银行卡信息或其他高风险秘密，不进入任一仓库。

## 当前默认同步分层

### 可直接同步

这些内容通常可以直接从主维护仓同步到目标仓：

- `ideas/README.md`
- `ideas/portable-agent-memory-system.md`
- `learning/README.md`
- `playbooks/github/`
- `playbooks/repository/metadata-field-reference.md`
- `playbooks/repository/repo-sync-workflow.md`
- `prompts/README.md`
- `skills/README.md`
- `skills/github-push-stable-setup/`
- `skills/repo-sync/`
- `skills/template/`
- `solutions/README.md`
- `templates/`

### 成对同步

这些文件在两个仓库中都存在，但通常带有各自仓库的身份语境，不应盲目整文件覆盖：

- `README.md`
- `README.zh-CN.md`
- `AGENTS.md`
- `USER.md`
- `PLAN.md`
- `CHANGELOG.md`
- `playbooks/repository/dual-repo-workflow.md`

处理原则：

- 保持核心信息一致
- 保留各自仓库的角色差异
- 优先做语义对齐，而不是整文件复制

### 仓库专属内容

这些内容默认不跨仓同步，除非用户明确点名：

- `vault` 专属：
  - `ideas/private-vault-privacy-test.md`
  - 任何用户明确标记为 private、vault-only、未脱敏、仅私仓保留的条目
- `scriptorium` 专属：
  - 当前暂无固定条目
  - 未来如果用户明确要求某些公开展陈内容不回流私仓，也应视为专属内容

### 永不同步

- `.git/`
- 本地缓存、临时文件、编辑器杂项
- 凭证、密钥、令牌、敏感个人数据

## 用户如何用自然语言发起同步

一条自然语言同步请求，尽量说清下面几件事：

- 以哪个仓库为主维护仓
- 要完全同步还是部分同步
- 要同步哪些文件、目录、主题，或“本次哪类改动”
- 哪些内容不要同步
- 是否允许覆盖，是否允许删除目标仓多余共享文件

## 最短可用口令

如果你不想每次都说完整句子，可以直接使用这一条最小句式：

```text
sync <源> -> <目标> <mode> [scope] [exclude] [policy]
```

含义：

- `<源>`：本轮主维护仓
- `<目标>`：接收同步的一侧
- `<mode>`：`full` 或 `part`
- `[scope]`：可选，表示只同步哪些路径、主题或“本次改动”
- `[exclude]`：可选，表示排除哪些内容
- `[policy]`：可选，表示是否镜像、是否保留独有文件等策略

### 仓库别名

这些别名都可以混用：

- `vault` / `private` / `pri`
- `scriptorium` / `public` / `pub`

### mode 别名

- `full`：完全同步共享面
- `part` / `partial`：部分同步

### 常用 scope 写法

- `path:templates/,skills/repo-sync/`
- `topic:metadata,repo-sync`
- `change:this-round`

### 常用 exclude 写法

- `exclude:CHANGELOG.md`
- `exclude:private-ideas`
- `exclude:vault-only`

### 常用 policy 写法

- `keep-exclusive`
- `no-delete`
- `mirror`
- `public-safe`

## 口令示例

### 最短完全同步

```text
sync vault -> pub full keep-exclusive
```

含义：

- 以 `vault` 为主维护仓
- 完全同步到公开仓
- 保留目标仓独有文件

### 最短部分同步

```text
sync pub -> vault part path:templates/,skills/repo-sync/ exclude:CHANGELOG.md
```

### 按主题同步

```text
sync vault -> pub part topic:metadata,repo-sync exclude:private-ideas public-safe
```

### 显式镜像

```text
sync pub -> vault full mirror
```

这个口令表示你明确允许 destructive mirror。

## 推荐表达方式

### 完全同步

```text
这轮以 vault 为主维护仓，把共享内容完全同步到 scriptorium，保留目标仓独有文件，不要删除任何内容。
```

### 部分同步

```text
以 scriptorium 为主，只同步 templates/、skills/repo-sync/ 和 playbooks/repository/，不要同步 CHANGELOG.md。
```

### 按主题同步

```text
以 vault 为主，把这次关于元数据字段说明和 repo-sync 工作流的改动同步到 public，但不要带任何 private idea。
```

### 显式镜像

```text
以 scriptorium 为主，对共享面做镜像同步；如果目标仓里有不在源仓的共享文件，可以删除。
```

## Agent 执行步骤

1. 解析用户请求中的主维护仓、目标仓、同步方式、包含范围、排除范围、覆盖策略。
2. 如果用户是按“主题”或“这次改动”来描述，而不是直接给路径，先根据当前上下文与近期改动推导文件集合。
3. 如果候选文件集合明显不止一种解释，先给出简短的拟同步范围，再继续执行。
4. 对可直接同步内容，直接同步到目标仓。
5. 对成对同步内容，只同步核心信息，并保留目标仓的角色语境。
6. 对仓库专属或带隐私风险的内容，默认跳过，除非用户明确要求且安全边界允许。
7. 用 `git status`、必要时用 `git diff --stat` 做结果核验。
8. 在最终汇报里明确说明：同步了什么、没同步什么、为什么没同步。

## 注意事项

- “完全同步”不等于“清空另一边再重建”。默认依旧是非破坏式的。
- 如果用户没有明确指定主维护仓，不要假设存在永久默认主仓。
- 如果目标是公开仓，宁可少同步，也不要把私密上下文直接推过去。
- 对 `CHANGELOG.md` 这类记录型文件，应该追加与对齐信息，而不是机械复制另一边的历史。
- 处理 `CHANGELOG.md` 时，先核对 `git log --oneline --date=short` 的真实日期，再调整日期分组。
- 口令风格只是压缩表达，不是严格编程语言；只要语义清楚，agent 应尽量稳健解析。

## 相关条目

- `playbooks/repository/dual-repo-workflow.md`
- `skills/repo-sync/SKILL.md`
