# OpenClaw 配置片段示例

将以下配置合并到你的 `openclaw.json` 文件中。

## 基础配置

```json5
{
  skills: {
    entries: {
      "multimodal-agent": {
        enabled: true,
        // 方式1：直接填写API Key（不推荐，可能泄露）
        // apiKey: "sk-your-api-key-here",
        
        // 方式2：从环境变量获取（推荐）
        apiKey: { source: "env", provider: "default", id: "MEDIA_PROCESSOR_API_KEY" },
        
        // 环境变量注入
        env: {
          MEDIA_PROCESSOR_API_KEY: "",  // 【需填写】或通过系统环境变量设置
          // 可添加更多环境变量
          DASHSCOPE_API_KEY: "",        // 阿里百炼
          MINIMAX_API_KEY: "",          // MiniMax
          OPENAI_API_KEY: "",           // OpenAI
        },
        
        // 具体API配置
        config: {
          // 默认提供商
          default_provider: "aliyun-bailian",  // 【需填写】你使用的提供商
          
          // 超时配置
          timeout: {
            sync: 30,       // 同步API超时（秒）
            poll_interval: 5,  // 异步轮询间隔（秒）
            max_poll_time: 300, // 最大轮询时间（秒）
            download: 120   // 文件下载超时（秒）
          },
          
          // 重试配置
          retry: {
            max_attempts: 3,
            exponential_base: 2  // 指数退避基数
          },
          
          // 输出目录
          output_dir: "${workspace}/output",  // 默认工作区output目录
          
          // 文件清理
          cleanup: {
            enabled: true,
            max_age_hours: 24  // 24小时后清理临时文件
          },
          
          // 各类型提供商配置
          providers: {
            // 【需填写】根据你使用的API填写以下配置
            video: {
              primary: "",      // 【需填写】如 "aliyun-bailian"
              fallbacks: []     // 备选提供商
            },
            image: {
              primary: "",      // 【需填写】如 "openai" 或 "dall-e"
              fallbacks: []
            },
            audio: {
              tts_provider: "", // 【需填写】TTS提供商
              stt_provider: ""  // 【需填写】STT提供商，如 "whisper-local"
            },
            document: {
              mode: "local"     // local 或 api
            }
          }
        }
      }
    }
  }
}
```

---

## 阿里百炼配置示例

如果你使用阿里百炼API：

```json5
{
  skills: {
    entries: {
      "multimodal-agent": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "DASHSCOPE_API_KEY" },
        env: {
          DASHSCOPE_API_KEY: "sk-your-dashscope-key"  // 【需替换】
        },
        config: {
          default_provider: "aliyun-bailian",
          providers: {
            video: {
              primary: "aliyun-bailian",
              endpoints: {
                "aliyun-bailian": {
                  submit: "https://dashscope.aliyuncs.com/api/v1/services/aigc/video-generation/generate",
                  status: "https://dashscope.aliyuncs.com/api/v1/tasks",
                  model: "wan2.7-t2v"
                }
              }
            },
            audio: {
              tts_provider: "aliyun-bailian",
              // CosyVoice 使用 WebSocket 协议（SDK），无 HTTP 端点
              tts_model: "cosyvoice-v3.5-plus",  // cosyvoice-v3-plus 或 cosyvoice-v3.5-plus
              stt_provider: "aliyun-bailian"
            }
          }
        }
      }
    }
  }
}
```

---

## OpenAI配置示例

```json5
{
  skills: {
    entries: {
      "multimodal-agent": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
        env: {
          OPENAI_API_KEY: "sk-your-openai-key"  // 【需替换】
        },
        config: {
          default_provider: "openai",
          providers: {
            image: {
              primary: "openai",
              endpoints: {
                "openai": {
                  endpoint: "https://api.openai.com/v1/images/generations",
                  model: "dall-e-3"
                }
              }
            },
            audio: {
              tts_provider: "openai",
              stt_provider: "whisper-api",
              endpoints: {
                "openai-tts": {
                  endpoint: "https://api.openai.com/v1/audio/speech",
                  model: "tts-1"
                },
                "openai-stt": {
                  endpoint: "https://api.openai.com/v1/audio/transcriptions",
                  model: "whisper-1"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

---

## 本地处理配置

仅使用本地处理（无API）：

```json5
{
  skills: {
    entries: {
      "multimodal-agent": {
        enabled: true,
        config: {
          default_provider: "local",
          providers: {
            document: {
              mode: "local"
            },
            audio: {
              stt_provider: "whisper-local"
            }
          },
          local: {
            whisper_model: "base",  // tiny | base | small | medium | large
            use_gpu: false          // 是否使用GPU加速
          }
        }
      }
    }
  }
}
```

需要安装依赖：
```bash
pip install python-pptx python-docx openpyxl pdfplumber whisper
```

---

## 多提供商混合配置

同时使用多个API：

```json5
{
  skills: {
    entries: {
      "multimodal-agent": {
        enabled: true,
        env: {
          DASHSCOPE_API_KEY: "sk-xxx",    // 视频用阿里
          OPENAI_API_KEY: "sk-yyy",       // 图片/语音用OpenAI
        },
        config: {
          default_provider: "mixed",
          providers: {
            video: {
              primary: "aliyun-bailian",
              fallbacks: ["minimax"]
            },
            image: {
              primary: "openai",
              fallbacks: ["dall-e"]
            },
            audio: {
              tts_provider: "openai",
              stt_provider: "whisper-local"
            },
            document: {
              mode: "local"
            }
          }
        }
      }
    }
  }
}
```

---

## Agent配置

确保agent能使用此skill：

```json5
{
  agents: {
    defaults: {
      skills: ["multimodal-agent"],  // 或在具体agent中配置
      // 其他agent配置...
    },
    list: [
      {
        id: "main",
        skills: ["multimodal-agent", "other-skills..."]  // main agent添加此skill
      }
    ]
  }
}
```

---

## 配置生效步骤

1. 编辑 `C:\Users\admin\.openclaw-new\.openclaw\openclaw.json`
2. 将上述配置片段合并到对应位置
3. 重启OpenClaw Gateway：
   ```bash
   openclaw gateway restart
   ```
4. 或等待自动热重载（如果启用config watch）

---

_配置完成后，agent即可通过multimodal-agent skill处理各类媒体任务。_