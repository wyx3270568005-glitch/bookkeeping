---
name: bookkeeping
description: >-
  飞书账本记账 skill。触发条件：用户发送记账请求（结构化字段或自然语言）或发送带金额的图片。
  **强制规则：检测到图片中有金额 → 立即执行记账流程，不询问用户「要不要记账」。**
  自动补全项目归属和标签，立即写入飞书表格，不等确认。
  每晚23:00定时抄送当日记录供用户审阅。
triggers:
  - 记账
  - 记一笔
  - 花了
  - 收到
  - 刚才.*花
  - 刚才.*付
  - 刚才.*充
  - 用户发送截图且无其他文字说明时，先用OCR或视觉理解读取图片内容，若发现有金额，优先推定为记账请求，直接记录到指定账本内；若用户说「不是记账」则取消
  - 发了一张图片
  - 图片.*记账
  - 截图.*记账
  - 图片.*多少钱
  - 飞书.*图片
  - 微信.*图片
tags: [feishu, bitable, bookkeeping, productivity]
---

# 飞书账本记账 Skill

## 飞书表格信息，即《账本》的信息
- APP_TOKEN: WK85b97Eeaq5wesyAiMcfzl2nId
- TABLE_ID: tblo93G5FW2nVzvr
- 域名: feishu (open.feishu.cn)
- API: Bitable 记录 CRUD（读写删均可用）

## Auth / API 优先级

**lark-cli（OAuth 用户身份）是所有飞书操作的第一优先级**，已安装并完成授权。
**直接 API（bot token / tenant_access_token）是 fallback**，仅在 lark-cli 不可用时使用。

> ⚠️ **配置状态**：OAuth 授权已完成。
> ⚠️ **环境注意**：
> - `lark-cli` 不在系统 PATH 中，必须用**完整路径**：`/opt/data/home/.npm-global/bin/lark-cli`
> - 环境中无 `curl` 命令，Python urllib 是可靠的 fallback（已实现，见下方"方式 B"）
> - 执行 lark-cli 时必须加 `HOME=/opt/data/home` 前缀。

**常量：**
- `--base-token WK85b97Eeaq5wesyAiMcfzl2nId`
- `--table-id tblo93G5FW2nVzvr`（表名 `数据表`）




## 字段说明

| 字段 | field_id | 类型 | 说明 |
|------|----------|------|------|
| 名称 | fldVUitYU8 | 文本 | |
| 金额 | fldFcNErot | 数字 | |
| 类型 | fldvhbVlTa | 单选 | 支出 / 收入 |
| 账本 | fldX0xhEgN | 单选 | 个人 / 工作 / 家庭01 / 家庭02 |
| 项目归属 | fldoshLsbM | 单选 | 按规则补全 |
| 标签 | fldMm9MprN | 单选 | 按规则补全 |
| 结算情况 | fldKQSgcCZ | 单选 | 按规则补全（见下方结算规则） |
| 支出日期（手动） | fldIz5GNSp | 日期 | 不填，系统自动记录 |

### 账本推断规则（截图记账时）

收到截图记账请求时，根据名称和标签关键词推断账本：

**推断为「工作账本」的关键词：**
- 投流、播放、推广、互动、抽奖
- 小米微博
- 推广费、制作费、制作成本
- 采买（xx采买）

**推断为「个人账本」的关键词：**
- 食物相关：午饭、晚饭、早餐、早饭、夜宵、咖啡、奶茶、零食、水、饮用水、面包、面条、食材、外卖、食堂、蛋糕、酸奶、烤肠、月饼、绝味、牛排、水果、烧烤、小吃、凉粉、蜜雪冰城、瑞幸、茶叶等
- 日用、数码、医疗、娱乐、学习、礼金、住宿、通讯、家装、家电、服饰（衣服鞋子等）、纳税、保险、捐赠、家具

**兜底：**
- 无法推断 → 默认「个人账本」
- 用户可随时指定账本，覆盖自动推断

### 图片记账流程（⚠️ 强制执行，不得询问用户）

**Step 1：图片识别**
- 只要收到图片，必须先调用RapidOCR进行内容识别，提取文本；
- 若图片中有金额，直接推定为记账请求，直接进入 Step 2；
- 若图片当中没有金额，转为调用大模型的视觉理解能力进行处理，不视为记账请求。根据上下文推断用户意图； 


