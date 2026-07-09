# api2img 配置与代码模板

## 首次配置

检测 `~/.api2img/config.json` 是否存在，不存在则引导用户填入：

```
请提供 image generation API 配置：
1. API Base URL（例：https://api.example.com/v1）
2. API Key
```

### config.json
```json
{
  "baseUrl": "用户填入的URL"
}
```

### secret.json
```json
{
  "apiKey": "用户填入的Key"
}
```

---

## 代码模板 · image edit API（标准用法）

用于人设图、分镜图、UI提取等所有图片生成。

**重要**：Node.js fetch 在此 API 上会超时（大 base64 响应约 1.7MB），必须使用 curl 发请求。

### bash + curl 模板

```bash
# 读取配置
CONFIG_DIR="$HOME/.api2img"
BASE_URL=$(node -e "const c=require('$CONFIG_DIR/config.json');console.log(c.baseUrl.replace(/\/+$/,'').replace(/\/v1$/,''))")
API_KEY=$(node -e "console.log(require('$CONFIG_DIR/secret.json').apiKey)")
API_URL="$BASE_URL/v1/images/edits"

# 参数：IMAGE_PATH, PROMPT_FILE, OUT_PATH, SIZE
IMAGE_PATH="$1"
PROMPT_FILE="$2"  # 提示词写入临时文件，避免 heredoc 转义问题
OUT_PATH="$3"
SIZE="${4:-1024x1536}"

# Windows 兼容临时目录
TMPDIR=$(cygpath -m "$TEMP" 2>/dev/null || echo "/tmp")
RESP_FILE="$TMPDIR/api2img_resp_$$.json"

# 发送请求（curl，超时 300s）
curl -s --max-time 300 \
  -H "Authorization: Bearer $API_KEY" \
  -F "image=@$IMAGE_PATH" \
  -F "prompt=<$PROMPT_FILE" \
  -F "model=gpt-image-2" \
  -F "n=1" \
  -F "size=$SIZE" \
  -F "quality=high" \
  "$API_URL" > "$RESP_FILE"

# 解码 base64 并保存
node -e "
const fs=require('fs');
const j=JSON.parse(fs.readFileSync('$RESP_FILE','utf-8'));
if(j.data && j.data[0] && j.data[0].b64_json){
  fs.writeFileSync('$OUT_PATH',Buffer.from(j.data[0].b64_json,'base64'));
  console.log('OK:','$(basename "$OUT_PATH")');
}else{
  console.error('ERR:',JSON.stringify(j).substring(0,300));
  process.exit(1);
}
"
rm -f "$RESP_FILE"
```

### 注意事项
- 提示词先写入临时文件再用 `-F "prompt=<file"` 传入，避免 shell 转义问题
- Windows 上用 `cygpath -m "$TEMP"` 获取 Node.js 和 bash 都能访问的路径
- curl 超时设 300s，足够处理 4K 图片生成
- 不要用 Node.js fetch — 此 API 返回大 base64 JSON 会导致 fetch 超时

## 常用尺寸
| 用途 | size参数 | 说明 |
|------|---------|------|
| 人设图 | `1536x1536` | 正方形，4K级 |
| 分镜图 | `1536x1024` | 21:9超宽横版，竖条strips左到右排列 |
| UI参考/绿幕 | `1024x1536` | 9:16竖版 |
