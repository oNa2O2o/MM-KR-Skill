# Seedance提示词 · 版本B · HuggingField JSON

ZH ≤1800字符

## 输出格式

```json
[
  {"lang":"en","prompt":"<<<image_1>>> Character reference sheet ... <<<image_2>>> Scene composition reference ... Style & Mood: ... Dynamic Description: ... Static Description: ... Audio: ..."},
  {"lang":"zh","prompt":"<<<image_1>>> 角色参考 ... <<<image_2>>> 场景构图参考 ... 风格与氛围：... 动态描述：... 静态描述：... 音频：..."}
]
```

## 段落结构

### Style & Mood / 风格与氛围
调色板、光影、镜头、氛围。不可省略。
画风以参考图为准，用 `{profile.art_style}` + `{profile.aesthetic_keywords}` 描述。

### Dynamic Description / 动态描述
逐镜头散文描述。镜头运动+角色动作。现在时态。
镜头自然晃动模拟手持拍摄。
每次cut遵循双重对比规则（同时改变景别+摄影模式）。

### Static Description / 静态描述
场景、道具、环境细节。为Dynamic中引用的一切建立基础。

### Audio / 音频（仅对话场景）
`{profile.language}` 台词原文（不翻译），音效/环境音描述。

## 核心规则
- 输出纯JSON，不加markdown围栏或解释
- 中文为母语导演笔记风格，不是翻译
- 图片引用：`<<<image_1>>>` `<<<image_2>>>`
- 台词保持 `{profile.language}` 原文，EN和ZH提示词中都用原语言
- 禁止年龄标记词（boy/girl/child/少年/少女等）
- 角色用功能标签描述（the figure / 身影）
- 微表情描述为物理动作（jaw clenches / 下颌咬紧）
- 禁止slop词表（见完整列表）
- 默认in medias res（场景已在进行中）

## 镜头语言速查
- 角度：low-angle/仰拍, high-angle/俯拍, eye-level/平视, OTS/过肩
- 运动：tracking/跟拍, dolly-in/推镜头, crane/摇臂, handheld/手持, orbit/环绕
- 时间：slow-motion/升格, speed ramp/变速
- 转场：smash cut/硬切, match cut/匹配剪辑

## 时长
在Style & Mood段末标注：Duration: Xs
时长 = `{profile.language}` 台词音节数 ÷ `{profile.speech_rate}` + 动作缓冲，目标8-12秒，上限15秒
