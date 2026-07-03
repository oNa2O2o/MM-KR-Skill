# MMv2-3screen · MiraiMind 三屏买量视频素材制作

---

## 触发

`/MMv2-3screen`、"做MM素材"、"买量视频"、"做KR素材"、"做JP素材"

---

## 定位

输入：大致剧情 + 角色参考图
输出：完整交付文件夹（文档 + 人设图 + 分镜图 + UI素材 + Seedance提示词&参考图）

通过用户档案适配不同地区（KR/JP/CN/EN等）、语言、画风和内容方向，同一Skill支持不同设计师的工作需求。

---

## Phase 0 · 初始化

### Step 0：自动更新检查

每次调用时**必须先执行**，再做任何其他操作。

1. 读取本地 `MMv2-3screen-参考/version.json` 的 `version` 字段
2. 获取远程版本：
   ```bash
   curl -sf --connect-timeout 5 "https://raw.githubusercontent.com/oNa2O2o/MMv2-3screen/main/version.json"
   ```
3. 比较版本（语义化：major.minor.patch）：
   - 获取失败 → `[更新检查] 无法连接，使用本地 vX.X.X`，继续
   - 版本相同 → `[更新检查] 已是最新 vX.X.X`，继续
   - 远程更高 → 展示 changelog，询问"更新/跳过"
4. 更新流程：
   ```bash
   SKILL_DIR="<.claude/commands绝对路径>"
   BACKUP_DIR=$(mktemp -d)
   cp -r "$SKILL_DIR/MMv2-3screen.md" "$SKILL_DIR/MMv2-3screen-参考" "$BACKUP_DIR/"
   curl -sL "https://github.com/oNa2O2o/MMv2-3screen/archive/refs/heads/main.tar.gz" -o /tmp/mmv2-update.tar.gz
   EXTRACT_DIR=$(mktemp -d)
   tar xzf /tmp/mmv2-update.tar.gz -C "$EXTRACT_DIR"
   SRC="$EXTRACT_DIR/MMv2-3screen-main"
   if [ -f "$SRC/MMv2-3screen.md" ]; then
     cp "$SRC/MMv2-3screen.md" "$SKILL_DIR/"
     cp -r "$SRC/MMv2-3screen-参考/"* "$SKILL_DIR/MMv2-3screen-参考/"
     echo "更新成功"; cat "$SKILL_DIR/MMv2-3screen-参考/version.json"
   else
     cp -r "$BACKUP_DIR/"* "$SKILL_DIR/"; echo "更新失败，已回滚"
   fi
   rm -rf "$BACKUP_DIR" "$EXTRACT_DIR" /tmp/mmv2-update.tar.gz
   ```
   注意：更新不会覆盖用户档案（`~/.mmv2-3screen/profile.json`），用户个性化数据安全。

### Step 1：加载用户档案

检测 `~/.mmv2-3screen/profile.json` 是否存在：

**已存在** → 读取并展示摘要：
```
[用户档案] 设计师：{designer} | 地区：{region} | 语言：{language} | 画风：{art_style} | 方向：{content_direction}
已有案例：{cases.length}个
```

**不存在** → 启动首次配置引导：

```
欢迎使用 MMv2-3screen！首次使用需要配置你的工作档案。

1. 你的名字/代号？
2. 主要制作哪个地区的素材？
   - KR（韩国）→ 预设韩语+韩式厚涂+6-8音节/秒
   - JP（日本）→ 预设日语+日系厚涂+7-9音节/秒
   - CN（中国）→ 预设中文+新国潮厚涂+4-6音节/秒
   - EN（英语区）→ 预设英语+Western digital painting+3-5音节/秒
   - 自定义 → 逐项填入
3. 内容方向？（如：恋爱养成·女性向、动作·男性向、悬疑·通用）
4. 目标受众？（如：18-35女性、Z世代男性）
```

用户回答后：
1. 从 `MMv2-3screen-参考/profile-template.json` 的 `presets` 读取对应地区预设
2. 合并用户自定义字段
3. 写入 `~/.mmv2-3screen/profile.json`
4. 展示完整档案让用户确认

**后续所有Phase中，凡涉及语言、画风、美学参数、语速、UI标签等内容，均从用户档案读取，不使用硬编码值。**

以下文档中用 `{profile.xxx}` 标记的位置，执行时替换为用户档案对应字段值。

### Step 2：环境检测

