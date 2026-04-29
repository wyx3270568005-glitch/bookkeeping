---
metadata: |-
  # Skill Change Log

  本文件记录本地 Skill 的所有变更。日志按日期分组、按时间分小组。

  排序规则：
  - 日期倒序：新的日期在前，旧的日期在后。
  - 同一日期内，具体时间正序：旧的时间在前，新的时间在后。
  - 新增日志时，先找到对应日期；如果日期不存在，则在最上方新建日期分组。
  - 在对应日期内，按时间顺序插入到正确位置。

  每次 Agent 修改 Skill 文件前，默认先创建变更前备份；用户明确要求不备份时跳过。

  ```text
  backup/<变更ID>/<原文件相对路径>
  ```

  示例：

  ```text
  backup/CHG-20260429-143000/bookkeeping/SKILL.md
  ```

  备份保留策略：
  - `backup/` 只保留近 15 天的备份。
  - 每次 Agent 新增备份后，应检查 `backup/` 下的变更 ID 目录。
  - 超过 15 天的备份目录可以删除。
  - 删除超期备份前，应确认对应变更日志仍保留在本文件中。
  - 如果用户明确要求长期保留某个变更 ID 的备份，则不要自动删除该变更 ID 目录。

  当用户输入“退回 <变更ID> 前版本”或“回滚 <变更ID>”时，Agent 必须：
  1. 在本文件中查找对应变更 ID。
  2. 读取该记录的“影响文件（完整路径）”。
  3. 到 `backup/<变更ID>/` 下找到对应文件的变更前备份。
  4. 用备份文件覆盖当前文件。
  5. 追加一条新的回滚日志记录，说明回滚来源和影响文件。

  正文格式：

  ### HH:mm:ss

  - 变更ID：CHG-YYYYMMDD-HHMMSS
  - 影响文件（完整路径）：`T:\bookkeeping\bookkeeping\SKILL.md`
  - 来源：agent / 用户
  - 原因：说明为什么修改
  - 概述：说明本次变更动作

  ## 2026-04-29

  ### 18:31:21

  - 变更ID：CHG-20260429-183121
  - 影响文件（完整路径）：`T:\bookkeeping\README.md`；`T:\bookkeeping\examples\input.md`；`T:\bookkeeping\examples\output.md`；`T:\bookkeeping\templates\record.md`；`T:\bookkeeping\log.md`
  - 来源：用户要求，agent 执行
  - 原因：根据 `SKILL.md` 初始化整个项目目录及关联文件，并遵循用户“不备份”的当前要求。
  - 概述：补齐 README、输入示例、输出示例、单笔记录模板，并将 log.md 保持为隐藏 metadata。
---