**Step 2：发现金额 → 自动推定为记账请求**
- 检测到金额后，**立即执行 Step 3-5**，**不询问用户**
- 提取名称、金额，推断类型（支出/收入）
- 禁止在此步骤后回复用户"要不要记账"或"确认一下"等确认性话语

**Step 3：推断账本**
- 根据名称关键词推断（规则见"账本推断规则"）

**Step 4：补全项目归属、标签和结算情况**
- 按本 skill 规则补全

### 结算情况补全规则

**原则：** 结算情况用于跟进商单项目的外包制作费是否已支付给外包人员，以及该商单项目的总制作费是否已结算给接单者（andy，数码博主）。

**规则：**
- **项目归属为「无」** → 结算情况填「**是**」（非项目类个人/工作支出，无需跟进结算）
- **项目归属有特定值**（如「妙界R9」「小米微博2026年04月」）→ 结算情况填「**否**」（属于商单项目，需跟进结算状态）

**写入时统一补「否」，等用户手动改为「是」确认已结算。**

**Step 5：立即写入飞书表格**
- 通过飞书CLI能力写入，不等确认
- 写入成功后用代码块表格回复用户

**例外**：用户明确说「不是记账」→ 取消写入

## 记账流程（重要）

1. **立即写入表格**，不等用户确认
2. 写入后用代码块居中对齐表格回复用户，格式如下：

单条记录：
```
       名称            金额     类型    账本    项目归属     标签  
─────────────────────────────────────────────────────────────────
     东方树叶          5.00     支出    个人       无        食物  
```

多条记录：
```
       名称            金额     类型    账本         项目归属          标签  
─────────────────────────────────────────────────────────────────────────────
     东方树叶          5.00     支出    个人            无             食物  
       打车           12.50     支出    个人            无             交通  
    微博投流         300.00     支出    工作       微博2026年04月      推广费  
```

对齐规则：用 CJK 双宽度计算（中文字符占2列，ASCII占1列），每列居中对齐，列间距2空格，表头与数据之间用 ─ 分隔线。

3. 结尾带上账本链接（单独成段，飞书会自动解析生成预览卡片）：
   查看账本：https://lxxtrhuie3n.feishu.cn/base/WK85b97Eeaq5wesyAiMcfzl2nId?table=tblo93G5FW2nVzvr

4. **每晚23:00定时任务**抄送当日全部记录发送到飞书会话给用户审阅（由 cron job 处理），同样使用上述代码块表格格式。优先通过飞书CLI实现。

