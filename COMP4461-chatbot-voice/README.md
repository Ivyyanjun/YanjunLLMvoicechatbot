# 🎙️ 语音聊天机器人：初学者运行指南

这个项目是一个能**用嘴说话**的聊天机器人：你对着麦克风说话，它把语音转成文字，交给大模型回答，再把回答读出来。

本指南会一步一步带你在自己的电脑上把它运行起来，**Mac 和 Windows 都适用**。凡是两种系统命令不同的地方，都会分开写。

> ⏱️ 第一次安装大约需要 20–30 分钟，网速慢的话会更久。以后每次运行只需要 3 行命令。

---

## 开始之前：你需要准备

| 需要什么 | 说明 |
|---|---|
| **Python 3.11** | ⚠️ 必须是 3.11，不要装最新版。安装方法见第 0 步 |
| **Git** | 用来下载代码 |
| **API Key** | 在 [CloseAI](https://platform.closeai-asia.com/dashboard) 注册并创建，形如 `sk-...` |
| **麦克风和扬声器** | 笔记本自带的就可以 |
| **能访问 Google 的网络** | 语音识别用的是 Google 的在线服务 |

---

## 第 0 步：安装 Python 3.11 和 Git（只做一次）

**已经装过的可以跳过。** 在终端里输入 `python3.11 --version`（Mac）或 `py -3.11 --version`（Windows），显示 `Python 3.11.x` 就说明已经装好了。

### Mac

1. 打开 <https://www.python.org/downloads/release/python-3119/>，拉到页面底部，下载 **macOS 64-bit universal2 installer**，双击安装。
2. Git：打开“终端”，输入 `git --version`。如果弹窗提示安装开发者工具，点“安装”即可。

### Windows

1. 打开 <https://www.python.org/downloads/release/python-3119/>，拉到页面底部，下载 **Windows installer (64-bit)**。
   安装第一页**一定要勾选 ☑️ Add python.exe to PATH**，然后点 Install Now。
2. Git：从 <https://git-scm.com/download/win> 下载安装，所有选项保持默认。
3. 打开 **PowerShell**，运行下面这条命令。它允许 PowerShell 运行脚本，只需要做一次：

   ```powershell
   Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
   ```

   如果问是否确认，输入 `Y` 回车。

> 💡 装完软件后，要**关掉终端再重新打开**，新装的程序才能被识别。

---

## 第 1 步：下载代码

打开终端（Mac 用“终端”，Windows 用“PowerShell”），运行：

```bash
cd ~
git clone https://github.com/assemzh/COMP4461-chatbot-voice.git
cd COMP4461-chatbot-voice
```

> ⚠️ **想好放在哪里再下载。** 下一步建好虚拟环境后，**不要再移动这个文件夹**，否则会出现 `command not found: streamlit` 的错误（解决方法见文末“常见问题”）。

---

## 第 2 步：建立虚拟环境

虚拟环境是这个项目**专用的工具箱**，和你电脑上的其他项目互不干扰。

| | Mac | Windows |
|---|---|---|
| 建立 | `python3.11 -m venv .venv` | `py -3.11 -m venv .venv` |
| 激活 | `source .venv/bin/activate` | `.venv\Scripts\Activate.ps1` |

✅ **激活成功的标志**：命令行最前面出现 `(.venv)`。

再确认一下版本：

```bash
python --version
```

必须显示 `Python 3.11.x`。

> 💡 即使你之前做过别的项目（比如 COMP4461-chatbot），这里也要**重新建一个**虚拟环境。每个项目各用各的。

---

## 第 3 步：安装需要的工具包

### 3.1（仅 Mac）安装麦克风支持

```bash
brew install portaudio
```

如果提示 `brew: command not found`，先到 <https://brew.sh> 按首页的说明安装 Homebrew。**Windows 不需要这一步。**

### 3.2 安装 Python 工具包（Mac 和 Windows 相同）

确认命令行前面有 `(.venv)`，然后运行这一行：

```bash
pip install --default-timeout=120 -r requirements.txt
```

`requirements.txt` 是这个项目的“购物清单”，里面列了需要安装的 5 个工具包：

| 工具包 | 作用 |
|---|---|
| `openai` | 连接大模型 |
| `streamlit` | 生成网页聊天界面 |
| `pyaudio` | 读取麦克风 |
| `SpeechRecognition` | 把说的话转成文字 |
| `pyttsx3` | 把文字读出来 |

✅ 最后一行出现 `Successfully installed ...`，并且没有红色的 `ERROR`，就是装好了。

> 🐢 **下载太慢或报 `Read timed out`？** 换用阿里云镜像，会快很多：
>
> ```bash
> pip install --default-timeout=120 -i https://mirrors.aliyun.com/pypi/simple/ -r requirements.txt
> ```
>
> 💡 在 Mac 上安装时，会看到一长串 `pyobjc-framework-...` 在下载。这是正常的，是朗读功能需要的系统组件，耐心等待即可。

---

## 第 4 步：修改代码，连接大模型

原代码连接的是学校的 Azure 服务，需要改成 CloseAI。

1. 用 VS Code 或任意文本编辑器打开 `simple_chatbot_voice.py`。
2. 找到 `chat_with_openai()` 函数里的这一段：

   ```python
           client = AzureOpenAI(
        
   ```

3. 整段替换成：

   ```python
           client = OpenAI(
               api_key=openai_api_key,
               base_url="https://api.openai-proxy.org/v1",
           )
   ```

4. **保存文件**（Mac 按 `Cmd + S`，Windows 按 `Ctrl + S`）。

> 💡 **想用 DeepSeek？** 把 `base_url` 改成 `"https://api.deepseek.com"`，再把文件第 8 行改成 `model_name = "deepseek-flash"`，然后在侧边栏填 DeepSeek 的 API Key。

---

## 第 5 步：运行

确认命令行前面有 `(.venv)`，然后运行：

```bash
streamlit run simple_chatbot_voice.py
```

- 第一次运行可能让你输入邮箱，**直接按回车跳过**。
- 浏览器会自动打开 <http://localhost:8501>。

### 怎么使用

1. 在网页**左侧边栏**粘贴你的 API Key。
2. 选择 **Voice**，点击 **Speak**，然后**马上**对着麦克风说一句**英文**，比如 “What is an AI agent?”
3. 等一下，机器人会显示回答，并把它读出来。

> 🎤 **第一次点 Speak 时**，系统会询问是否允许使用麦克风，请点 **允许**。
> 如果之前点了拒绝：
> - Mac：系统设置 → 隐私与安全性 → 麦克风，打开“终端”的开关
> - Windows：设置 → 隐私和安全性 → 麦克风，打开“允许桌面应用访问麦克风”

**停止程序**：回到终端，按 `Ctrl + C`。

---

## 以后每次运行

安装只需要做一次。以后每次打开电脑，只需要 3 行命令：

**Mac：**

```bash
cd ~/COMP4461-chatbot-voice
source .venv/bin/activate
streamlit run simple_chatbot_voice.py
```

**Windows：**

```powershell
cd ~\COMP4461-chatbot-voice
.venv\Scripts\Activate.ps1
streamlit run simple_chatbot_voice.py
```

> 记住：**每次新开一个终端窗口，都要重新激活虚拟环境。**

---

## 常见问题

| 看到的情况 | 原因 | 解决方法 |
|---|---|---|
| `command not found: streamlit`（但前面有 `(.venv)`） | 项目文件夹建好环境后被移动过 | 把文件夹移回原位置；或者在新位置运行 `deactivate`、`rm -rf .venv`，然后从第 2 步重新做（重装会比第一次快） |
| `command not found: streamlit`（前面没有 `(.venv)`） | 没激活虚拟环境 | 运行第 2 步的激活命令 |
| `Read timed out` / `ReadTimeoutError` | 网速太慢，下载中断 | 用第 3 步里的阿里云镜像命令重新安装 |
| `nodename nor servname provided` | 连不上下载地址 | 检查网络；换一个镜像，或开关 VPN 后再试 |
| `No module named 'distutils'` | Python 版本太新（3.12 及以上） | 删掉 `.venv`，用 Python 3.11 重建 |
| `source : 无法将“source”项识别为…`（Windows） | 用了 Mac 的命令 | 改用 `.venv\Scripts\Activate.ps1` |
| `无法加载文件 …Activate.ps1，因为在此系统上禁止运行脚本`（Windows） | PowerShell 默认禁止运行脚本 | 运行 `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` |
| `401 … invalid subscription key` | 代码还在连接学校的 Azure | 按第 4 步改代码，**保存**，然后按 `Ctrl + C` 停掉程序再重新运行 |
| `401 … Incorrect API key` | API Key 填错了 | 重新复制 Key，注意前后不要带空格 |
| `Unknown Value Error` | 没听清你说的话 | 点 Speak 后马上开口，说得清楚一点，用英文 |
| `Wait Timeout Error. No speech detected` | 5 秒内没检测到声音 | 检查麦克风权限和系统的输入设备 |
| 话说到一半就被截断 | 稍微停顿一下，程序就以为你说完了 | 见下方“小改进 1” |
| 机器人有文字回复，但没有声音（Mac） | 朗读工具在 Mac 上有时不出声 | 先在终端运行 `say "hello"` 测试 Mac 能否发声；能发声的话，见下方“小改进 2” |

还是解决不了？请把终端里**完整的报错文字**截图或复制下来，发给助教。

---

## 可选的小改进

### 小改进 1：说话中间停顿时不被截断

在 `speech_to_text()` 函数里，`recognizer = sr.Recognizer()` 这一行的**下一行**加上：

```python
    recognizer.pause_threshold = 1.5   # 停顿超过 1.5 秒才算说完（默认 0.8 秒）
```

### 小改进 2：Mac 上没有朗读声音

把整个 `text_to_speech()` 函数替换成下面这段。Mac 上会改用系统自带的 `say` 命令朗读，Windows 上保持原来的方式：

```python
import subprocess
import sys

def text_to_speech(text):
    if sys.platform == "darwin":                      # Mac：用系统自带的 say 命令
        subprocess.run(["say", "-r", "180", text])    # -r 180 是语速
        return
    tts_engine = pyttsx3.init()                       # Windows：用 pyttsx3
    tts_engine.setProperty('rate', 150)
    tts_engine.say(text)
    tts_engine.runAndWait()
    if tts_engine._inLoop:
        tts_engine.endLoop()
```

### 小改进 3：识别中文

代码默认只识别英文。想说中文的话，把 `speech_to_text()` 里的

```python
return recognizer.recognize_google(audio)
```

改成：

```python
return recognizer.recognize_google(audio, language='zh-CN')   # 普通话；粤语用 'zh-HK'
```

---

## 注意事项

- 🔑 **不要把 API Key 写进代码里，再上传到 GitHub。** 运行时在网页侧边栏粘贴就好。
- 📁 **建好虚拟环境后，不要移动项目文件夹。**
- 🧰 **每个项目用自己的虚拟环境**，不要几个项目共用一个。
- 📄 项目里的 `tts_module.py` 是 Mac 专用的朗读示例，**不需要运行它**，它不影响主程序。
