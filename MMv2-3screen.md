# MMv2-3screen · MiraiMind 三屏买量视频素材制作

---

## 触发

`/MMv2-3screen`、"做MM素材"、"买量视频"、"做KR素材"、"做JP素材"

---

## 定位

输入：大致剧情 + 角色参考图
输出：完整交付文件夹（文档 + 人设图 + Seedance提示词&参考图 + 分镜图 + UI素材）

通过用户档案适配不同地区（KR/JP/CN/EN等）、语言、画风、角色偏好和内容方向，同一Skill支持不同设计师的工作需求。

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
   # Windows 兼容：使用 TEMP 环境变量替代 /tmp 和 mktemp
   TMPDIR_WIN=$(cygpath -m "$TEMP" 2>/dev/null || echo "$TEMP")
   SKILL_DIR=$(cygpath -m "$(cd "$(dirname "$0")/../.claude/commands" 2>/dev/null && pwd)" 2>/dev/null)
   # 如果 SKILL_DIR 解析失败，由调用方（Claude）直接填入 .claude/commands 的绝对路径
   BACKUP_DIR="$TMPDIR_WIN/mmv2-backup-$$"
   EXTRACT_DIR="$TMPDIR_WIN/mmv2-extract-$$"
   mkdir -p "$BACKUP_DIR" "$EXTRACT_DIR"
   cp -r "$SKILL_DIR/MMv2-3screen.md" "$SKILL_DIR/MMv2-3screen-参考" "$BACKUP_DIR/"
   curl -sL "https://github.com/oNa2O2o/MMv2-3screen/archive/refs/heads/main.tar.gz" -o "$TMPDIR_WIN/mmv2-update.tar.gz"
   tar xzf "$TMPDIR_WIN/mmv2-update.tar.gz" -C "$EXTRACT_DIR"
   SRC="$EXTRACT_DIR/MMv2-3screen-main"
   if [ -f "$SRC/MMv2-3screen.md" ]; then
     cp "$SRC/MMv2-3screen.md" "$SKILL_DIR/"
     cp -r "$SRC/MMv2-3screen-参考/"* "$SKILL_DIR/MMv2-3screen-参考/"
     echo "更新成功"; cat "$SKILL_DIR/MMv2-3screen-参考/version.json"
   else
     cp -r "$BACKUP_DIR/"* "$SKILL_DIR/"; echo "更新失败，已回滚"
   fi
   rm -rf "$BACKUP_DIR" "$EXTRACT_DIR" "$TMPDIR_WIN/mmv2-update.tar.gz"
   ```
   注意：更新不会覆盖用户档案（`~/.mmv2-3screen/profile-*.json`），用户个性化数据安全。

### Step 1：加载用户档案

扫描 `~/.mmv2-3screen/` 目录下所有 `profile-*.json` 文件：

**找到多个档案** → 列出所有可用档案供用户选择：
```
[用户档案] 检测到以下档案：
  1. KR — 设计师：{designer} | 画风：{art_style} | 案例：{cases.length}个
  2. EN — 设计师：{designer} | 画风：{art_style} | 案例：{cases.length}个
  ...
请选择本次使用的档案（输入编号或地区代码）：
```
用户选择后加载对应 `profile-{REGION}.json`，展示完整摘要：
```
[已激活] 设计师：{designer} | 地区：{region} | 语言：{language} | 画风：{art_style} | 方向：{content_direction}
角色偏好：发色={character_preferences.hair_color} | 体型={character_preferences.body_proportion} | 服装={character_preferences.outfit_color}
已有案例：{cases.length}个
```

**只找到一个档案** → 直接加载并展示摘要，不询问。

**没有档案** → 启动首次配置引导：

```
欢迎使用 MMv2-3screen！首次使用需要配置你的工作档案。

1. 你的名字/代号？
2. 主要制作哪个地区的素材？
   - KR（韩国）→ 预设韩语+韩式厚涂+6-8音节/秒+亮色发/亮色衣/9头身
   - JP（日本）→ 预设日语+日系厚涂+7-9音节/秒
   - CN（中国）→ 预设中文+新国潮厚涂+4-6音节/秒
   - EN（英语区）→ 预设英语+Western digital painting+3-5音节/秒
   - 自定义 → 逐项填入