检测 `~/.api2img/config.json` 和 `~/.api2img/secret.json`
- 不存在 → 引导填入 image generation API 的 baseUrl 和 apiKey
- 配置格式见 `MMv2-3screen-参考/api2img-setup.md`

### Step 3：接收输入

- 大致剧情描述
- 角色参考图（文件路径或拖入）
- 工作目录（默认当前目录）
- 创建交付文件夹：`YYYY年M月D日_题材·关键词`
- 所有文件平铺在此文件夹中，用角色名前缀区分

---

## Phase 1 · 剧情·脚本·标题 ⛔ 用户说"合格"才继续

并列输出两个版本，用户选择或合并使用。

### 版本A · 快速简案（默认）

每个角色产出一个完整文档 `XX线_完整文档.md`，包含：

#### 顶部总览
- 身份、性格、好感度走向（数值三段递进）
- 视频大标题备选×3（`{profile.language}` + 中文）
- 标签（`{profile.language}`）

#### 按幕编排（每一幕包含该幕全部文案）
每一幕依次排列以下内容，不拆分到不同章节：
- 小标题备选×3（`{profile.language}` + 中文）
- 情节描述
- `{profile.language}` 台词原文 + 中文翻译（禁止内心戏）
- 结尾动作
- 场景标签：`{profile.ui_labels.scene_tag}` / `{profile.ui_labels.scene_tag_zh}`
- 好感度：`{profile.ui_labels.affinity}` / `{profile.ui_labels.affinity_zh}`
- 问题文字：`{profile.language}` / 中文
- 选项A/B：`{profile.language}` / 中文

### 版本B · DeepWhite编剧版

基于 deepwhite-screenwriting-v1（山音+screenwriter引擎），额外产出：

- **控制性理念**：`生活变成X，因为主角做了Y，而原因是Z`
- **人物深度**：Want / Need / Arc，人物矛盾性
- **场景价值转变**：每场标注进入价值→退出价值
- **因果性审计**：逐场检查因果链
- **双轨节奏标注**：情节松紧 × 情感轻重
- **量化评分**：7维度×10分
- **自检清单**：山音自检 + screenwriter审计

格式见 `MMv2-3screen-参考/deepwhite-编剧引擎.md`

### 通用规则
- 单角色单线，3段式情绪递进，不混线
- 台词必须直接可表演，不写内心戏、不写心理描写
- 只写能被摄影机拍到、麦克风听到的内容
- **主视角（观众）不说话**：所有台词只属于非POV角色，禁止给主视角写对白
- 每个视频一个文档，不分文档

---

## Phase 2 · 人设图 ⛔ 用户确认

1. 使用 **image edit API**（`/v1/images/edits`），用户原图作输入
2. 提示词模板：`保持这个角色的外貌特征和画风不变。生成这个角色的人设参考图，白色背景，包括：左侧全身站姿，右上面部五官大特写，【剧情对应的】三个表情，右下角一排配色色块。不要任何文字。`
3. 分辨率：4K（`1536x1536` 或 API支持的最大正方形尺寸）
4. 输出到项目目录：`角色名_人设图.png`（不建子文件夹）

### 附加：DeepWhite双语图片提示词

额外输出 `角色名_图片提示词.md`：
- 结构：`[主体+动作] + [场景] + [构图] + [光影] + [风格] + [镜头] + [色调]`
- English Prompt + 中文提示词，嵌入 `{profile.art_style}` + `{profile.aesthetic_keywords}` 固定参数
- 格式见 `MMv2-3screen-参考/deepwhite-图片提示词模板.md`

---

## Phase 3 · 分镜图 ⛔ 用户确认

1. 使用 **image edit API**，人设图作输入
2. 根据场景的动作变化拆分关键帧数量（通常3-6帧）
3. 所有帧按9:16比例排列到一张图上，4K分辨率
4. 提示词要求：
   - 保持角色外貌和画风不变
   - 描述每个帧的场景/动作/表情
   - **服装描述必须逐项对照人设图**，不凭记忆
   - 禁止夸张瞪眼等AI味表情
   - **禁止越轴**：车内等封闭空间双人场景中，摄像机位置固定在一侧（如副驾），视角方向只在"看角色"和"看前方/窗外"之间切换，不可跨越180度轴线
5. 输出到项目目录：`角色名_分镜_场景X.png`（不建子文件夹）

**画风固定参数**：`{profile.art_style}` + `{profile.aesthetic_keywords}` 全部关键词
详见 `MMv2-3screen-参考/画风约束.md`

### 附加：DeepWhite双语图片提示词