### ⚠️ 飞书消息格式注意事项
- **不要用 markdown 表格**（`|---|---|`）→ 飞书文本消息不支持，会乱码
- **不要用空格对齐** → 飞书用非等宽字体，空格对齐会错位
- **用反引号代码块**（` ``` `）→ 等宽字体，对齐可靠

## 项目归属补全规则

### 规则 1：个人/家庭账本 → 填「无」
若账本为「个人」「家庭01」「家庭02」，项目归属一律填「无」。

### 规则 2：工作账本 → 按名称解析
仅当账本为「工作」时，执行以下规则：

**2a. 小米微博系列**
名称含「小米微博」→ 项目归属填「小米微博xx年xx月」
- xx年xx月 = 记录创建时的实际年月
- 例：2026年4月创建 → 「小米微博2026年04月」

**2b. 后缀剥离规则（投流/播放/推广/互动/抽奖）**
若名称匹配以下任一格式，剥离后缀，取前缀作为项目归属：
- `xx-投流` / `xx投流` → 项目归属 = `xx`
- `xx-播放` / `xx播放` → 项目归属 = `xx`
- `xx-推广` / `xx推广` → 项目归属 = `xx`
- `xx-互动` / `xx互动` → 项目归属 = `xx`
- `xx-抽奖` / `xx抽奖` → 项目归属 = `xx`

示例：
| 名称 | 项目归属 |
|------|---------|
| 酷态科6号Ultra-投流 | 酷态科6号Ultra |
| QCY播放 | QCY |
| 妙界投流 | 妙界 |
| 英伟达互动 | 英伟达 |
| 小米微博抽奖 | （走规则2a）小米微博2026年xx月 |


**2c. 兜底规则**
不匹配以上规则的工作账本记录 → 项目归属填「无」，等用户手动指定。

## 标签补全规则

### 原则
- 只能从已有标签中选择，不得新建标签（除非用户明确要求）
- 通过名称关键词匹配推断
- 已有标签（26个）：食物、交通、推广费、制作费、制作成本、日用、数码、医疗、娱乐、学习、礼金、住宿、其他、通讯、捐赠、服饰、纳税、会员费、家装、家电、抽佣、家具、保险-人身、保险-财产、保险-综合

### 关键词 → 标签映射表

| 标签 | 匹配关键词（名称中含以下任一即匹配） |
|------|--------------------------------------|
| 食物 | 午饭、晚饭、早餐、早饭、夜宵、咖啡、奶茶、零食、水、饮用水、面包、面条、鸡蛋、食材、外卖、食堂、蛋糕、酸奶、蓝莓、兔腿、烤肠、月饼、绝味、牛排、水果、西瓜、烧烤、小吃、凉粉、泡芙、仙草、芋圆、蜜雪冰城、瑞幸、茶叶、茶、乌龙茶、红茶、菊花、汽水、苏打水、雪糕、粉、肠旺面、肉肠、洋芋粑、坚果、鸡肉、料酒、橄榄油、卤牛肉、蔬菜、干酵母、全麦粉、果干、奶粉、胡萝卜、甜点、披萨 |
| 交通 | 打车、公交、滴滴、运费、顺风车、高铁、出租车、客车、地铁、车马费、停车费、租车、滑板车、摩托、跑腿、快递、顺丰、同城 |
| 推广费 | 投流、播放、互动、点赞、推广、dou+、千川、抽奖（仅工作账本） |
| 制作费 | 制作费、外包制作、稿费、视频制作费、配音、拍摄道具、光轴 |
| 制作成本 | 制作成本（搭配推广费/制作费使用，描述项目支出） |
| 娱乐 | 三角洲、按摩、陪玩、网费、游戏充值、足疗、酒吧、猫咖、网吧、弹力带、健身器材、瑜伽球、游戏软件购买 |
| 日用 | 湿厕纸、纸巾、垃圾袋、鼠标垫、蛋糕原料、牙膏、牙刷、毛巾、挂钩、衣架、唇膏、护手霜、护肤品、帽子、手套、纸巾盒、洗碗粉、洗衣液、爆炸盐、保鲜盒、分装盒、厨房纸、地垫、桌布、卷纸、双面胶、擦镜布、吸油纸、驱蚊喷雾、湿巾、马桶圈、无线开关、指甲刀、眼镜护理液、洗浴用品、衣物清新剂、漏勺、燃气灶电池、养生壶、扫吧、洗菜篮、除臭剂、家居用品、雨伞、鞋带、熨烫机、手机膜、清洁剂、浴室用品、加湿器滤芯、擦镜纸、花洒维修、空调清洗、裤架、袋子、怀炉保护套、煤油、鼠标充电器、置物架配件、喷油壶、磁吸灯条、纽扣、微波炉加热盘、夹子混装、毛毯和水、衣柜、地毯、小刮刀、日用、理发、米家射频遥控器、香氛机、米家 |
| 数码 | 键盘、鼠标、耳机、手机壳、充电宝、数据线、充电器、手机膜、手机支架、路由器、内存卡、固态硬盘、机械硬盘、体脂秤、音箱、小米手环、iPhone、散热器风扇、酷态科数据线、小米充电宝、AirTag、xiaomitag、vivox300、vivo无线充、vivo耳机、手机配件、诱骗器、凯夫拉手机壳 |
| 抽佣 | 返点、带货佣金、抽佣 |
| 服饰 | 衣服、服饰、羊毛衫、冲锋衣、牛仔裤、夹克外套、鞋子、袜子、皮带、护发膏、唇膏、固体香膏、手套、外套、高领内衣、表带、隐形眼镜、眼镜、香水、护肤品、护手霜 |
| 会员费 | VIP、88VIP、会员、订阅、智谱VIP、剪映VIP、即梦VIP、minimaxVIP、gminiproVIP |
| 医疗 | 药品、解酒药、皮肤药、维生素、门诊、检查费、拔牙、驾考、药物异维A酸、洗鼻器、医疗 |
| 学习 | 考试用品、驾校、学费、报名 |
| 礼金 | 赠予、红包、礼金、投资分红、乔丹礼金、礼物、鲜花 |
| 住宿 | 酒店、民宿 |
| 通讯 | 话费、充话费、网费 |
| 家装 | 洞洞板、装修、空气净化器滤芯、洗衣机清洗 |
| 家电 | 家电 |
| 捐赠 | 捐赠、赠与 |
| 保险-人身 | 保险相关人身险（由用户手动选择） |
| 保险-财产 | 保险相关财产险（由用户手动选择） |
| 保险-综合 | 保险相关综合险（由用户手动选择） |
| 纳税 | 纳税 |
| 兜底 | 以上均不匹配 → 「其他」 |

### 特殊说明
- 「#」开头的名称（如" #零食"" #调料"）→ 去掉「#」后按关键词匹配
- 名称为 None 或无法判断 → 标签「其他」

## 写入 API（推荐 lark-cli，速度更快）

### ⚠️ 关键实现细节（本次调试发现）

**1. `record-list` 返回扁平数组，字段顺序固定：**
```
索引:  0=名称, 1=创建日期, 2=最后更新时间, 3=金额, 4=金额02,
       5=支出日期（手动）, 6=结算情况, 7=支出日期（自动）,
       8=标签, 9=项目归属, 10=类型, 11=账本, 12=月份, 13=年份
