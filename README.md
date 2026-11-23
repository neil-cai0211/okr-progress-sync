# OKR 进展汇总同步

将多位组员的飞书 OKR 进展整理后，追加到 Leader 对应 KR 下。每位组员独立成块，保留四段结构，便于每周跟进和复盘。

## 能做什么

- 根据 Leader KR 中的 @ 提及，或组员 O 的对齐关系，确定回填位置。
- 默认读取组员最新一条进展；需要合并历史记录时，由用户明确范围。
- 按「本周进展、发现的问题、风险同步与资源支持、下周进度计划」整理内容。
- 写入前检查已有记录，避免重复；通过新增记录追加内容。
- 完成后列出已同步、待确认和已跳过的人员。

## 使用条件

这是供 AI 助手读取并执行的 skill。使用环境需要：

1. 能加载本仓库的 [SKILL.md](SKILL.md) 及其参考文档。
2. 已安装并登录 `lark-cli`，可按用户身份调用通讯录与 OKR 相关命令。
3. 当前账号有权读取相关人员的 OKR、对齐关系和进展，并向目标 KR 新增进展。
4. 用户已明确需要同步的 Leader 和组员。

具体命令见 [SKILL.md](SKILL.md)。命令兼容性、账号权限和实际写入效果需在使用环境中核验；仓库中的案例用于说明流程。

## 获取与使用

克隆仓库：

```bash
git clone https://github.com/neil-cai0211/okr-progress-sync.git
```

将 `okr-progress-sync` 文件夹放到所用 AI 工具约定的 skill 目录，保持 `SKILL.md` 与 `references/` 的相对位置不变，再按该工具的方式加载。

加载后，可以这样提出请求：

> 用 okr-progress-sync，把组员甲、组员乙、组员丙的最新 OKR 进展同步到 Leader 张主管对应的 KR 下。有疑问的对应关系先列出来让我确认。

以上姓名为占位示例，使用时替换为实际人员。如果需要指定周期或合并历史进展，请同时说明。

## 回填示例

同一条 Leader KR 下，每位组员各新增一条进展记录，内容结构如下：

```text
@组员甲
本周进展：……
发现的问题：……
风险同步与资源支持：……
下周进度计划：……
```

同批组员按处理顺序逐个写入，每人保留完整四段。内容格式细节见 [内容格式规范](references/content-format.md)。

## 对齐与写入规则

| 情况 | 处理方式 |
| --- | --- |
| Leader 的 KR 中直接 @ 了组员 | 按 skill 规则对应到该 KR |
| 组员的 O 对齐了 Leader 的 O | 比较双方 KR 内容；对应明确时回填，有疑问时请用户确认 |
| 没有上述对齐关系 | 跳过，并在结果中说明 |
| 本周已有该组员的记录 | 避免重复写入 |
| 事实、数字或内容归属不明确 | 保留原意，不补造；影响回填位置时先确认 |

写入使用 `+progress-create` 新增记录，保留 Leader 已有内容。@ 提及需同时传入对应人员的 `open_id`。

## 文件导航

| 文件 | 内容 |
| --- | --- |
| [SKILL.md](SKILL.md) | 技能入口、执行步骤和命令示例 |
| [references/alignment-rules.md](references/alignment-rules.md) | 三种对齐路径与边界情况 |
| [references/content-format.md](references/content-format.md) | 四段模板、@ 提及和写入前检查 |
| [references/example-scenario.md](references/example-scenario.md) | 1 位 Leader、4 位组员的完整虚构案例 |
