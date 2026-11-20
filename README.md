# 游戏外服语音双向实时翻译（Windows · 本地免费 · 悬浮窗）

打外服游戏时：

- **听队友**：把队友从耳机/扬声器里说出来的外语，实时转成中文；
- **说回去**：你对着麦克风说中文，自动译成队友的语言，一键复制粘贴到游戏聊天框，或直接朗读出来。

语音识别全部在本机完成（`faster-whisper`），**不上传音频**；只有识别出的文字会送到翻译接口。

***

## 一、它怎么工作

```
队友语音 ──(扬声器回环)──┐
                        ├── Whisper 本地识别 ── 翻译 ── 悬浮窗 / 控制台
我说话   ──(麦克风)──────┘                          └─ 复制 / 朗读
```

1. WASAPI 回环捕获当前默认播放设备里正在播放的声音（即队友语音）；
2. 麦克风单独采集你的中文；
3. 两路都按「静音停顿」自动切句，交给同一个 Whisper 模型识别；
4. 队友方向 外语→中文；我方方向 中文→队友语言，结果实时写入悬浮窗并落地存档。

> 适用：游戏内语音、Discord / 腾讯语音 / YY / Steam 语音等，声音走默认播放设备即可。

***

## 二、安装（首次约 10–20 分钟，主要在下载模型）

### 1. 安装 Python

Python **3.10 / 3.11 / 3.12**，安装时勾选 **Add Python to PATH**。
装完 `Win + R` → `cmd` → `python --version` 能看到版本号即成功。

### 2. 放好文件

把整个文件夹放到好找的位置，例如 `D:\game-voice-translator`。目录结构：

```
game-voice-translator\
├─ game_translator.py   主程序
├─ config.json          配置（改这个，不改代码）
├─ requirements.txt     依赖
├─ 启动.bat             双击启动
└─ history\             运行后自动生成：历史记录
```

### 3. 安装依赖

在该文件夹的地址栏输入 `cmd` 回车（当前目录打开命令行），执行：

```
pip install -r requirements.txt
```

### 4. 启动

```
:: 双击「启动.bat」，或手动：
python game_translator.py

:: 纯命令行模式（不开窗口）
python game_translator.py --console
```

首次运行会自动下载 Whisper 模型（`base` 约 150MB），之后走本地缓存。

***

## 三、悬浮窗怎么用

| 控件 | 作用 |
| --- | --- |
| **暂停 / 继续** | 暂停后队友与我方音频都不再识别、不再写记录（例如你要打电话时） |
| **清空** | 只清当前窗口的显示，**已写入 history 的记录不会丢** |
| **保存** | 把本次会话导出为 `history/记录_时间.md`（人可读的 Markdown） |
| **历史** | 打开历史窗口，可回看最近 400 条记录，并提供「打开记录文件夹」 |
| **🎤 点我说中文** | 点一下开始录音，停顿约 0.8 秒自动结束并翻译；录音中按钮变红，再点一次可提前结束 |
| **📋 复制** | 把最新一句我方译文复制到剪贴板，直接粘贴进游戏聊天框 |
| **🔊 朗读** | 用 Windows 自带 SAPI 语音把译文读出来（不需要额外装东西） |
| **自动朗读** | 勾上后，每说一句自动朗读，适合希望队友直接听到外放的场景 |

<!--editor-kit-table-meta=%7B%22version%22%3A1%2C%22cells%22%3A%5B%7B%22row%22%3A3%2C%22col%22%3A1%2C%22ops%22%3A%5B%7B%22insert%22%3A%22*%22%2C%22attributes%22%3A%7B%22lmkr%22%3A%221%22%7D%7D%2C%7B%22insert%22%3A%22%E6%8A%8A%E6%9C%AC%E6%AC%A1%E4%BC%9A%E8%AF%9D%E5%AF%BC%E5%87%BA%E4%B8%BA%20%22%7D%2C%7B%22insert%22%3A%22history%2F%E8%AE%B0%E5%BD%95_%E6%97%B6%E9%97%B4.md%22%2C%22attributes%22%3A%7B%22inlineCode%22%3A%22true%22%7D%7D%2C%7B%22insert%22%3A%22%EF%BC%88%E4%BA%BA%E5%8F%AF%E8%AF%BB%E7%9A%84%20Markdown%EF%BC%89%5Cn%22%7D%5D%7D%5D%7D-->