```
**今日记录在 offset ≈ 4200 附近**（新记录按创建时间升序，越新越靠后）。

**2. 筛选今日记录用 r[1]（创建日期），不要用 r[5]：**
```python
today = datetime.now().strftime('%Y-%m-%d')
if str(r[1])[:10] == today:  # ✅ 用创建日期
    # 不要用 r[5]（支出日期手动），大量记录是 None
```

**3. 发消息用 `--text`，不要用 `--content`：**
```shell
# ✅ --text 直接传纯文本（可靠）
HOME=/opt/data/home lark-cli im +messages-send --chat-id <id> --text '<内容>'

# ❌ --content 要求 JSON 格式，复杂内容必报错
HOME=/opt/data/home lark-cli im +messages-send --chat-id <id> --content '{"text":"..."}'
```

**4. `lark-cli` 必须用完整路径：**
```shell
/opt/data/home/.npm-global/bin/lark-cli   # ✅
lark-cli                                    # ❌ 不在 PATH 中
```
执行时必须加 `HOME=/opt/data/home` 前缀。

**5. Cron ticker 不稳定——Gateway 重启会导致定时任务跳过：**
日志路径：`/opt/data/logs/gateway.log`
- ticker 线程会随 Gateway 重启而停止/重启
- 若 23:00 UTC+8（15:00 UTC）时 ticker 恰好停止，任务将跳过
- 手动触发 `cronjob action=run` 可补发，但不保证准时
- 任务输出在 `/opt/data/cron/output/<job_id>/`

**6. 获取飞书 chat_id：**
```python
import json
with open('/opt/data/channel_directory.json') as f:
    data = json.load(f)
feishu_channels = data['platforms']['feishu']
chat_id = feishu_channels[0]['id']  # 用户 DM open_id
```

---

**方式 A：lark-cli（推荐，写入最快）**
```shell
HOME=/opt/data/home /opt/data/home/.npm-global/bin/lark-cli base +record-upsert \
  --base-token WK85b97Eeaq5wesyAiMcfzl2nId \
  --table-id 数据表 \
  --json '{"名称":"xxx","金额":xx,"类型":"支出","账本":"个人","项目归属":"无","标签":"其他","结算情况":"是"}'
```

**方式 B：直接 API（curl / bot token，fallback）**
当 lark-cli 不可用时，使用环境变量中的 `FEISHU_APP_ID` + `FEISHU_APP_SECRET` 获取 tenant_access_token，再调 Bitable API：

```python
import urllib.request, json, os

# Step 1: get tenant_access_token
app_id = os.environ['FEISHU_APP_ID']
app_secret = os.environ['FEISHU_APP_SECRET']
req = urllib.request.Request(
    'https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal',
    data=json.dumps({'app_id': app_id, 'app_secret': app_secret}).encode(),
    headers={'Content-Type': 'application/json'}
)
resp = urllib.request.urlopen(req, timeout=10)
token = json.loads(resp.read())['tenant_access_token']

