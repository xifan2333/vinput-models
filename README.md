# vinput-models

fcitx5-vinput 的 ASR 模型注册表。CLI 工具通过此仓库的 `registry.json` 获取可下载的模型列表。

## registry.json 格式

```json
[
  {
    "name": "model-name",
    "display_name": "显示名称",
    "description": "模型描述",
    "url": ["https://...tar.bz2"],
    "sha256": "校验和",
    "size_bytes": 12345678,
    "model_type": "sense_voice",
    "language": "zh",
    "vinput_model": { ... }
  }
]
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 模型唯一标识，用作本地目录名 |
| `display_name` | string | 用户可读的显示名称 |
| `description` | string | 模型简要描述 |
| `url` | string[] | 下载地址列表（tar.bz2 归档） |
| `sha256` | string | 归档文件 SHA-256 校验和 |
| `size_bytes` | number | 归档文件大小（字节） |
| `model_type` | string | 模型类型标识（见下方） |
| `language` | string | 模型支持的语言（仅用于展示） |
| `vinput_model` | object | 安装后写入 `vinput-model.json` 的内容 |

## vinput_model 规范

`vinput_model` 对象在模型安装后写入本地 `vinput-model.json`，ASR 引擎据此加载模型。

结构：

```json
{
  "model_type": "<type>",
  "files": { ... },
  "params": { ... }
}
```

- `model_type` — 模型类型，决定 sherpa-onnx 加载方式
- `files` — 模型文件路径（相对于模型目录），`tokens` 为所有类型必需
- `params` — 模型特定参数，无需配置时留空 `{}`

> ASR 识别语言由用户配置 `config.json` 中的 `default_language` 字段控制，不在模型元数据中指定。

## 支持的模型类型

### 单文件模型

适用于：`dolphin`、`paraformer`、`nemo_ctc`、`wenet_ctc`、`tdnn`、`zipformer_ctc`、`telespeech_ctc`、`omnilingual`、`medasr`、`fire_red_asr_ctc`

```json
{
  "model_type": "<type>",
  "files": {
    "model": "<model>.onnx",
    "tokens": "tokens.txt"
  },
  "params": {}
}
```

`paraformer` 可选参数：

```json
"params": { "modeling_unit": "cjkchar" }
```

`modeling_unit` 可选值：`cjkchar`（默认）、`bpe`、`cjkchar+bpe`

### sense_voice

```json
{
  "model_type": "sense_voice",
  "files": {
    "model": "<model>.onnx",
    "tokens": "tokens.txt"
  },
  "params": {
    "use_itn": true
  }
}
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `use_itn` | bool | 智能文本规范化（数字、日期等转换） |

### whisper

```json
{
  "model_type": "whisper",
  "files": {
    "encoder": "<encoder>.onnx",
    "decoder": "<decoder>.onnx",
    "tokens": "tokens.txt"
  },
  "params": {
    "tail_paddings": -1,
    "enable_token_timestamps": false,
    "enable_segment_timestamps": false
  }
}
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `tail_paddings` | int | `-1` | 尾部填充 |
| `enable_token_timestamps` | bool | `false` | 基于 DTW 的 token 级时间戳 |
| `enable_segment_timestamps` | bool | `false` | Whisper 原生段级时间戳 |

### moonshine

```json
{
  "model_type": "moonshine",
  "files": {
    "preprocessor": "<preprocessor>.onnx",
    "encoder": "<encoder>.onnx",
    "uncached_decoder": "<uncached_decoder>.onnx",
    "cached_decoder": "<cached_decoder>.onnx",
    "merged_decoder": "<merged_decoder>.onnx"
  },
  "params": {}
}
```

`merged_decoder` 为可选字段。

### zipformer_transducer

```json
{
  "model_type": "zipformer_transducer",
  "files": {
    "encoder": "<encoder>.onnx",
    "decoder": "<decoder>.onnx",
    "joiner": "<joiner>.onnx",
    "tokens": "tokens.txt"
  },
  "params": {}
}
```

### fire_red_asr

```json
{
  "model_type": "fire_red_asr",
  "files": {
    "encoder": "<encoder>.onnx",
    "decoder": "<decoder>.onnx",
    "tokens": "tokens.txt"
  },
  "params": {}
}
```

### canary

```json
{
  "model_type": "canary",
  "files": {
    "encoder": "<encoder>.onnx",
    "decoder": "<decoder>.onnx",
    "tokens": "tokens.txt"
  },
  "params": {
    "tgt_lang": "zh",
    "use_pnc": true
  }
}
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `tgt_lang` | string | 目标语言，默认同 `default_language` |
| `use_pnc` | bool | 启用标点和大小写 |

### funasr_nano

```json
{
  "model_type": "funasr_nano",
  "files": {
    "encoder_adaptor": "<encoder_adaptor>.onnx",
    "llm": "<llm>.onnx",
    "embedding": "<embedding>.onnx",
    "tokenizer": "<tokenizer>.txt"
  },
  "params": {
    "use_itn": true,
    "max_new_tokens": 1024,
    "temperature": 1.0,
    "top_p": 0.9,
    "seed": 0,
    "system_prompt": "",
    "user_prompt": "",
    "hotwords": ""
  }
}
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `use_itn` | bool | `false` | 智能文本规范化 |
| `max_new_tokens` | int | `1024` | 最大生成 token 数 |
| `temperature` | float | `1.0` | 采样温度 |
| `top_p` | float | `0.9` | Top-p 采样 |
| `seed` | int | `0` | 随机种子 |
| `system_prompt` | string | `""` | 系统提示词（可选） |
| `user_prompt` | string | `""` | 用户提示词（可选） |
| `hotwords` | string | `""` | 热词（可选） |

## 通用可选文件

以下文件适用于所有模型类型，按需添加到 `files` 中：

```json
"files": {
  "bpe_vocab": "bpe.vocab",
  "lm": "lm.onnx",
  "hotwords_file": "hotwords.txt",
  "rule_fsts": "rule.fst",
  "rule_fars": "rule.far"
}
```

| 文件 | 说明 |
|------|------|
| `bpe_vocab` | BPE 词表 |
| `lm` | 语言模型，配合 `params.lm_scale`（默认 0.5）使用 |
| `hotwords_file` | 热词文件 |
| `rule_fsts` | FST 规则文件 |
| `rule_fars` | FAR 规则文件 |