- 窗口**始终置顶**，按住标题栏或空白处可拖动到不挡画面的位置；
- 支持 **F8 一键说话**（需 `pip install keyboard`，部分环境要管理员权限才生效）。

***

## 四、配置说明（`config.json`）

```
{
  "whisper_model": "base",          // tiny / base / small / medium / large-v3
  "translator_backends": ["mymemory", "google"],  // 依次尝试，前一个失败自动换下一个
  "teammate": {
    "source_language": "",          // 队友语言：留空自动检测；确定就填 en / ja / ko / ru
    "target_language": "zh-CN"
  },
  "me": {
    "source_language": "zh",
    "target_language": "en"         // ★ 队友说什么语言就填什么：ja / ko / ru / es ...
  },
  "audio": {
    "silence_rms": 0.012,           // 静音阈值：环境吵调大，轻声听不到调小
    "silence_end_sec": 0.8,         // 停顿多久算一句
    "max_chunk_sec": 8.0,
    "mic_max_sec": 15.0             // 我说一句最长录音
  },
  "tts": { "auto": false, "rate": 0 },   // 自动朗读 / 语速（-10~10）
  "hotkey": { "enabled": true, "key": "f8" },
  "ui": { "always_on_top": true, "alpha": 0.95, "max_lines": 60 }
}
```

**最重要的两项**：`me.target_language` 填成队友的语言（默认英文），`teammate.source_language` 如果固定填上（如 `"ja"`）识别更快更准。

### 翻译后端怎么选

| 后端 | 是否需要 | 说明 |
| --- | --- | --- |
| `mymemory` | 免费、免注册 | 默认首选，国内一般可直连；**免费额度有限（约每天上千词）**，长期用建议配百度 |
| `google` | 免费 | 需要能访问谷歌翻译；国内多数网络会失败，脚本会自动跳过 |
| `baidu` | 需注册 key | 国内最稳。到百度翻译开放平台申请「通用文本翻译」，把 `baidu_appid`、`baidu_key` 填进 config，并把 `"baidu"` 加进 `translator_backends` |
| `argos` | 本地离线 | 完全不联网，需 `pip install argostranslate` + 装语言包，质量略弱于在线 |

<!--editor-kit-table-meta=%7B%22version%22%3A1%2C%22cells%22%3A%5B%7B%22row%22%3A3%2C%22col%22%3A2%2C%22ops%22%3A%5B%7B%22insert%22%3A%22*%22%2C%22attributes%22%3A%7B%22lmkr%22%3A%221%22%7D%7D%2C%7B%22insert%22%3A%22%E5%9B%BD%E5%86%85%E6%9C%80%E7%A8%B3%E3%80%82%E5%88%B0%E7%99%BE%E5%BA%A6%E7%BF%BB%E8%AF%91%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0%E7%94%B3%E8%AF%B7%E3%80%8C%E9%80%9A%E7%94%A8%E6%96%87%E6%9C%AC%E7%BF%BB%E8%AF%91%E3%80%8D%EF%BC%8C%E6%8A%8A%20%22%7D%2C%7B%22insert%22%3A%22baidu_appid%22%2C%22attributes%22%3A%7B%22inlineCode%22%3A%22true%22%7D%7D%2C%7B%22insert%22%3A%22%E3%80%81%22%7D%2C%7B%22insert%22%3A%22baidu_key%22%2C%22attributes%22%3A%7B%22inlineCode%22%3A%22true%22%7D%7D%2C%7B%22insert%22%3A%22%20%E5%A1%AB%E8%BF%9B%20config%EF%BC%8C%E5%B9%B6%E6%8A%8A%20%22%7D%2C%7B%22insert%22%3A%22%5C%22baidu%5C%22%22%2C%22attributes%22%3A%7B%22inlineCode%22%3A%22true%22%7D%7D%2C%7B%22insert%22%3A%22%20%E5%8A%A0%E8%BF%9B%20%22%7D%2C%7B%22insert%22%3A%22translator_backends%22%2C%22attributes%22%3A%7B%22inlineCode%22%3A%22true%22%7D%7D%2C%7B%22insert%22%3A%22%5Cn%22%7D%5D%7D%2C%7B%22row%22%3A4%2C%22col%22%3A2%2C%22ops%22%3A%5B%7B%22insert%22%3A%22*%22%2C%22attributes%22%3A%7B%22lmkr%22%3A%221%22%7D%7D%2C%7B%22insert%22%3A%22%E5%AE%8C%E5%85%A8%E4%B8%8D%E8%81%94%E7%BD%91%EF%BC%8C%E9%9C%80%20%22%7D%2C%7B%22insert%22%3A%22pip%20install%20argostranslate%22%2C%22attributes%22%3A%7B%22inlineCode%22%3A%22true%22%7D%7D%2C%7B%22insert%22%3A%22%20%2B%20%E8%A3%85%E8%AF%AD%E8%A8%80%E5%8C%85%EF%BC%8C%E8%B4%A8%E9%87%8F%E7%95%A5%E5%BC%B1%E4%BA%8E%E5%9C%A8%E7%BA%BF%5Cn%22%7D%5D%7D%5D%7D-->

