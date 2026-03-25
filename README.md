# xiaozhi-server - 小智后端核心服务

## 目录

- [项目简介](#项目简介)
- [功能特性](#功能特性)
- [系统架构](#系统架构)
- [快速开始](#快速开始)
- [配置说明](#配置说明)
- [目录结构](#目录结构)
- [模块说明](#模块说明)
- [开发指南](#开发指南)
- [常见问题](#常见问题)

---

## 项目简介

**xiaozhi-server** 是小智ESP32智能硬件的核心后端服务，采用 Python 异步架构实现，为智能语音交互提供完整的后端能力。

本服务是 [xiaozhi-esp32-server](https://github.com/xinnan-tech/xiaozhi-esp32-server) 项目的核心组件，由华南理工大学刘思源教授团队研发。

---

## 功能特性

### 核心能力
- **实时语音交互** - 流式 ASR、流式 TTS、VAD 语音活动检测
- **多模态对话** - 支持文本、语音、视觉多模态交互
- **声纹识别** - 支持多用户声纹注册与识别，实现个性化回应
- **记忆系统** - 支持本地短期记忆和云端长期记忆
- **意图识别** - 基于 LLM 和函数调用的智能意图理解
- **工具调用** - 插件系统、IoT 控制、MCP 协议支持

### 服务能力
- **WebSocket 服务** - 实时双向通信（默认端口 8000）
- **HTTP 服务** - OTA 升级、视觉分析接口（默认端口 8003）
- **认证系统** - 设备 Token 认证、MAC 白名单
- **配置热更新** - 支持从 API 动态获取配置

---

## 系统架构

### 技术栈
| 组件 | 技术选型 |
|------|---------|
| **异步框架** | asyncio + websockets |
| **配置管理** | YAML + 本地覆盖 |
| **日志系统** | loguru |
| **音频处理** | pydub + opuslib |
| **ML 框架** | torch + funasr |

### 消息处理流程

```
设备 (ESP32)
    ↓ WebSocket
连接握手 (hello)
    ↓
音频接收 (receiveAudio)
    ↓
VAD 检测 → ASR 语音识别 → 声纹识别
    ↓
意图识别 → 工具调用 (可选)
    ↓
LLM 对话生成
    ↓
TTS 语音合成
    ↓
音频发送 (sendAudio)
```

---

## 快速开始

### 环境要求

- Python 3.9+
- FFmpeg（音频处理必需）
- 内存：2GB+（使用本地模型需要 4GB+）

### 安装依赖

```bash
# 进入 xiaozhi-server 目录
cd main/xiaozhi-server

# 安装 Python 依赖
pip install -r requirements.txt

# 检查 FFmpeg 是否安装
# Windows: 下载 ffmpeg 并添加到 PATH
# Linux: sudo apt install ffmpeg
# macOS: brew install ffmpeg
```

### 配置准备

1. 创建数据目录和自定义配置：
```bash
mkdir data
touch data/.config.yaml
```

2. 编辑 `data/.config.yaml`（仅需填写需要覆盖的配置）：
```yaml
selected_module:
  ASR: FunASR
  LLM: ChatGLMLLM
  TTS: EdgeTTS

LLM:
  ChatGLMLLM:
    api_key: "你的智谱API密钥"
```

### 启动服务

```bash
python app.py
```

启动成功后会显示：
```
Websocket地址是        ws://192.168.x.x:8000/xiaozhi/v1/
OTA接口是             http://192.168.x.x:8003/xiaozhi/ota/
视觉分析接口是        http://192.168.x.x:8003/mcp/vision/explain
```

### 测试服务

使用浏览器打开 `test/test_page.html` 进行音频交互测试。

---

## 配置说明

### 配置文件机制

系统采用**双层配置**机制：
- `config.yaml` - 主配置文件（包含所有选项和默认值）
- `data/.config.yaml` - 用户覆盖配置（仅需填写需要修改的项）

**优先级**：`data/.config.yaml` > `config.yaml` > 智控台配置

### 服务器配置

```yaml
server:
  ip: 0.0.0.0              # 监听地址
  port: 8000                # WebSocket 端口
  http_port: 8003           # HTTP 端口
  websocket: ws://...       # 对外 WebSocket 地址
  vision_explain: http://... # 视觉分析接口地址
  auth:
    enabled: false           # 是否启用设备认证
    tokens:                  # 设备 Token 列表
      - token: "xxx"
        name: "device1"
```

### 模块选择配置

```yaml
selected_module:
  VAD: SileroVAD            # 语音活动检测
  ASR: FunASR               # 语音识别
  LLM: ChatGLMLLM           # 大语言模型
  VLLM: ChatGLMVLLM         # 视觉语言模型
  TTS: EdgeTTS               # 语音合成
  Memory: nomem              # 记忆模块
  Intent: function_call      # 意图识别
```

### 推荐配置方案

#### 入门全免费配置
| 模块 | 选型 | 说明 |
|------|------|------|
| ASR | FunASR | 本地模型，免费 |
| LLM | ChatGLMLLM | glm-4-flash 免费 |
| VLLM | ChatGLMVLLM | glm-4v-flash 免费 |
| TTS | LinkeraiTTS | 灵犀流式免费 |
| Intent | function_call | 函数调用 |
| Memory | mem_local_short | 本地短期记忆 |

#### 流式配置（推荐）
| 模块 | 选型 | 说明 |
|------|------|------|
| ASR | DoubaoStreamASR | 豆包流式，体验好 |
| LLM | DoubaoLLM | 豆包大模型 |
| VLLM | QwenVLVLLM | 千问视觉模型 |
| TTS | HuoshanDoubleStreamTTS | 火山双流式 |
| Intent | function_call | 函数调用 |
| Memory | mem_local_short | 本地短期记忆 |

---

## 目录结构

```
xiaozhi-server/
├── app.py                      # 主入口文件
├── config.yaml                 # 主配置文件
├── requirements.txt            # Python 依赖
├── agent-base-prompt.txt       # 基础提示词
├── config/                     # 配置模块
│   ├── settings.py            # 配置加载入口
│   ├── config_loader.py       # 配置加载器
│   ├── logger.py              # 日志配置
│   └── manage_api_client.py   # 智控台 API 客户端
├── core/                       # 核心实现
│   ├── connection.py          # 连接管理器（核心）
│   ├── websocket_server.py    # WebSocket 服务
│   ├── http_server.py         # HTTP 服务
│   ├── auth.py                # 认证中间件
│   ├── handle/                # 消息处理器
│   │   ├── helloHandle.py     # 握手处理
│   │   ├── textHandle.py      # 文本消息
│   │   ├── receiveAudioHandle.py  # 接收音频
│   │   ├── sendAudioHandle.py     # 发送音频
│   │   ├── intentHandler.py   # 意图处理
│   │   ├── abortHandle.py     # 中断处理
│   │   └── reportHandle.py    # 上报处理
│   ├── api/                   # HTTP API 处理
│   │   ├── ota_handler.py     # OTA 升级
│   │   └── vision_handler.py  # 视觉分析
│   ├── providers/             # 服务提供商
│   │   ├── asr/               # 语音识别
│   │   ├── llm/               # 语言模型
│   │   ├── tts/               # 语音合成
│   │   ├── vad/               # 语音活动检测
│   │   ├── intent/            # 意图识别
│   │   ├── memory/            # 记忆模块
│   │   ├── vllm/              # 视觉模型
│   │   └── tools/             # 工具系统
│   └── utils/                 # 工具模块
│       ├── modules_initialize.py  # 模块初始化
│       ├── dialogue.py        # 对话管理
│       ├── prompt_manager.py  # 提示词管理
│       ├── asr.py             # ASR 工厂
│       ├── llm.py             # LLM 工厂
│       ├── tts.py             # TTS 工厂
│       └── ...
├── plugins_func/              # 插件系统
│   ├── register.py            # 插件注册
│   └── functions/             # 插件实现
│       ├── get_weather.py     # 天气查询
│       ├── play_music.py      # 音乐播放
│       ├── get_time.py        # 获取时间
│       ├── change_role.py     # 角色切换
│       └── ...
├── models/                    # 模型文件目录
├── music/                     # 音乐文件目录
├── data/                      # 数据目录
│   └── .config.yaml          # 用户覆盖配置
├── tmp/                       # 临时文件目录
└── test/                      # 测试工具
    └── test_page.html        # 音频测试页面
```

---

## 模块说明

### 核心连接管理 (connection.py)

`ConnectionHandler` 是整个服务的核心类，负责：
- WebSocket 连接生命周期管理
- 音频缓冲区管理
- ASR → Intent → LLM → TTS 流程协调
- 设备认证与绑定
- 动态配置更新

### Providers 系统

所有 AI 服务都通过统一的 Provider 接口实现，便于扩展。

#### ASR（语音识别）
支持的提供商：
- `FunASR` - 本地模型（推荐免费方案）
- `FunASRServer` - FunASR 独立服务
- `DoubaoASR` / `DoubaoStreamASR` - 豆包语音识别
- `AliyunASR` / `AliyunStreamASR` - 阿里云语音识别
- `TencentASR` - 腾讯云语音识别
- `BaiduASR` - 百度语音识别
- `OpenaiASR` / `GroqASR` - OpenAI 兼容接口
- `SherpaASR` - Sherpa ONNX 本地模型

#### LLM（大语言模型）
支持的提供商：
- `ChatGLMLLM` - 智谱 AI（glm-4-flash 免费）
- `DoubaoLLM` - 火山引擎豆包
- `AliLLM` / `AliAppLLM` - 阿里云百炼
- `DeepSeekLLM` - DeepSeek
- `GeminiLLM` - Google Gemini
- `OllamaLLM` - Ollama 本地模型
- `DifyLLM` - Dify 平台
- `FastgptLLM` - FastGPT
- `CozeLLM` - Coze 扣子
- `XinferenceLLM` - Xinference
- `HomeAssistant` - Home Assistant 对话代理

#### TTS（语音合成）
支持的提供商：
- `EdgeTTS` - 微软 Edge TTS（免费）
- `DoubaoTTS` - 豆包语音合成
- `HuoshanDoubleStreamTTS` - 火山双流式
- `AliyunTTS` / `AliyunStreamTTS` - 阿里云 TTS
- `TencentTTS` - 腾讯云 TTS
- `MinimaxTTS` - Minimax
- `OpenAITTS` - OpenAI TTS
- `FishSpeech` - FishSpeech 本地模型
- `GPT_SOVITS_V2` / `GPT_SOVITS_V3` - GPT-SoVITS
- `CosyVoiceSiliconflow` - 硅基流动
- `LinkeraiTTS` - 灵犀流式（免费）
- `CustomTTS` - 自定义 TTS 接口

#### VAD（语音活动检测）
- `SileroVAD` - Silero VAD 本地模型

#### Intent（意图识别）
- `nointent` - 不使用意图识别
- `intent_llm` - 基于 LLM 的意图识别
- `function_call` - 函数调用（推荐）

#### Memory（记忆系统）
- `nomem` - 无记忆
- `mem_local_short` - 本地短期记忆（推荐）
- `mem0ai` - Mem0 AI 云端记忆

### 插件系统

插件通过 `@register_function` 装饰器注册，内置插件包括：

| 插件 | 功能 |
|------|------|
| `get_weather` | 天气查询（和风天气 API） |
| `get_news_from_chinanews` | 中国新闻 RSS |
| `get_news_from_newsnow` | NewsNow 新闻 |
| `play_music` | 音乐播放 |
| `change_role` | 角色切换 |
| `get_time` | 获取时间 |
| `handle_exit_intent` | 退出指令识别 |
| `hass_*` | Home Assistant 集成 |

---

## 开发指南

### 添加新的 ASR Provider

1. 在 `core/providers/asr/` 下创建新文件：
```python
from .base import ASRProviderBase

class MyASR(ASRProviderBase):
    def __init__(self, config):
        super().__init__(config)
        # 初始化

    async def speech_to_text(self, audio_data):
        # 实现语音识别逻辑
        text = await self._do_recognize(audio_data)
        return text
```

2. 在 `core/utils/asr.py` 中添加工厂函数。

3. 在 `config.yaml` 中添加配置项。

### 添加新插件

1. 在 `plugins_func/functions/` 下创建插件文件：
```python
from plugins_func.register import register_function, ToolType, Action, ActionResponse

@register_function("my_plugin", "插件描述", ToolType.SERVER_PLUGIN)
async def my_plugin(param1: str, param2: int = 0):
    """
    插件函数说明
    """
    # 实现插件逻辑
    result = do_something(param1, param2)
    return ActionResponse(action=Action.SUCCESS, result=result)
```

2. 在 `config.yaml` 的 `Intent.function_call.functions` 中启用插件。

### 消息类型

| 类型 | 方向 | 说明 |
|------|------|------|
| `hello` | 设备→服务 | 握手初始化 |
| `listen` | 设备→服务 | 监听模式控制 |
| `audio` | 设备→服务 | Opus 音频数据 |
| `text` | 设备→服务 | 文本消息 |
| `iot` | 双向 | IoT 设备控制 |
| `mcp` | 双向 | MCP 协议消息 |
| `abort` | 设备→服务 | 中断当前操作 |
| `speech` | 服务→设备 | TTS 音频数据 |

---

## 常见问题

### FFmpeg 未找到

**问题**：启动时提示 "FFmpeg not installed"

**解决**：
- Windows：从 https://ffmpeg.org/download.html 下载，解压并添加到 PATH
- Linux：`sudo apt install ffmpeg`
- macOS：`brew install ffmpeg`

### 配置不生效

**问题**：修改了 config.yaml 但没有生效

**解决**：
- 请修改 `data/.config.yaml` 而不是 `config.yaml`
- 或者检查是否启用了智控台配置（`read_config_from_api`）

### TTS 经常报错

**问题**：使用免费 TTS 时经常报错

**解决**：
- 免费 TTS 并发限制较低，建议使用付费方案
- 火山引擎豆包 TTS 起步价 30 元有 100 并发
- 或者使用 `EdgeTTS`（微软免费，稳定）

### 音频停顿过长被截断

**问题**：说话停顿时长较长就被截断

**解决**：
```yaml
VAD:
  SileroVAD:
    min_silence_duration_ms: 500  # 从默认 200 改大
```

### 更多帮助

- 项目 FAQ：[../../docs/FAQ.md](../../docs/FAQ.md)
- 部署文档：[../../docs/Deployment.md](../../docs/Deployment.md)
- 提交 Issue：https://github.com/xinnan-tech/xiaozhi-esp32-server/issues

---

## 许可证

MIT License

---

## 鸣谢

由华南理工大学刘思源教授团队研发。

感谢以下项目和公司的支持：
- [百聆语音对话机器人](https://github.com/wwbin2017/bailing)
- [十方融海](https://www.tenclass.com/)
- [玄凤科技](https://github.com/Eric0308)
- [汇远设计](http://ui.kwd988.net/)
- [西安勤人信息科技](https://www.029app.com/)