每张分镜图同步输出双语提示词，文件名 `角色名_分镜图片提示词.md`，平铺在项目目录中。
格式见 `MMv2-3screen-参考/deepwhite-图片提示词模板.md`

---

## Phase 4 · Seedance提示词 ⛔ 用户确认

### 产出
每个视频一个文档，直接放在项目目录中：`角色名_Seedance提示词.md`。参考图同样平铺在项目目录。

### 时长计算
`{profile.language}` 台词音节数 ÷ `{profile.speech_rate}` = 朗读秒数，加上动作/停顿缓冲 → 目标8-12秒，单条上限15秒。

### 默认输出：HuggingField中文版

**版本B · HuggingField JSON**（默认）
- 格式见 `MMv2-3screen-参考/huggingfield模板.md`
- 默认只输出中文提示词部分
- ZH ≤1800字符

### 可选版本（用户主动要求时输出）

**版本A · 情绪导演六段式**（用户说"加情绪导演版"时输出）
- 格式见 `MMv2-3screen-参考/情绪导演模板.md`
- 六段：视听限制 / 语言台词 / 运镜手法 / 风格色调与光景 / 角色与场景设定 / 时间轴详细叙事
- 总字数 ≤1900字，中文

**版本C · DeepWhite分镜版**（用户说"加DeepWhite版"时输出）
- 格式见 `MMv2-3screen-参考/deepwhite-分镜提示词模板.md`
- 风格四段 → 素材句柄 → 镜头段落 → 环境音效 → 防错警告
- 摄影机-情绪同步 + 表演微节拍
- 每条 ≤2200中文字符

### 通用规则（三版共用）
- 镜头自然晃动模拟手持拍摄
- [图1]人设图 [图2]分镜图 引用格式（版本C中对应@image1/@image2）
- 禁止引用拟声词音节
- 无比喻修辞，纯物理描述
- 服装描述逐项对照人设图/分镜图
- `{profile.language}` 台词保持原文，不翻译

### 参考图组织
每个场景准备 `角色名_场景X_图1_人设.png` + `角色名_场景X_图2_分镜.png`，平铺在项目目录中

---

## Phase 5 · CTA文案（自动输出，无需确认）

每次交付标配一条5秒CTA文案：
- 前半句：剧情钩子（概括本条素材的核心drama）
- 后半句：产品引导（MiraiMind + 角色多样性卖点）
- 输出5版备选，`{profile.language}` + 中文对照
- 用户选定后记录在交付文档中

---

## Phase 6 · 整理交付

验证最终文件夹结构（所有文件平铺在项目目录中，不建子文件夹）：
```
YYYY年M月D日_题材·关键词/
├── 角色A_人设图.png
├── 角色A_图片提示词.md
├── 角色A线_完整文档.md
├── 角色A线_DeepWhite编剧版.md（如用户选用）
├── 角色A_分镜_场景1.png
├── 角色A_分镜_场景2.png
├── 角色A_分镜_场景3.png
├── 角色A_Seedance提示词.md
├── 角色A_分镜图片提示词.md
├── 角色A_场景1_图1_人设.png
├── 角色A_场景1_图2_分镜.png
├── 角色A_场景2_图1_人设.png
├── 角色A_场景2_图2_分镜.png
├── 角色A_场景3_图1_人设.png
├── 角色A_场景3_图2_分镜.png
├── UI参考.png（仅用户要求时生成）
├── UI绿幕模板.png（仅用户要求时生成）
└── （角色B/C同理，用角色名前缀区分）
```

---

## Phase 7 · 复盘 ⛔ 与用户对话

1. 罗列本次制作过程中用户提出的所有修改点
2. 逐条询问：**"是否固化为标准规则？"**
3. 用户确认的规则 → 追加到本Skill的固定规则部分
4. 如用户认为本次交付合格，询问是否**添加到个人案例库**：
   ```
   是否将本次交付加入你的案例库？
   案例名称：[自动建议，用户可改]
   案例路径：[交付文件夹路径]
   案例备注：[简述题材、角色数、特殊手法]
   ```
   用户确认 → 写入 `~/.mmv2-3screen/profile.json` 的 `cases` 数组

---

## 用户档案系统

### 档案位置
`~/.mmv2-3screen/profile.json` — 存放在用户主目录，不随Skill更新被覆盖。

