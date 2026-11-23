---
name: okr-progress-sync
description: "将多位组员的 OKR 进展报告汇总回填到 Leader 的 OKR 周报中。自动识别 OKR 对齐关系，按人成块、块内四段（本周进展、发现的问题、风险同步与资源支持、下周进度计划）追加到 Leader 对应 KR 下，不覆盖已有内容，按填写顺序排列。适用于每周 OKR 跟进会、周报汇总、团队 OKR 进度同步、部门 OKR 复盘前的准备等场景。当用户提到「汇总组员OKR进展」「更新到Leader周报」「OKR进展回填」「把组员进展同步到上级」时使用。"
---

# OKR 进展汇总同步

## 核心流程

执行前先确认两个输入：**Leader 是谁**、**哪些组员需要同步**。如果用户没说清楚，先问。

### Step 1：获取 Leader 的 OKR 结构

```bash
# 1.1 查 Leader 用户 ID（已知 open_id 可跳过）
lark-cli contact +search-user --query "Leader姓名" --has-chatted --as user

# 1.2 获取 Leader 当前周期
lark-cli okr +cycle-list --user-id "ou_leader" --as user

# 1.3 获取 Leader 周期详情（所有 O 和 KR）
lark-cli okr +cycle-detail --cycle-id 7000000000000000001 --as user
```

记录下 Leader 的每个 O 的 ID、文本，以及每个 KR 的 ID、文本。

### Step 2：逐个组员判断对齐关系

对每个组员，走下面三条路径之一。**对齐规则详见 [references/alignment-rules.md](references/alignment-rules.md)。**

#### 路径 A：KR 级直接承接（最精确）

Leader 的 KR 文本里 @ 了这位组员。
- 判定方式：遍历 Leader 的每个 KR，检查 ContentBlock 里的 mention 是否包含该组员。
- 结果：组员直接对应这条 KR，不需要软判断。

#### 路径 B：O 级对齐（需要软判断）

组员的 O 通过飞书 OKR 的「添加对齐」功能对齐到了 Leader 的某个 O。
```bash
# 2.1 获取组员的周期和 O/KR 列表
lark-cli okr +cycle-list --user-id "ou_member" --as user
lark-cli okr +cycle-detail --cycle-id 7000000000000000002 --as user

# 2.2 查询组员的对齐关系（对齐了谁）
lark-cli okr objective.alignments list \
  --objective-id "组员的O_ID" \
  --align-type "aligning" --page-all --as user
```
- 查到对齐到 Leader 的某个 O → 进入软判断：对比组员各 KR 的内容和 Leader 该 O 下的 KR，判断组员每条进展对应 Leader 的哪条 KR。
- 软判断置信度高就直接执行；**有疑问就停下来问用户**，不要瞎猜。

#### 路径 C：无对齐关系（跳过）

既没有 KR @ 承接，也没有 O 级对齐 → **跳过该组员**，在最终汇总里列出来告诉用户"XX 没有对齐关系，已跳过"。

### Step 3：读取组员最新进展

```bash
# 读取组员某条 KR 下的最新进展记录
lark-cli okr +progress-list \
  --target-id "组员KR_ID" \
  --target-type key_result \
  --page-size 5 --as user
```

取最新一条进展记录的内容。如果组员在自己的 O 下写进展而非 KR 下，改用 `--target-type objective`。

### Step 4：组装回填内容

内容格式规则详见 [references/content-format.md](references/content-format.md)。

核心规则：
- **按人成块**：每个组员在 Leader 进展里是一个独立块，块内按四段排列；不要把多个组员的进展混在同一行、也不要按"所有人进展 → 所有人问题"的方式跨人合并。
- 块内四段对应：本周进展 / 发现的问题 / 风险同步与资源支持 / 下周进度计划。
- 每条组员进展前面加 `@姓名`（用 mention 格式）。
- **追加模式**：在 Leader 已有进展后面新增一条记录，绝不修改或覆盖已有内容；同一条 KR 下多个组员各占一块，按处理先后顺序排列。

### Step 5：写入 Leader 对应 KR

每个组员创建一条独立进展记录（即一个人一块），内容为 `@姓名 + 四段`：

```bash
lark-cli okr +progress-create \
  --content '{"text":"@王颖颖\n本周进展：...\n发现的问题：...\n风险与资源：...\n下周计划：...","mention":["ou_wangyingying"]}' \
  --target-id "Leader_KR_ID" \
  --target-type key_result \
  --source-title "OKR周报汇总" \
  --as user
```

同一条 KR 下有多个组员时，逐个调用上面的命令，后写入的块接在前一条后面。

### Step 6：输出汇总报告

完成后给用户一个清单：
- ✅ 已同步：姓名 → 对应 Leader KR 编号
- ⚠️ 待确认：姓名 → 软判断有疑问，等用户确认后再写入
- ⏭️ 已跳过：姓名 → 无对齐关系

## 关键原则

1. **按人成块**：Leader 进展里一个组员一个块，块内四段齐全；块与块之间不混排、不按段落跨人合并。
2. **不覆盖**：永远用 `+progress-create` 新增，不用 `+progress-update` 修改已有记录；新块接在已有内容后面，按填写顺序排列。
3. **不瞎猜**：对齐关系、分段归属或 KR 对应置信度不足时，列出判断理由问用户，用户确认后再写。
4. **先读后写**：写入前先看一下 Leader 这条 KR 下已有的进展，确认这个组员这周还没写过（避免重复）。
5. ** mention 用 open_id**：@ 人时用 `mention` 数组传 open_id，不要只在文本里打名字。

## 参考文档

- [对齐判断规则](references/alignment-rules.md) — 三种对齐路径的判定细节和软判断指引
- [内容格式规范](references/content-format.md) — 四段模板格式、@人写法、内容润色原则
- [完整场景案例](references/example-scenario.md) — 1 Leader + 4 组员的端到端填写示例（含对齐判定、回填结果、待确认分支）
