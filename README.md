# bookkeeping

`bookkeeping` 是一个用于自动记账的 Agent Skill。它根据用户的自然语言、结构化字段或包含金额的图片，推断账本记录字段，并写入用户指定的飞书多维表格。它最初由作者在Hermes Agent上创建，Hermes安装在绿联云DX4600的docker上。作者把Hermes绑定到了飞书和微信，作为聊天机器人，通过聊天窗口发送自然语言、结构化字段或图片，Agent基于skill.md命中为记账请求后，都能完成记账。在使用过程中，用户可以随时纠正agent，让skill不断进化，做到了真正的“越用越流畅，越用越懂你”（不是

## 快速安装

**Step1：一段话，让Agent帮你安装Skill的骨架**

把下面这段话直接粘贴给你的Agent：
```
帮我把bookkeep skill安装到本地，你需要先把这个代码仓库完成地下载到本地，保存在当前agent的skill根目录。由于每台设备的依赖和网络环境不同，请你自行选择可用的下载命令和镜像源。注意，先完整克隆整个仓库，不要遗漏任何目录和文件。完成克隆后，把该skill的安装路径和目录树及目录树解释发给用户。然后，阅读skill.md和readme.md，等待用户的下一步操作。https://github.com/andy-JustSayWhen/bookkeeping.git
```

完成安装后，Agent的Skill根目录下，应当有以下文件夹和文件：


```text
bookkeeping/
├── SKILL.md
├── README.md
├── log.md
├── examples/
│   ├── input.md
│   └── output.md
├── templates/
│   └── record.md
└── backup/
```
**Step2：一段话，让Agent帮你安装Skill的依赖**
这个记账skill主要依赖飞书CLI完成账本的读写删等功能，如果你此前已经安装并配置了飞书CLI，那么无需重复安装。跳过此步骤，直接看Step3。

把下面这段话直接粘贴给你的Agent：

```text
帮我安装飞书CLI，严格按照官方文档的四个步骤。完成安装和验证。https://open.feishu.cn/document/no_class/mcp-archive/feishu-cli-installation-guide.md
```

## 必要依赖

- OCR 工具：优先 RapidOCR；如果当前 Agent 环境不适用，可选择其他 OCR 工具替代。
- 飞书 CLI：用于创建、读取和写入飞书多维表格。

## 账本初始化

首次初始化时，Agent 应创建或绑定一个飞书多维表格作为《账本》，并在 [SKILL.md](SKILL.md) 的“账本信息”章节固化必要信息：

- `APP_TOKEN`
- `TABLE_ID`
- 域名
- API 能力
- 外链
- 字段说明

默认字段包括：

| 字段 | 类型 | 说明 |
|---|---|---|
| 名称 | 文本 | 消费或收入对象 |
| 金额 | 数字 | 金额，回复展示时保留两位小数 |
| 类型 | 单选 | `收入` 或 `支出` |
| 账本 | 单选 | 默认包含 `个人`、`工作` |
| 日期 | 日期 | 默认使用记录创建时的北京时间 |

## 使用方式

用户可以在 Agent 聊天窗口中通过以下方式记账：

- 自然语言：`记一笔，东方树叶 5 元`
- 结构化字段：`早餐 20 支出 个人`
- 图片或截图：发送微信、飞书、支付宝、订单、账单等含金额图片
- 修改规则：告诉 Agent 新增、删除或调整记账规则

图片内识别到金额时，Agent 会直接推定为记账请求并进入记账流程，不能追问“要不要记账”。

## 字段推断

当前 Skill 主要推断这些字段：

- `名称`：从用户原话、结构化字段或 OCR 结果中提取消费或收入对象。
- `金额`：提取金额数字，回复展示时保留两位小数。
- `类型`：按语义推断为 `支出` 或 `收入`。
- `账本`：用户指定优先；否则按关键词推断为 `个人`、`工作` 或家庭账本；无法推断时默认 `个人`。
- `日期`：默认使用记录创建时的北京时间；用户明确指定日期时按用户指定。

## 关联文件

- [examples/input.md](examples/input.md)：用户输入示例，帮助 Agent 学习触发表达。
- [examples/output.md](examples/output.md)：标准输出示例，帮助 Agent 保持回复格式稳定。
- [templates/record.md](templates/record.md)：单笔记账回复模板。
- [log.md](log.md)：隐藏在 metadata 中的变更日志说明和记录。
- `backup/`：保存变更前备份，用于按变更 ID 回滚；用户明确要求不备份时跳过。

## 维护规则

Agent 修改 Skill 相关文件时应遵循 [SKILL.md](SKILL.md)：

- 不删除原有内容和结构，除非用户明确要求。
- 默认先在 `backup/<变更ID>/` 创建变更前备份；用户明确说“不备份”时跳过。
- 修改完成后，在根目录 [log.md](log.md) 记录变更；用户明确说“不写日志”时跳过。
- 用户说“退回 <变更ID> 前版本”或“回滚 <变更ID>”时，按备份自动回滚。
