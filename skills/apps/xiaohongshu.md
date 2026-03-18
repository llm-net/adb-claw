# 小红书 (Xiaohongshu / RedNote)

- 包名: `com.xingin.xhs`
- Scheme: `xhsdiscover://`

## 深度链接

**中文输入必须用深度链接**，`adb-claw type` 不支持中文，会报错 `TYPE_FAILED`。

```bash
# 搜索（推荐方式）
keyword=$(python3 -c "import urllib.parse; print(urllib.parse.quote('保健品推荐'))")
adb-claw open "xhsdiscover://search/result?keyword=${keyword}&type=51"

# 打开搜索首页（空搜索框）
adb-claw open "xhsdiscover://search"

# 启动 App（无深度链接时）
adb-claw app launch com.xingin.xhs
```

| 动作 | 链接 | 参数说明 |
|------|------|----------|
| 搜索内容 | `xhsdiscover://search/result?keyword={encoded}&type=51` | keyword 需 URL encode；type=51 经测试有效（综合结果页）|
| 打开搜索页 | `xhsdiscover://search` | 打开搜索页但不触发搜索 |

> **注意**：`type` 参数值 51 经实测可正常加载搜索结果页。其他 type 值未测试，如失效可省略 type 参数。

## 已知布局

### 搜索结果页

```
┌─────────────────────────────────────────────────┐
│ [<]  搜索关键词                     [X]  [搜索]   │
├─────────────────────────────────────────────────┤
│  全部 ▼  │  用户  │  商品  │  图片  │  地点  │ 问一问 │  ← 分类 Tab
├─────────────────────────────────────────────────┤
│  综合 │ 可购买 │ 最新 │ {场景标签...}              │  ← 二级筛选（综合 Tab 下）
├─────────────────────────────────────────────────┤
│                                                   │
│   [封面]  帖子标题...                              │
│           作者名     日期     点赞数               │
│                                                   │
│   [封面]  帖子标题...                              │
│                                                   │
└─────────────────────────────────────────────────┘
```

**关键元素定位**：
- 搜索输入框: `text` 含搜索词，`content_desc="全部删除"` 的按钮在其右侧（⚠️见已知问题）
- 分类 Tab: `text="全部"` / `text="用户"` / `text="商品"` 等
- 搜索按钮: `text="搜索"`
- 帖子卡片: `content_desc` 格式为 `"笔记,{标题},来自{作者},{N}赞，"`
- 视频帖卡片: `content_desc` 格式为 `"视频,{标题},来自{作者},{N}赞，"`
- 相关搜索: `text="大家都在搜"` 下方列表

### 用户搜索结果页（用户 Tab）

```
┌─────────────────────────────────────────────────┐
│  全部 ▼  │  [用户]  │  商品  │  图片  │  ...      │
├─────────────────────────────────────────────────┤
│  [头像]  账号名称  ✓                    [关注]    │
│           粉丝 X.X万                              │
│           {认证标签 / 小红书号}                   │
│           发过相关笔记（可选显示）                  │
├─────────────────────────────────────────────────┤
│  [头像]  账号名称                       [关注]    │
│           粉丝 XXX                               │
│           小红书号：XXXXXXXXXX                    │
└─────────────────────────────────────────────────┘
```

**关键元素定位**：
- 用户名: `text="{账号名}"`
- 粉丝数: `text="粉丝 X.X万"` 或 `text="粉丝 XXXX"`
- 认证标签: `text="保健食品"` / `text="营养师"` 等，在粉丝数下方
- 关注按钮: `text="关注"`（未关注）/ `text="已关注"`（已关注）

### 用户主页（个人主页）

```
┌─────────────────────────────────────────────────┐
│ [<]  头图                          [更多]         │
├─────────────────────────────────────────────────┤
│  [头像]  账号名称                                  │
│           {认证标签}  {小红书号}  IP:{地区}         │
│           个人简介...                              │
│           {学校} {城市} {达人标签...}               │
│                                                   │
│  [关注数]关注   [粉丝数]粉丝   [获赞与收藏数]获赞   │
│                                     [关注] [私信]  │
├─────────────────────────────────────────────────┤
│  Ta的瞬间 │ 直播预告 │ 签到足迹 │ 心情日签          │  ← 动态栏（滑动显示）
├─────────────────────────────────────────────────┤
│  [橱窗选品 3 件预览]                               │  ← 橱窗入口（有橱窗才显示）
├─────────────────────────────────────────────────┤
│  笔记  │  选品  │  收藏                            │  ← 内容 Tab
├─────────────────────────────────────────────────┤
│  {合集1}  {合集2}  {合集3}                         │  ← 合集（有才显示）
├─────────────────────────────────────────────────┤
│  [置顶帖1]  [置顶帖2]                              │
│  [帖子瀑布流]                                      │
└─────────────────────────────────────────────────┘
```