# Step 2: write record
payload = {
    'fields': {
        '名称': 'xxx', '金额': 0.01, '类型': '支出',
        '账本': '个人', '项目归属': '无', '标签': '其他', '结算情况': '是'
    }
}
req2 = urllib.request.Request(
    'https://open.feishu.cn/open-apis/bitable/v1/apps/WK85b97Eeaq5wesyAiMcfzl2nId/tables/tblo93G5FW2nVzvr/records',
    data=json.dumps(payload).encode(),
    headers={'Content-Type': 'application/json', 'Authorization': f'Bearer {token}'},
    method='POST'
)
result = json.loads(urllib.request.urlopen(req2, timeout=10).read())
print(result['data']['record']['record_id'])
```

环境变量已就绪，无需额外配置。

## 修改 API（lark-cli）
```shell
# 先用 record-list 查出 record_id，再用 upsert 更新
HOME=/opt/data/home lark-cli base +record-upsert --base-token WK85b97Eeaq5wesyAiMcfzl2nId --table-id 数据表 --record-id <record_id> --json '{"名称":"xxx"}'
```

## 删除 API（lark-cli）
```shell
HOME=/opt/data/home lark-cli base +record-delete --base-token WK85b97Eeaq5wesyAiMcfzl2nId --table-id 数据表 --record-id <record_id> --yes
```

> 删除前建议先用 record-list 确认 record_id 正确。

## 每晚23:00抄送（cron job）

定时任务：查询当日所有记录，格式化后发送到飞书会话给用户审阅。用户可指出错误，实时修改。涉及飞书相关的功能，优先使用 lark-cli 实现。

### 定时任务执行步骤（含关键实现细节）

**Step 1：查询全量记录（需大 limit + 翻页）**

Bitable `record-list` API 按创建时间**升序**返回记录，新记录在后面。
今日（2026-04-28）新记录出现在 offset ≈ 4200 的位置，建议直接用 `--limit 500 --offset 4200` 一次拉取。

```shell
HOME=/opt/data/home /opt/data/home/.npm-global/bin/lark-cli base +record-list \
  --base-token WK85b97Eeaq5wesyAiMcfzl2nId \
  --table-id 数据表 \
  --limit 500 --offset 4200
```

**⚠️ 重要：`record-list` 返回的是扁平数组，字段顺序固定：**
```
索引:  0=名称, 1=创建日期, 2=最后更新时间, 3=金额, 4=金额02,
       5=支出日期（手动）, 6=结算情况, 7=支出日期（自动）,
       8=标签, 9=项目归属, 10=类型, 11=账本, 12=月份, 13=年份
```
筛选今日记录用 `r[1]`（创建日期）判断，格式为 `"YYYY-MM-DD HH:MM:SS"`。

**Step 2：获取飞书 chat_id**

从 `/opt/data/channel_directory.json` 读取当前用户的飞书会话 ID：
```python
import json
with open('/opt/data/channel_directory.json') as f:
    data = json.load(f)
feishu_channels = data['platforms']['feishu']
chat_id = feishu_channels[0]['id']  # 用户 DM 的 open_id
```

**Step 3：发送消息（用 --text，不要用 --content）**

```shell
HOME=/opt/data/home /opt/data/home/.npm-global/bin/lark-cli im +messages-send \
  --chat-id <chat_id> \
  --text '<代码块表格内容>'
```

⚠️ `--content` 要求 JSON 格式（如 `'{"text":"hello"}'`），内容复杂时容易报错。
✅ 用 `--text` 直接传纯文本更可靠。

**Step 4：格式化表格**

用代码块（反引号```）等宽字体，表头与数据行对齐示例：
```
       名称              金额       类型    账本        项目归属      标签
────────────────────────────────────────────────────────────────────────────────
  东方树叶                   5  支出    个人        无             食物
────────────────────────────────────────────────────────────────────────────────
  今日支出合计: 377.85 元
```
计算合计时，金额字段为 `r[3]`，类型为 `r[10]`（`"支出"` 才累加）。

**筛选逻辑**：
- 今日 = `str(r[1])[:10] == datetime.now().strftime('%Y-%m-%d')`
- 用**创建日期**（r[1]）而非「支出日期（手动）」（r[5]）筛选——后者很多记录是 None

## 规则维护
- 每次用户纠正错误时，询问用户是否更新本 skill 的规则
- 新增标签必须由用户明确要求
- 项目归属的命名规则可随用户反馈持续迭代
