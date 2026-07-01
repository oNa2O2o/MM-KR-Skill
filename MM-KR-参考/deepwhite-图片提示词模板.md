# DeepWhite 图片提示词构建器（MM-KR适配版）

来源：deepwhite-image-prompt-builder。

## 适用位置

Phase 2（人设图）& Phase 3（分镜图）生成图片时，额外输出双语提示词文档供用户手动使用或换模型生成。

## 提示词结构

`[主体+动作] + [场景/背景] + [构图] + [光影] + [风格/美学] + [镜头/焦段] + [色调]`

## 输出格式

```markdown
**English Prompt**
[一段完整的英文提示词]

**中文提示词**
[一段可直接粘贴使用的中文提示词，保持相同意图而非逐字翻译]

**可迭代方向**
[可选，一行简短建议]
```

## 核心原则

### 主体优先
首句写清主体和动作，让画面能立即被理解。

### 构图语言
使用摄影术语：全景/中景/特写、平视/仰拍/俯拍、三分法/居中对称/过肩/POV。

### 光影即情绪
金色逆光、阴天漫射、硬光高对比、明暗法、霓虹填充光、烛光、雾中路灯、棚拍柔光。

### 镜头语言
85mm人像、24mm广角、50mm标准、微距、长焦压缩、浅景深、深焦、柔焦。

### 材质与质感
丝绸、拉丝金属、开裂混凝土、透明玻璃、湿沥青、纸质纹理、旧皮革。

### 色调分级
暖琥珀、冷青、去饱和哑光、高饱和鲜艳、高对比深黑、提亮暗部、双色调。

## MM-KR固定画风适配

在双语提示词中嵌入韩式厚涂固定参数：

**English**: Korean thick-painting illustration style, dark dreamy atmospheric tone, low saturation, soft haze glow, textured rendering, bloom blur blending

**中文**: 韩式厚涂画风，暗朦调，低饱和，泛朦柔光，质感渲染，泛光模糊晕染

## 人设图提示词示例

**English Prompt**
Character reference sheet of a Korean young woman, Korean thick-painting illustration style. White background. Left side: full-body standing pose in a cream knit sweater and dark fitted skirt, dark brown wavy hair past shoulders. Upper right: face close-up showing delicate features, soft gaze, subtle makeup. Three drama expressions below: gentle smile with slight head tilt, surprised wide eyes with parted lips, tearful gaze with furrowed brows. Lower right: color palette swatches. Dark dreamy atmospheric tone, low saturation, soft haze glow, textured rendering, bloom blur blending. No text, no watermarks.

**中文提示词**
韩式厚涂画风角色人设图，白色背景。左侧全身站姿：奶白色针织毛衣搭配深色修身短裙，深棕色微卷长发过肩。右上面部大特写：五官精致柔和，眼神温柔，淡妆。下方三个剧情表情：微侧头温柔浅笑、惊讶睁大眼睛微张嘴唇、含泪凝视微蹙眉头。右下角配色色块。暗朦调，低饱和，泛朦柔光，质感渲染，泛光模糊晕染。不要任何文字水印。

## 分镜图提示词示例

**English Prompt**
Storyboard grid of 4 key frames in 9:16 vertical format, arranged on a single sheet. Korean thick-painting style. Frame 1: medium shot, young woman sitting at a cafe window table, afternoon warm light from left, melancholic gaze into distance. Frame 2: close-up of her hand stirring coffee slowly, steam rising. Frame 3: over-shoulder shot showing a man approaching from the entrance, shallow depth of field. Frame 4: two-shot across the table, tension in body language, cool-toned overhead light. Dark dreamy atmosphere, low saturation, bloom blur, textured rendering. No text, no watermarks.

**中文提示词**
4帧关键分镜排列在一张图上，每帧9:16竖版比例，韩式厚涂画风。帧1：中景，年轻女子坐在咖啡馆靠窗位置，午后暖光从左侧照入，忧郁目光望向远处。帧2：特写她的手缓慢搅动咖啡，热气升腾。帧3：过肩镜头，一名男子从入口走来，浅景深虚化。帧4：两人隔桌对坐双人镜头，肢体语言紧张，顶部冷色调灯光。暗朦调，低饱和，泛光模糊晕染，质感渲染。不要任何文字水印。

## 规则

- 不推荐具体生成模型或平台
- 不输出视频提示词（无时间轴、无运镜、无音频）
- 不加负面提示词段落（除非用户要求）
- 不堆砌无关风格标签
- 不改变用户提供的角色身份、品牌细节
