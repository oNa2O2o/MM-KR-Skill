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

```javascript
const fs = require('fs'), path = require('path');
const cd = path.join(require('os').homedir(), '.api2img');
const cfg = JSON.parse(fs.readFileSync(path.join(cd,'config.json'),'utf-8'));
const sec = JSON.parse(fs.readFileSync(path.join(cd,'secret.json'),'utf-8'));
const apiUrl = cfg.baseUrl.replace(/\/+$/,'').replace(/\/v1$/,'') + '/v1/images/edits';

async function editImage(imagePath, prompt, outPath, size, retries=2) {
  for (let i = 0; i <= retries; i++) {
    const imageBuffer = fs.readFileSync(imagePath);
    const formData = new FormData();
    formData.append('image', new Blob([imageBuffer], {type:'image/png'}), 'image.png');
    formData.append('prompt', prompt);
    formData.append('model', 'gpt-image-2');
    formData.append('n', '1');
    formData.append('size', size || '1024x1536');
    formData.append('quality', 'high');
    const controller = new AbortController();
    const t = setTimeout(() => controller.abort(), 240000);
    try {
      const r = await fetch(apiUrl, {
        method: 'POST',
        signal: controller.signal,
        headers: {'Authorization': 'Bearer ' + sec.apiKey},
        body: formData
      });
      clearTimeout(t);
      const j = await r.json();
      if (j.data && j.data[0]) {
        const item = j.data[0];
        fs.mkdirSync(path.dirname(outPath), {recursive: true});
        if (item.b64_json) {
          fs.writeFileSync(outPath, Buffer.from(item.b64_json, 'base64'));
        } else if (item.url) {
          const ir = await fetch(item.url);
          fs.writeFileSync(outPath, Buffer.from(await ir.arrayBuffer()));
        }
        console.log('OK:', path.basename(outPath));
        return;
      } else {
        console.error('ERR attempt', i, JSON.stringify(j).substring(0, 300));
      }
    } catch (e) {
      clearTimeout(t);
      console.error('ERR attempt', i, ':', e.message);
    }
  }
  console.error('FAILED after retries');
}
```

## 常用尺寸
| 用途 | size参数 | 说明 |
|------|---------|------|
| 人设图 | `1536x1536` | 正方形，4K级 |
| 分镜宫格图 | `1536x1536` 或更大 | 正方形容纳多帧 |
| 单场景分镜 | `1024x1536` | 9:16竖版 |
| UI参考/绿幕 | `1024x1536` | 9:16竖版 |