### 档案结构
```json
{
  "designer": "设计师名",
  "region": "KR",
  "language": "韩语",
  "art_style": "韩式厚涂",
  "art_style_en": "Korean thick-painting style",
  "aesthetic_keywords": ["暗朦调", "泛朦", "低饱和", "反射", "质感", "泛光模糊晕染"],
  "aesthetic_keywords_en": ["dark dreamy atmospheric tone", "soft haze glow", ...],
  "speech_rate": "6-8",
  "content_direction": "恋爱养成·女性向",
  "target_audience": "18-35女性",
  "ui_labels": {
    "scene_tag": "{N}일차 - {地点}",
    "affinity": "호감도 {X}",
    "scene_tag_zh": "第{N}天 - {地点}",
    "affinity_zh": "好感度 {X}"
  },
  "cases": [
    {
      "name": "千金三选一",
      "date": "2026-06-30",
      "path": "F:\\钟海明\\2026\\2026年6月30日MM-KR-千金三选一",
      "region": "KR",
      "notes": "3角色×3场景，恋爱养成选择题，韩式厚涂，首批完整交付"
    }
  ]
}
```

### 档案读取规则
- 每次调用Skill时在 Step 1 自动读取
- 所有 `{profile.xxx}` 标记处替换为实际值
- 如果档案存在但缺少某个字段 → 使用该 region 预设的默认值
- 用户随时可说"修改档案"触发重新配置

### 案例库用途
- Phase 1 创作时参考历史案例的题材和风格
- 新用户首次使用时无案例，随使用积累
- 案例只存引用路径和简述，不复制实际文件

---

## 固定规则

### 画风
以用户提供的参考图为准，image edit API 自动延续参考图画风。无参考图时使用 `{profile.art_style}` + `{profile.aesthetic_keywords}`。

### 图片生成
- 一律使用 image edit API（`/v1/images/edits`），以参考图为输入
- 禁止纯文本生成（角色不一致）
- 人设图和分镜图均需4K
- api2img代码模板见 `MMv2-3screen-参考/api2img-setup.md`

### 角色表现
- 服装描述必须逐项对照人设图，禁止凭记忆
- 禁止夸张瞪眼等AI味浓的表情
- 表情自然克制
- **主视角（观众）默认不说话**：剧情和台词中禁止出现主视角角色的对白，所有台词只属于非POV角色

### 文档
- 每个视频一个完整文档，不分文档
- 文档按幕编排，每一幕的所有文案（剧情、台词、标题、问题、选项）放在一起
- 中文文档，`{profile.language}` 台词 + 中文翻译
- 交付文件夹命名：`YYYY年M月D日_题材·关键词`

### 文件结构
- 所有产出文件平铺在项目目录中，不建子文件夹
- 用角色名前缀区分不同角色的文件

### Seedance提示词
- 提示词和参考图平铺在项目目录中
- [图1][图2]引用格式，禁止@图片1@图片2（版本C中@image1/@image2与之对应）
- 禁止引用拟声词音节（Seedance会当对白朗读）
- 镜头自然晃动模拟手持拍摄
- 时长按 `{profile.language}` 台词音节计算（`{profile.speech_rate}` 音节/秒），目标8-12秒，单条上限15秒

### DeepWhite集成
- 编剧版（Phase 1版本B）：山音+screenwriter引擎，量化评分，因果审计
- 图片提示词（Phase 2&3附加）：双语提示词文档，供跨平台使用
- 分镜提示词（Phase 4版本C）：风格四段+镜头段落+微表演+摄影机-情绪同步，≤2200字
- 参考文件：`MMv2-3screen-参考/deepwhite-编剧引擎.md`、`deepwhite-图片提示词模板.md`、`deepwhite-分镜提示词模板.md`

---

## 附录 · UI素材（用户主动要求时执行）

默认不执行。用户说"做UI"、"加UI素材"时触发。

### Step 1：UI参考图
- 用分镜图作为背景，叠加游戏风格UI
- UI元素使用 `{profile.ui_labels}` 中的标签格式
- 输出到项目目录：`UI参考.png`

### Step 2：UI绿幕提取
- 使用 **image edit API**，UI参考图作输入
- 提示词：`在纯亮绿色色度键背景（hex #00FF00）上，保留图片中所有UI元素（对话框、好感度条、选项按钮、场景标签等），保持原始配色、大小、比例完全不变。完全移除角色插图和所有背景场景。每个UI元素之间留出间距，单独排列。背景必须是纯色 #00FF00，不能有任何渐变、阴影或纹理。`
- 输出到项目目录：`UI绿幕模板.png`
- UI全局复用，不带文案