3. 内容方向？（如：恋爱养成·女性向、动作·男性向、悬疑·通用）
4. 目标受众？（如：18-35女性、Z世代男性）
```

用户回答后：
1. 从 `MMv2-3screen-参考/profile-template.json` 的 `presets` 读取对应地区预设（含 `character_preferences`）
2. 合并用户自定义字段
3. 写入 `~/.mmv2-3screen/profile-{REGION}.json`（按地区代码命名，不写入 `profile.json`）
4. 展示完整档案让用户确认

**档案文件命名规则**：所有档案统一使用 `profile-{REGION}.json` 格式（如 `profile-KR.json`、`profile-EN.json`）。旧的 `profile.json` 不再使用，首次检测到时自动迁移为 `profile-{region}.json`。

**用户说"修改档案"** → 列出所有档案，用户选择要修改的档案后进入编辑流程。
**用户说"添加档案"** → 启动首次配置引导，创建新地区档案。

**后续所有Phase中，凡涉及语言、画风、美学参数、语速、UI标签、角色偏好等内容，均从用户档案读取，不使用硬编码值。**

以下文档中用 `{profile.xxx}` 标记的位置，执行时替换为用户档案对应字段值（包括 `{profile.character_preferences.hair_color}` 这类嵌套字段）。

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

默认输出版本A（快速简案）。用户说"加DeepWhite编剧版"时额外输出版本B。

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
- 问题+选项组×3版本（三选一）

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

**文档规范**
- 每个视频一个完整文档，不分文档
- 文档按幕编排，每一幕的所有文案（剧情、台词、标题、问题、选项）放在一起
- 中文文档，`{profile.language}` 台词 + 中文翻译

**剧情结构**
- 单角色单线，3段式情绪递进，不混线
- **场景跨度必须拉大**：三幕不能都在同一栋楼/同一空间内，转场变化要明显（例：宿舍门口→舞台后台→男厕隔间）
- **POV必须有明确身份设定**：写清楚POV是谁（idol/学生/警察/职员等），身份决定drama的动机与压力源，无身份则动机悬空
- **POV默认选择最drama的选项推进下一幕**：写剧情时假设玩家每次都选最激烈的动作，让因果链闭合递进（例：上锁→她消失三天→捂嘴→她把捂嘴当"他还想碰我"证据→翻墙进厕所）

**逐秒运镜脚本先出节拍表对齐**（适用于一镜到底 / 逐秒节拍 / 含明确运镜的脚本）
- **动手前先出节拍表跟用户确认**，颗粒度到「拍号｜画面｜角色朝向·动作｜台词｜机位运镜」。对齐后再一次性产出脚本+Seedance提示词(Phase 3)+分镜(Phase 4)。
  - **为什么**：逐秒脚本里每一拍的"动作/朝向/机位"强耦合，且同一信息会分散写在正文逐秒节拍、开头视角说明、结尾段、Seedance的EN/ZH双语、分镜提示词等多处。不先对齐就动手，会陷入"改一处漏三处"的返工——用户否一次改一次，反复打架。
- **运镜语言（尤其结尾镜头：过肩/拉远/特写）是独立决策点，用 AskUserQuestion 让用户早期拍板**，不要自己默认一个、被否了再猜。
  - **为什么**：结尾镜头是分叉点，猜错的代价是脚本+提示词+分镜三份产物一起返工；分镜图虽可逆，但每次重跑都要用户看图才发现方向错，成本转嫁给了用户。
- **节拍表是单一事实源**：任何一拍改动，同步刷新所有文件里所有相关段落，禁止只改被点名的那一处。

**台词与选项**
- 台词必须直接可表演，不写内心戏、不写心理描写
- 只写能被摄影机拍到、麦克风听到的内容
- **主视角（观众）不说话**：所有台词只属于非POV角色，禁止给主视角写对白
- **台词一句一个信息点**：视频中每句台词展示时间仅1-2秒，过长的台词主动拆分，禁止一句话塞两个信息（如"你是我的男人，不许低头"应拆为两句）
- **区分台词层级**：标注"氛围"和"角色台词"，方便制作时分层处理（如：配角嘲讽属于氛围音，女主命令属于角色台词）
- **选项必须极短且必须drama**：选项控制在**2-4词、一眼扫完**的长度（韩语约6韩字内/中文约6字内/英语约3词内），两个选项都要是激烈动作，禁止"重新打开门/直接上锁"这种平淡应对
- **选项必须围绕当幕核心冲突物/道具**：选项动作必须和当幕的核心冲突直接关联（如欠条→撕欠条、酒杯飞来→避开/挡下），禁止脱离剧情的随机肢体动作（如推墙、搂腰）
- **选项问题必须是即时画面描述**：描述正在发生的那一秒的物理画面（如"酒杯正朝她飞来"），禁止氛围概括（如"气氛紧张""牌桌将翻"）
- **结尾动作停在悬念点**：结尾动作只写到"僵持不下/脚步声将至/有人推门"，不能提前把选项动作演完；选项本身才是玩家的抉择时刻，禁止倒果为因
- **每个问题给3个版本供三选一**：问题文字+选项组一次输出3套，让用户挑最带感的组合

---

## Phase 2 · 人设图 ⛔ 用户确认

### 生成方式

1. 使用 **image edit API**（`/v1/images/edits`），用户原图作输入
2. 提示词模板（自动注入设计师档案偏好）：
   ```
   保持这个角色的外貌特征和画风不变。生成这个角色的人设参考图，白色纯净背景。
   身体比例：{profile.character_preferences.body_proportion}（禁止Q版/大头娃娃）。
   发色遵循：{profile.character_preferences.hair_color}。
   服装配色遵循：{profile.character_preferences.outfit_color}。
   布局：左侧全身站姿（从头到脚完整可见），右上面部五官大特写，右中三个【剧情对应的】表情，右下角一排配色色块。
   风格：{profile.art_style_en} + {profile.aesthetic_keywords_en 全部关键词}。
   不要任何文字。
   ```
3. 分辨率：4K（`1536x1536` 或 API支持的最大正方形尺寸）
4. 输出到项目目录：`角色名_人设图.png`（不建子文件夹）

### 图片生成通用规则（Phase 2、4 共用）

- 一律使用 image edit API（`/v1/images/edits`），以参考图为输入
- 禁止纯文本生成（`/v1/images/generations`），角色不一致
- 人设图分辨率：`1536×1536`（正方形4K级）；分镜图分辨率：`1536×1024`（21:9超宽横版）
- api2img 代码模板见 `MMv2-3screen-参考/api2img-setup.md`
- 服装描述必须逐项对照人设图，禁止凭记忆
- 禁止夸张瞪眼、嘴巴大张等AI味浓的表情
- 表情自然克制

### 附加：DeepWhite双语图片提示词

额外输出 `角色名_图片提示词.md`：
- 结构：`[主体+动作] + [场景] + [构图] + [光影] + [风格] + [镜头] + [色调]`
- English Prompt + 中文提示词，嵌入 `{profile.art_style}` + `{profile.aesthetic_keywords}` + `{profile.character_preferences}` 固定参数
- 格式见 `MMv2-3screen-参考/deepwhite-图片提示词模板.md`

---

## Phase 3 · Seedance提示词 ⛔ 用户确认

**执行顺序**：本 Phase 必须先于 Phase 4 分镜图。分镜是本 Phase 动态描述段落的可视化，而不是本 Phase 的输入源。写完动态描述节拍后再进入 Phase 4 出分镜，然后回到本 Phase 补齐 [图2] 分镜引用。

### 产出
每个视频一个文档，直接放在项目目录中：`角色名_Seedance提示词.md`。

### 时长计算
`{profile.language}` 台词音节数 ÷ `{profile.speech_rate}` = 朗读秒数，加上动作/停顿缓冲 → 目标8-12秒，单条上限15秒。

### 默认输出：HuggingField中文版

**版本A · HuggingField JSON**（默认）
- 格式见 `MMv2-3screen-参考/huggingfield模板.md`
- 默认只输出中文提示词部分
- ZH ≤1800字符

### 可选版本（用户主动要求时输出）

**版本B · 情绪导演六段式**（用户说"加情绪导演版"时输出）
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
- 参考图引用统一使用 `@image1`（人设图，色彩/五官锚定）和 `@image2`（分镜图，动作/构图/POV锚定），所有版本格式一致
- 禁止 `@图片1` `@图片2`、`[图1]` `[图2]`、`<<<image_1>>>` `<<<image_2>>>` 等其他格式
- 禁止引用拟声词音节（Seedance会当对白朗读）
- 无比喻修辞，纯物理描述
- 服装描述逐项对照人设图
- `{profile.language}` 台词保持原文，不翻译

### 参考图组织
每个场景准备 `角色名_场景X_图1_人设.png` + `角色名_场景X_图2_分镜.png`，平铺在项目目录中。图1在 Phase 2 完成后复制，图2在 Phase 4 生成分镜时同步复制。

---

## Phase 4 · 分镜图 ⛔ 用户确认

**执行顺序**：本 Phase 必须在 Phase 3 Seedance 提示词的"动态描述"段落完成之后执行。分镜就是动态描述节拍的可视化。

### 生成方式

1. 使用 **image edit API**，人设图作输入
2. 根据 Phase 3 Seedance 提示词的"动态描述"段落拆分关键帧数量（通常3-6帧）
3. **分镜格式：21:9超宽横版（1536×1024），关键帧以竖版9:16 strips从左到右排列**，条与条之间用细黑线分隔。API size参数使用 `1536x1024`，在提示词中明确要求21:9超宽比例构图。禁止输出竖版单张或正方形。
4. **必须生成线稿**：黑色墨水线稿，白色纸面，无上色，clean manga line drawing。分工上：人设图承担色彩/五官锚定，分镜承担动作/构图/POV/空间关系锚定，两张一起喂给 Seedance。
5. 提示词要求：
   - 保持角色外貌和画风不变
   - 描述每个帧的场景/动作/表情
   - **服装描述必须逐项对照人设图**，不凭记忆
   - **同场景内服装必须完全一致**：高领不能变成拉开、外套不能变成敞开、任何衣物细节在所有 strip 里必须保持相同状态
   - **同场景内空间构图不能跳轴**：门/家具/角色位置在所有 strip 中固定在画面同一侧（例：门始终在右侧，角色始终从左侧接近），禁止镜像翻转
   - 禁止夸张瞪眼等 AI 味表情
   - **第一人称 POV 严格执行**：绝不显示男主的脸/身体，仅允许男主的手从画面下方伸入
6. 输出到项目目录：`角色名_分镜_场景X.png`（不建子文件夹），同时复制一份为 `角色名_场景X_图2_分镜.png` 作为 Seedance 参考图

### 附加：DeepWhite双语图片提示词

每张分镜图同步输出双语提示词，文件名 `角色名_分镜图片提示词.md`，平铺在项目目录中。
格式见 `MMv2-3screen-参考/deepwhite-图片提示词模板.md`

---

## Phase 5 · CTA文案

### 产出规范

每次交付标配一条5-6秒 CTA 文案：
- **每版CTA必须包含 MiraiMind 产品名**，禁止遗漏
- **前半句**：剧情钩子（概括本条素材的核心 drama）
- **后半句**：必须包含 MiraiMind 多角色/多故事卖点，列举**至少3种代表性角色属性**（如병娇/傲娇/御姐/千金/黑道女等），突出"{profile.language} 数量规模词"（如韩语"수백 명의 그녀"）。禁止只写产品名不写卖点。
- 自动输出 4-5 版备选，`{profile.language}` + 中文对照
- 用户选定后记录在交付文档中

### 附加：CTA配音演员指导（选定版本后自动输出）

CTA 确定后同步输出配音指导，写入交付文档，包含：
- **声线选择**：性别、年龄感、音色（例：男声/20代中后半/略带气声低音）
- **分句处理**：逐句标注语速、重音、尾音处理
- **情绪曲线**：分句情绪起伏（例：紧迫警告→展示菜单→诱惑收尾）
- **时长控制**：总时长目标 + 分段配比
- **参考对标**：具体的作品或广告类型（例：Netflix 韩剧预告片旁白）
- **禁忌**：语调、发音、节奏要避免的问题（例：品牌名不要发成日式）

---

## Phase 6 · 整理交付

### 文件结构规则
- 所有产出文件平铺在项目目录中，不建子文件夹
- 用角色名前缀区分不同角色的文件
- 交付文件夹命名：`YYYY年M月D日_题材·关键词`

### 验证最终文件夹结构
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
3. 用户确认的规则 → 按下表**就近归位到对应 Phase**，禁止把同一规则重复添加到多处（避免维护漂移）

### 规则归位表

| 规则类型 | 归位到 |
|---|---|
| 剧情结构 / 台词 / 选项 / POV身份 / 因果链 / 文档规范 | **Phase 1** 通用规则 |
| 人设图生成 / 图片工艺参数 / 表情克制 | **Phase 2** 图片生成通用规则 |
| Seedance 提示词 / 时长计算 / 引用格式 / 拟声词禁令 | **Phase 3** 通用规则 |
| 分镜图格式 / 线稿 / 服装一致 / 不跳轴 / POV视角 | **Phase 4** 提示词要求 |
| CTA 文案 / 多角色卖点 / 配音指导 | **Phase 5** |
| 文件命名 / 平铺结构 / 交付清单 | **Phase 6** 文件结构规则 |
| 用户档案字段 / 地区预设 | **用户档案系统** + `profile-template.json` |
| 角色偏好（发色 / 体型 / 服装色 / 地区口味） | **`{profile.character_preferences.xxx}`**，写入档案而非规则 |
| DeepWhite 引擎 / 引用参考文件 | 各 Phase 的"附加"小节 |

4. 如用户认为本次交付合格，询问是否**添加到个人案例库**：
   ```
   是否将本次交付加入你的案例库？
   案例名称：[自动建议，用户可改]
   案例路径：[交付文件夹路径]
   案例备注：[简述题材、角色数、特殊手法]
   ```
   用户确认 → 写入当前激活的 `~/.mmv2-3screen/profile-{REGION}.json` 的 `cases` 数组

---

## 用户档案系统

### 档案位置
`~/.mmv2-3screen/profile-{REGION}.json` — 存放在用户主目录，按地区代码命名（如 `profile-KR.json`、`profile-EN.json`），不随 Skill 更新被覆盖。每次启动 Skill 时从可用档案中选择激活。

### 档案结构
```json
{
  "designer": "设计师名",
  "region": "KR",
  "language": "韩语",
  "art_style": "韩式厚涂",
  "art_style_en": "Korean thick-painting style",
  "aesthetic_keywords": ["暗朦调", "泛朦", "低饱和", "反射", "质感", "泛光模糊晕染"],
  "aesthetic_keywords_en": ["dark dreamy atmospheric tone", "soft haze glow"],
  "speech_rate": "6-8",
  "content_direction": "恋爱养成·女性向",
  "target_audience": "18-35女性",
  "ui_labels": {
    "scene_tag": "{N}일차 - {地点}",
    "affinity": "호감도 {X}",
    "scene_tag_zh": "第{N}天 - {地点}",
    "affinity_zh": "好感度 {X}"
  },
  "character_preferences": {
    "hair_color": "亮色优先（铂金/奶金/樱粉/蜜橙/薰衣草/白发等）",
    "hair_color_reason": "该地区市场偏好该发色区间转化率高",
    "outfit_color": "亮色/浅色系优先，避免暗黑深色系主导",
    "body_proportion": "9头身模特比例，禁止Q版/大头娃娃",
    "body_proportion_reason": "地区乙游/恋爱题材主流审美"
  },
  "cases": [
    {
      "name": "案例名",
      "date": "YYYY-MM-DD",
      "path": "绝对路径",
      "region": "KR",
      "notes": "简述"
    }
  ]
}
```

### 档案读取规则
- 每次调用 Skill 时在 Step 1 自动读取
- 所有 `{profile.xxx}` 标记处替换为实际值（包括 `{profile.character_preferences.hair_color}` 这类嵌套字段）
- 如果档案存在但缺少某个字段 → 使用该 region 预设的默认值（见 `profile-template.json` 的 `presets`）
- 用户随时可说"修改档案"触发重新配置

### 案例库用途
- Phase 1 创作时参考历史案例的题材和风格
- 新用户首次使用时无案例，随使用积累
- 案例只存引用路径和简述，不复制实际文件

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
- UI 全局复用，不带文案