**关键元素定位**：
- 粉丝数: `content_desc="{N}粉丝"` → 取 `text` 字段
- 关注数: `content_desc="{N}关注"`
- 获赞与收藏: `content_desc="{N}获赞与收藏"`
- 原创内容数: `content_desc="{N}条原创内容"`（部分账号显示）
- 橱窗入口: `text="橱窗"` 或 `content_desc="营养品, 25件好物"`（点击进入橱窗）
- 笔记 Tab: `content_desc="已选定笔记"` 表示当前选中
- 帖子卡片: `content_desc="笔记,{标题},来自{账号名},{N}赞，"`

### 帖子详情页（图文笔记）

```
┌─────────────────────────────────────────────────┐
│ [<]  [作者头像]  作者名         [关注]   [分享]   │
├─────────────────────────────────────────────────┤
│                 图片（可左右滑动）                  │
├─────────────────────────────────────────────────┤
│  标题                                             │
│  正文内容...                                       │
│  #标签1 #标签2                                     │
│  日期  地点                                        │
├─────────────────────────────────────────────────┤
│  关键词搜索：{词1}  {词2}                          │
├─────────────────────────────────────────────────┤
│  共 N 条评论                                       │
│  [评论列表]                                        │
├─────────────────────────────────────────────────┤
│  说点什么...    [收藏N]  [评论N]  [点赞N]           │
└─────────────────────────────────────────────────┘
```

### 帖子详情页（视频）

视频类帖子打开后全屏播放，底部有帖子信息条。**向下滚动会切换到下一个视频**，不是滚动正文——这是最大的操作陷阱。

### 橱窗页（选品）

```
┌─────────────────────────────────────────────────┐
│ [搜索橱窗内的商品]                                 │
│  {账号名}的橱窗   粉丝 X.X万   新笔记 N篇          │
├─────────────────────────────────────────────────┤
│  {分类1} N件好物  │ {分类2} N件好物  │ ...         │  ← 分类筛选（可横向滚动）
├─────────────────────────────────────────────────┤
│  综合  │  销量  │  新品  │  价格                   │  ← 排序
├─────────────────────────────────────────────────┤
│  买手专属价                                        │
│  [商品图]  {品牌}·{商品名（混排字符）}              │
│            {折扣信息}  专属价 ¥XXX   已售 XXXX      │
└─────────────────────────────────────────────────┘
```

**注意**：橱窗商品名为**零宽字符分隔的散字**（如 `n​a​t​u​r​e​w​i​s​e`），UI tree 中 text 字段为混排字符，需拼接。过滤方法：抓取含 `·` 的 text 或含 `已售` 的 text。

## 设备差异

本 Profile 基于 **Xiaomi M2007J1SC（Phone）** 测试：

- 屏幕：1080×2120，Android 13 (SDK 33)
- 竖屏为主要使用模式
- 搜索结果双列瀑布流（图文模式）

Pad 上的差异未经测试，预计搜索结果页会变为三列或更宽布局。

## 常见工作流

### 搜索内容（推荐：全程深度链接）

```bash
# 1. 用深度链接搜索，keyword 必须 URL encode
keyword=$(python3 -c "import urllib.parse; print(urllib.parse.quote('保健品推荐'))")
adb-claw open "xhsdiscover://search/result?keyword=${keyword}&type=51"
sleep 3    # 等待结果加载

# 2. 默认是综合 Tab，需要看用户时切到「用户」Tab
adb-claw tap --text "用户"
sleep 2

# 3. 收集帖子信息
adb-claw ui tree | python3 -c "
import json,sys; data=json.load(sys.stdin)
for e in data['data']['elements']:
    d=e.get('content_desc','')
    if ('笔记' in d or '视频' in d) and '来自' in d: print(d[:120])
"
```

### 批量翻页收集帖子（作者+点赞数）

```bash
# 快速滚动 + 提取 text（过滤噪音词）
for i in 1 2 3 4 5; do
  adb-claw scroll down
  sleep 0.8
  adb-claw ui tree 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
for e in data['data']['elements']:
    t = e.get('text','').strip()
    if t and len(t)<80 and not any(x in t for x in ['搜索','全部','用户','商品','图片','地点','问一问','综合','最新','可购买','相关','大家都在']):
        print(t)
"
  echo "---"
done
```

