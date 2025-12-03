# Easy ASR

> 基于 sherpa-onnx WebAssembly 的实时语音识别系统

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![sherpa-onnx](https://img.shields.io/badge/powered%20by-sherpa--onnx-green.svg)](https://github.com/k2-fsa/sherpa-onnx)

## 📖 项目简介

Easy ASR 是一个纯浏览器端的实时语音识别应用，无需服务器支持，所有处理都在本地完成。基于 [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) WebAssembly 实现，支持多种先进的语音识别模型。

### ✨ 核心特性

- 🎙️ **实时流式识别**：支持麦克风实时录音和转写
- 📁 **文件批量处理**：支持上传音频/视频文件进行离线转写
- 🔄 **Two-Pass 识别**：结合在线 Zipformer/Paraformer 和离线 SenseVoice 双引擎，提升准确率
- 🌐 **AI 翻译集成**：支持中英互译（需配置翻译 API）
- 🎯 **VAD 语音检测**：支持 Silero VAD 和 TEN VAD，智能分段
- 💻 **离线优先**：ServiceWorker 支持，可离线使用
- ⚡ **WebGPU 加速**：SenseVoice 支持 WebGPU 后端加速（需浏览器支持）

---

## 🏗️ 技术架构

### 核心技术栈

- **sherpa-onnx WASM**：Next-gen Kaldi 的 WebAssembly 实现
- **ONNX Runtime Web**：用于 SenseVoice WebGPU 加速
- **Web Audio API**：音频采集和处理
- **Web Worker**：后台线程处理避免 UI 卡顿
- **Service Worker**：资源缓存和离线支持

### 识别引擎

#### 1. 在线流式识别（Online/Realtime）

| 模型 | 类型 | 描述 |
|------|------|------|
| **Zipformer** | Transducer | 默认模型，基于 encoder-decoder-joiner 架构 |
| **Paraformer** | Paraformer | Int8 量化版本，速度更快 |

**文件要求**：
- Zipformer: `encoder.onnx`, `decoder.onnx`, `joiner.onnx`, `tokens.txt`
- Paraformer: `paraformer-encoder.int8.onnx`, `paraformer-decoder.int8.onnx`, `paraformer-tokens.txt`

#### 2. 离线增强识别（Offline/Two-Pass）

| 后端 | 模型 | 特点 |
|------|------|------|
| **WASM** | SenseVoice | 纯 CPU 运算，兼容性好 |
| **WebGPU** | SenseVoice | GPU 加速，速度更快（需浏览器支持） |

**文件要求**：
- `second-pass-sense-voice.onnx` (或 `second-pass-sense-voice.int8.onnx`)
- `second-pass-tokens.txt`

#### 3. VAD（Voice Activity Detection）

| 模式 | 描述 |
|------|------|
| **Auto** | 自动选择可用的 VAD 引擎 |
| **Silero** | 快速轻量的 VAD 模型 |
| **TEN** | 更高精度的 VAD 模型 |

**文件要求**：
- Silero: `silero_vad.onnx`
- TEN: `ten-vad.onnx`

#### 4. 标点恢复（Punctuation）

使用 CT-Transformer 模型为识别文本添加标点符号。

**文件要求**：
- `punct-ct-transformer.onnx`
- `punct-ct-transformer-tokens.json`

---

## 🚀 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/anyshu/easy-asr.git
cd easy-asr
```

### 2. 下载 WASM 运行时文件

项目需要 sherpa-onnx WASM 运行时文件，请从以下地址下载：

🔗 **下载地址**：[https://huggingface.co/anyshu/sherpa-onnx-wasm-main-asr.data](https://huggingface.co/anyshu/sherpa-onnx-wasm-main-asr.data)

下载 `sherpa-onnx-wasm-main-asr.data` 文件并放入项目根目录。

> 💡 **注意**：项目中已包含 `sherpa-onnx-wasm-main-asr.js` 和 `sherpa-onnx-wasm-main-asr.wasm`，只需下载 `.data` 文件。

### 3. 准备模型文件

将以下模型文件放入项目根目录或 `assets/` 文件夹：

**必需文件（在线识别）**：
```
encoder.onnx
decoder.onnx
joiner.onnx
tokens.txt
```

**可选文件**：
```
# Paraformer 模型
paraformer-encoder.int8.onnx
paraformer-decoder.int8.onnx
paraformer-tokens.txt

# SenseVoice 离线模型（Two-Pass 增强识别）
second-pass-sense-voice.onnx  # 或 second-pass-sense-voice.int8.onnx
second-pass-tokens.txt

# VAD 模型
silero_vad.onnx
ten-vad.onnx

# 标点恢复
punct-ct-transformer.onnx
punct-ct-transformer-tokens.json
```

> 💡 **模型下载**：
> - **WASM 运行时**：[Hugging Face - sherpa-onnx-wasm-main-asr.data](https://huggingface.co/anyshu/sherpa-onnx-wasm-main-asr.data)
> - **识别模型**：[sherpa-onnx 模型仓库](https://github.com/k2-fsa/sherpa-onnx/releases) 下载对应的模型文件

### 4. 启动本地服务器

由于 WebAssembly 的跨域限制，需要通过 HTTP 服务器访问：

```bash
# 使用 Python（推荐）
python -m http.server 28000

# 或使用 Node.js
npx serve -p 28000 .

# 或使用 PHP
php -S localhost:28000
```

### 5. 访问应用

打开浏览器访问：`http://localhost:28000`

---

## 📚 使用指南

### 实时识别模式

1. 选择在线模型（Zipformer 或 Paraformer）
2. 选择 VAD 模式（Auto/Silero/TEN）
3. 点击 **Start Recording** 开始录音
4. 说话时实时显示识别结果
5. 点击 **Stop** 结束录音

### 文件上传模式

1. 切换到 **File Upload** 标签
2. 拖拽或选择音频/视频文件（支持 MP3, MP4, WAV, M4A 等）
3. 自动进行 VAD 分段和转写
4. 查看带时间戳的转写结果

### Two-Pass 增强识别

启用 Two-Pass 功能后，系统会：
1. 使用在线模型进行实时流式识别（低延迟）
2. 使用 SenseVoice 对结果进行二次处理（高准确率）
3. 自动替换为更准确的结果

**后端选择：**
- **WASM**：纯 CPU 运算，兼容性好，所有浏览器都支持
- **WebGPU (ORT)**：使用 GPU 加速，速度可提升 2-5 倍，但需要浏览器支持

💡 **提示**：首次使用 WebGPU 时，浏览器会下载 ONNX Runtime Web 库（约 2-3MB），之后会缓存

### AI 翻译功能

在 `app-asr.js` 中配置翻译 API：

```javascript
const translationConfig = {
  endpoint: 'http://localhost:8080/v1/chat/completions',  // 翻译 API 端点
  apiKey: 'your-api-key',                                  // API 密钥
  model: 'deepseek-v3.1',                                  // 模型名称
  enabled: false,                                           // 默认启用状态
  direction: 'en-zh',                                       // 翻译方向
};
```

支持的翻译方向：
- `en-zh`：英文 → 中文
- `zh-en`：中文 → 英文

---

## ⚙️ 配置说明

### 在线识别配置

编辑 `app-asr.js` 中的 `onlineModelSpecs`：

```javascript
const onlineModelSpecs = {
  zipformer: {
    label: 'Zipformer',
    type: 'transducer',
    assets: [
      {slot: 'encoder', paths: ['encoder.onnx'], label: 'encoder.onnx'},
      {slot: 'decoder', paths: ['decoder.onnx'], label: 'decoder.onnx'},
      {slot: 'joiner', paths: ['joiner.onnx'], label: 'joiner.onnx'},
      {slot: 'tokens', paths: ['tokens.txt'], label: 'tokens.txt'},
    ],
    tokens: './tokens.txt',
    hint: '使用 encoder/decoder/joiner（Zipformer）',
  },
  // ...
};
```

### 离线识别配置

在 `app-asr.js` 中配置 SenseVoice：

```javascript
const twoPassConfig = {
  enabled: true,          // 是否启用 Two-Pass
  backend: 'wasm',        // 'wasm' 或 'ort-webgpu'
};
```

### VAD 配置

VAD 分段策略：

```javascript
const offlineSegmentPolicy = {
  minSeconds: 2.4,      // 最小段落长度（SenseVoice 稳定性要求）
  maxSeconds: 18,       // 最大段落长度（控制延迟）
  maxBufferAgeMs: 6000  // 静音超时强制切分
};
```

---

## 🌐 浏览器兼容性

| 浏览器 | WebAssembly | WebGPU | 推荐版本 |
|--------|-------------|--------|----------|
| Chrome | ✅ | ✅ | >= 113 |
| Edge | ✅ | ✅ | >= 113 |
| Firefox | ✅ | ⚠️ | >= 102 |
| Safari | ✅ | ⚠️ | >= 16.4 |

> ⚠️ WebGPU 支持需要较新的浏览器版本，不支持时会自动降级到 WASM。

### WebGPU 使用和排查

**如何启用 WebGPU：**
1. 确保浏览器版本支持 WebGPU（Chrome/Edge >= 113）
2. 在 UI 中选择 Two-Pass Backend 为 "WebGPU (ORT)"
3. 打开浏览器控制台查看日志，确认使用了 WebGPU

**检查 WebGPU 是否正常工作：**
```javascript
// 在浏览器控制台运行
navigator.gpu !== undefined  // 应该返回 true
```

**常见问题：**
- 如果选项显示 "WebGPU (not available)"，说明浏览器不支持
- 查看控制台日志 `[SenseVoice ORT] ORT session created with provider`，显示 `webgpu` 表示成功
- 如果显示 `wasm`，说明降级到了 WASM 模式
- 确保模型文件名为 `second-pass-sense-voice.onnx`（不是 `sense-voice.onnx`）

---

## 📁 项目结构

```
easy-asr/
├── index.html                       # 主页面
├── app-asr.js                       # 主应用逻辑
├── sherpa-onnx-asr.js              # sherpa-onnx API 封装
├── sherpa-onnx-vad.js              # VAD 相关逻辑
├── sherpa-onnx-wasm-main-asr.js    # WASM 运行时 JS
├── sherpa-onnx-wasm-main-asr.wasm  # WASM 运行时二进制
├── sherpa-onnx-wasm-main-asr.data  # ⬇️ 需下载：WASM 运行时数据
├── offline-worker.js               # SenseVoice WASM Worker
├── sense-voice-ort-worker.js       # SenseVoice WebGPU Worker
├── service-worker.js               # PWA 离线支持
├── encoder.onnx                     # ⬇️ 需下载：在线模型文件
├── decoder.onnx                     # ⬇️ 需下载：在线模型文件
├── joiner.onnx                      # ⬇️ 需下载：在线模型文件
├── tokens.txt                       # ⬇️ 需下载：词表文件
└── README.md
```

> 📥 标记 **⬇️** 的文件需要额外下载

---

## 🔧 高级特性

### 自定义模型路径

修改 `resolvedOnlineModelPaths` 来自定义模型文件位置：

```javascript
const resolvedOnlineModelPaths = {
  zipformer: {
    encoder: './models/encoder.onnx',
    decoder: './models/decoder.onnx',
    joiner: './models/joiner.onnx',
    tokens: './models/tokens.txt',
  },
};
```

### Web Worker 通信

系统使用 Web Worker 进行后台处理：

- `offline-worker.js`：处理 SenseVoice WASM 识别
- `sense-voice-ort-worker.js`：处理 SenseVoice WebGPU 识别

消息协议：
```javascript
// 初始化
{ type: 'init', config: {...}, sampleRate: 16000 }

// 解码
{ type: 'decode', id: jobId, audioBuffer: Float32Array, sampleRate: 16000 }

// 释放
{ type: 'dispose' }
```

---

## 🔍 故障排查

### WebGPU 无法使用？

**症状**：Two-Pass Backend 选项显示 "WebGPU (not available)"

**解决方法**：

1. **检查浏览器版本**
   ```bash
   # Chrome/Edge: chrome://version
   # 确保版本 >= 113
   ```

2. **检查 WebGPU 支持**
   ```javascript
   // 在浏览器控制台运行
   console.log('WebGPU available:', navigator.gpu !== undefined);
   ```

3. **检查硬件加速**
   - Chrome: 访问 `chrome://gpu` 查看 GPU 状态
   - 确保 "WebGPU" 显示为 "Hardware accelerated"

4. **可能的原因**：
   - 旧版浏览器（更新到最新版）
   - 硬件加速被禁用（在浏览器设置中启用）
   - 显卡驱动过旧（更新显卡驱动）
   - 虚拟机环境（WebGPU 在某些虚拟机中不可用）

### WebGPU 已启用但没有加速效果？

**检查是否真的在使用 WebGPU**：

1. 打开浏览器控制台（F12）
2. 开始录音或上传文件
3. 查找日志：
   ```
   [SenseVoice ORT] ORT session created with provider webgpu
   ```
   - 如果显示 `webgpu`：✅ 正常使用 GPU 加速
   - 如果显示 `wasm`：⚠️ 降级到了 CPU 模式

**常见原因**：
- 模型文件损坏（重新下载）
- 浏览器内存不足（关闭其他标签页）
- GPU 被其他程序占用（关闭其他 GPU 密集型应用）

### WASM 运行时文件缺失？

**症状**：浏览器控制台报错 `Failed to load sherpa-onnx-wasm-main-asr.data`

**解决方法**：

1. **下载 WASM 运行时数据文件**
   - 访问：https://huggingface.co/anyshu/sherpa-onnx-wasm-main-asr.data
   - 下载 `sherpa-onnx-wasm-main-asr.data` 文件
   - 放入项目根目录（与 `index.html` 同级）

2. **确认文件完整性**
   - 文件大小应该在 10-50 MB 左右
   - 文件名必须是 `sherpa-onnx-wasm-main-asr.data`（不能改名）

3. **检查文件权限**
   ```bash
   # 确保文件可读
   ls -lh sherpa-onnx-wasm-main-asr.data
   ```

### 模型文件找不到？

**确保文件名正确**：
```
必需（WASM 运行时）：
✓ sherpa-onnx-wasm-main-asr.data  ← 从 Hugging Face 下载

必需（在线识别）：
✓ encoder.onnx
✓ decoder.onnx  
✓ joiner.onnx
✓ tokens.txt

Two-Pass（离线增强）：
✓ second-pass-sense-voice.onnx  ← 注意：不是 sense-voice.onnx
✓ second-pass-tokens.txt
```

**检查文件位置**：
- 所有文件应放在项目根目录
- 或放在 `assets/` 文件夹（需修改配置路径）

**查看加载状态**：
打开控制台，启动应用时会显示：
```
[ASR] Online model assets check:
  encoder.onnx: found
  decoder.onnx: found
  ...
```

### 识别结果不准确？

**尝试以下方法**：

1. **启用 Two-Pass**：大幅提升准确率
2. **选择合适的 VAD 模式**：
   - Auto: 自动选择
   - Silero: 快速但可能漏检
   - TEN: 更准确但稍慢
3. **调整 VAD 灵敏度**（需修改代码）
4. **使用质量更好的麦克风**
5. **减少环境噪音**

---

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

### 开发建议

1. Fork 本项目
2. 创建特性分支：`git checkout -b feature/amazing-feature`
3. 提交改动：`git commit -m 'Add amazing feature'`
4. 推送分支：`git push origin feature/amazing-feature`
5. 提交 Pull Request

---

## 📄 开源协议

本项目基于 MIT 协议开源，详见 [LICENSE](LICENSE) 文件。

---

## 🙏 致谢

- [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) - Next-gen Kaldi 语音识别框架
- [ONNX Runtime](https://onnxruntime.ai/) - 高性能推理引擎
- [SenseVoice](https://github.com/FunAudioLLM/SenseVoice) - 高精度语音识别模型

---

## 📞 联系方式

- 项目主页：https://github.com/anyshu/easy-asr
- Issue 反馈：https://github.com/anyshu/easy-asr/issues

---

## 🔮 未来规划

- [ ] 支持更多语言模型
- [ ] 添加说话人分离功能
- [ ] 优化移动端体验
- [ ] 增加热词定制功能
- [ ] 支持流式翻译
- [ ] 添加音频降噪预处理

---

**Made with ❤️ by Easy ASR Team**