#### 配置百度后端（推荐长期使用）

1. 注册 https://api.fanyi.baidu.com/ ，开通「通用文本翻译」，拿到 APPID 与密钥；
2. `config.json` 里：

```json editor-kit-meta=%7B%22version%22%3A1%2C%22markerAttributes%22%3A%7B%22list%22%3A%22number1%22%2C%22start%22%3A%222%22%2C%22ol-id%22%3A%22ihR9mstZ%22%7D%7D
"translator_backends": ["baidu", "mymemory", "google"],
"baidu_appid": "你的APPID",
"baidu_key": "你的密钥"
```

#### 完全离线（argos）

```
pip install argostranslate
argos-translate-cli --from-lang en --to-lang zh    :: 队友语言 → 中文
argos-translate-cli --from-lang zh --to-lang en    :: 中文 → 队友语言
```

然后 `"translator_backends": ["argos"]`。

***

## 五、常见问题

**Q：抓不到队友声音 / 提示「无法打开扬声器回环」**

- 确认游戏语音走的是**当前默认播放设备**（右下角音量图标 → 声音设置核对）；
- 老声卡需在「声音 → 录制」启用「立体声混音 / Stereo Mix」；
- 用蓝牙耳机时，部分驱动不支持回环，可先切有线/扬声器排查。

**Q：麦克风按钮点了没反应**

- 检查 Windows「设置 → 隐私 → 麦克风」允许桌面应用使用麦克风；
- 确认系统默认输入设备是你的耳麦，而不是虚拟声卡。

**Q：识别慢、字幕跟不上**

- `whisper_model` 降一档（`small` → `base` → `tiny`）；
- 关闭高占用程序；识别跑 CPU，显卡不参与。

**Q：翻译失败 / 显示「[翻译失败]」**

- 多后端都被拒了：多半是 MyMemory 当天额度用尽或网络问题 → 配百度 key，或换 `argos` 离线；
- 错误信息里会写明是哪个后端、什么原因。

**Q：复制按钮复制的是空**

- 需要先成功说一句并拿到译文；按钮复制的是**最新一条我方译文**。

**Q：同一句反复出现**

- 已做相似度去重；仍重复时调高 `dedup_ratio`（如 0.95），或调大 `silence_end_sec`。

**Q：我能听到自己说话被翻译两次吗**

- 队友方向走的是「系统播放的声音」，你说话走的是「麦克风」。若你用**外放**且队友语音也被麦克风收到，可能重复，建议戴耳机。

***

## 六、隐私与说明

- 语音识别在本地完成，音频不出本机；
- 只有**识别出的文字**会发送给翻译服务（MyMemory / Google / 百度按所选后端），介意时切 `argos` 离线；
- 历史记录存在本地 `history/`，按会话一个 `jsonl`，可随时删除；
- 机器翻译有误差，关键信息（时间、坐标、人数）请以游戏内实际沟通为准。

> （注：部分内容由豆包 AI 生成）