### 搜索特定用户并进入主页

```bash
# 搜索用户名
keyword=$(python3 -c "import urllib.parse; print(urllib.parse.quote('注册营养师Yuanyuan'))")
adb-claw open "xhsdiscover://search/result?keyword=${keyword}&type=51"
sleep 2
adb-claw tap --text "用户"
sleep 2
# 点击第一个用户结果
adb-claw tap --text "注册营养师Yuanyuan"
sleep 3
```

### 读取用户主页核心数据

```bash
adb-claw ui tree 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
for e in data['data']['elements']:
    d = e.get('content_desc','')
    if any(k in d for k in ['粉丝','关注','获赞与收藏','原创内容']):
        print(d)
"
```

输出示例：
```
14.8万粉丝
610关注
236.6万获赞与收藏
```

### 查看橱窗产品

```bash
# 进入主页后
adb-claw tap --text "橱窗"    # 或点击橱窗预览区域
sleep 2
# 点击分类（如「营养品」）
adb-claw tap --text "营养品"
sleep 2
# 提取产品名和销量
adb-claw ui tree 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
for e in data['data']['elements']:
    t = e.get('text','').strip()
    if t and ('·' in t or '已售' in t): print(t[:100])
"
# 滚动翻页继续提取
adb-claw scroll down && sleep 1
```

### 滚动个人主页收集所有帖子标题

```bash
for i in 1 2 3; do
  adb-claw scroll down && sleep 0.8
  adb-claw ui tree 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
for e in data['data']['elements']:
    d = e.get('content_desc','')
    if ('笔记' in d or '视频' in d) and '来自' in d: print(d[:120])
"
done
```

### 点入帖子读取正文和评论区

```bash
# 在个人主页帖子瀑布流中，先定位帖子 bounds
adb-claw ui tree 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
for e in data['data']['elements']:
    d = e.get('content_desc','')
    if '乳铁蛋白' in d:
        b = e['bounds']
        print(f'tap {(b[\"left\"]+b[\"right\"])//2} {(b[\"top\"]+b[\"bottom\"])//2}')
"
# 按输出坐标点击
adb-claw tap 806 1695
sleep 3
# 读取正文
adb-claw ui tree 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
for e in data['data']['elements']:
    t = e.get('text','').strip()
    if t and len(t)>30: print(t[:300])
"
# 滚动到评论区读取作者互动
adb-claw scroll down && sleep 1
```

## 已知问题

### ❶ `adb-claw type` 不支持中文，直接报错

**现象**：`adb-claw type "保健品"` 返回 `TYPE_FAILED: text contains non-ASCII characters`。

**原因**：底层 `adb shell input text` 不支持中文/CJK 字符。

**解决**：所有中文关键词搜索**必须用深度链接**，URL encode 后传入：
```bash
keyword=$(python3 -c "import urllib.parse; print(urllib.parse.quote('保健品'))")
adb-claw open "xhsdiscover://search/result?keyword=${keyword}&type=51"
```

---

### ❷ 「全部」Tab 与「全部删除」按钮容易误触

**现象**：搜索框有焦点时，`adb-claw tap --text "全部"` 点到了**全部删除**（清空搜索框）按钮，而不是搜索结果的「全部」分类 Tab。

**原因**：搜索框聚焦时，右侧的「全部删除」（`content_desc="全部删除"`）按钮的 text 也被匹配。

**解决**：
1. 优先用深度链接导航，不手动操作搜索框；
2. 如必须切换分类 Tab，先确认搜索框**没有焦点**（先点空白处），再 tap Tab；
3. 或直接用坐标点击（观察截图确认坐标再操作）。

---

### ❸ 「用户」Tab 只找用户名含关键词的账号

**现象**：搜索「保健品」切到「用户」Tab，结果只有账号名里带"保健品"字样的品牌小号，粉丝量普遍很小（几百到几千），找不到真正的 KOL。

**原因**：用户 Tab 按账号名匹配，不按内容匹配。真正的保健品 KOL（如营养师、配方师、健康博主）账号名中不含"保健品"。

**解决**：
1. 在「全部/综合」Tab 浏览热门帖，从帖子的作者名中识别 KOL；
2. 换更具体的关键词（如「保健品推荐」「营养师 保健品」「保健品测评」）；
3. 找到 KOL 名字后，再单独搜索其账号名进入主页。

---

### ❹ 截图有时显示旧内容（缓存）

**现象**：执行 `adb-claw open` 后立刻截图，截图内容仍是前一页面，不是新页面。

**原因**：App 页面渲染需要时间，截图先于渲染完成。

**解决**：`adb-claw open` 后固定等待 `sleep 2~3`，重要页面等待更长（如主页 `sleep 3`）。

---

### ❺ 视频帖中「滚动」= 切换下一个视频

**现象**：在视频帖子详情页执行 `adb-claw scroll down`，变成切换到下一个视频，而非滚动正文/评论区。

**原因**：小红书视频播放器占全屏，竖向滑动被识别为切换视频（和抖音一样）。

**解决**：
1. 读取视频帖正文时，直接从 UI tree 的 `text` 字段读，不需要滚动；
2. 若要读评论，先找评论区元素坐标，在评论区范围内小幅滑动（而非全屏 scroll）。

---

### ❻ resource_id 全部混淆，不可依赖

**现象**：UI tree 中所有元素的 `resource_id` 均为 `com.xingin.xhs:id/0_resource_name_obfuscated`，无法通过 ID 区分元素。

**原因**：小红书 App 对资源 ID 做了混淆。

**解决**：定位元素**只能依赖 `text` 和 `content_desc`**，必要时结合 `bounds`（坐标范围）。

---

### ❼ 橱窗商品名为零宽字符散字，提取需特殊处理

**现象**：橱窗商品名在 UI tree 中形如 `n​a​t​u​r​e​w​i​s​e​·​【​达​人​专​属​】​活​性​2​5​羟​基​维​生​素`，每个字符之间有零宽字符。

**原因**：小红书对橱窗商品名做了零宽字符插入（反爬/防复制）。

**解决**：过滤条件不依赖字符串精确匹配，而是判断 `·` 是否存在，或判断 `已售` 是否存在来锚定商品条目：
```python
if '·' in t or '已售' in t: print(t[:100])
```

---

### ❽ 帖子网格点击坐标需手动计算

**现象**：个人主页帖子列表为 2 列瀑布流，`content_desc` 含帖子信息但无法直接 `tap --desc`，因为 `--desc` 只匹配完整字符串。

**解决**：通过 UI tree 找到帖子元素的 `bounds`，手动计算中心点坐标后 tap：
```python
b = e['bounds']
cx = (b['left'] + b['right']) // 2
cy = (b['top'] + b['bottom']) // 2
# 然后 adb-claw tap {cx} {cy}
```

---

### ❾ 搜索词在搜索框中有残留时需先清空

**现象**：通过深度链接打开过一次搜索后，再 `adb-claw open "xhsdiscover://search"` 打开搜索页，输入框中可能仍有上次的词。

**解决**：始终使用带 `keyword` 参数的深度链接直接搜索，不复用搜索页手动改词。

---

### ❿ 「笔记」Tab 的 content_desc 是「已选定笔记」

**现象**：在个人主页想 `tap --text "笔记"` 却未必生效（Tab 已是激活状态）。

**原因**：当前选中的 Tab content_desc 变为 `"已选定笔记"`，text 仍为 `"笔记"`。

**解决**：用 `--text "笔记"` 可以正常 tap，因为 text 字段不变；但若 Tab 已选中则点击无副作用。

## 推荐能力

| 能力 | 方法 | 适用场景 |
|------|------|---------|
| 搜索内容/用户 | 深度链接 `xhsdiscover://search/result?keyword={encoded}&type=51` | 中文搜索唯一可靠方式 |
| 读取用户主页数据 | `ui tree` + 过滤 content_desc | 粉丝数、获赞数、认证标签 |
| 批量收集帖子作者+点赞 | 滚动 + 过滤 content_desc 含"笔记/来自" | KOL 筛选 |
| 读取橱窗产品 | tap 橱窗 → 滚动 + 过滤含"·"或"已售" | 品牌合作分析 |
| 读取帖子正文 | 进帖后 `ui tree` 的 text 字段 | 内容/产品分析 |
| 读取评论区作者回复 | 滚动到评论区 + `ui tree` | 找到作者推荐的具体产品 |

---

> 测试设备: Xiaomi M2007J1SC (cas), Android 13 (SDK 33), 屏幕 1080×2120, 密度 440dpi
> 测试 App 版本: 小红书 2026 年 3 月版
> 包名: `com.xingin.xhs`
