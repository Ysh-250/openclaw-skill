# GreenVideo.cc 自动化脚本开发任务报告

## 项目背景

**目标网站**: https://greenvideo.cc  
**功能**: 免费、无次数限制的视频解析下载，支持抖音、B站、YouTube 等 1000+ 平台

**为什么选择这个网站**:
- 不需要登录/验证码
- 免费、无次数限制
- 已替代之前的天天解析网站（会员次数用完）

**当前状态**: 
- 核心流程已通过人工操作浏览器验证成功 ✅
- 自动化脚本测试失败 ❌（headless 模式超时）

---

## 已验证的核心流程（关键！）

### 完整流程图

```
打开 greenvideo.cc
    ↓
输入视频链接（fill() 直接填入）
    ↓
点击「开始」按钮
    ↓
等待解析（5-10秒）
    ↓
解析成功标志：「下载」按钮出现
    ↓
第一次点击「下载」  ⚠️ 关键步骤
    ↓
按钮变成「点击下载」（loading 2-3秒）
    ↓
第二次点击「点击下载」 ⚠️ 关键步骤
    ↓
打开新标签页
    ↓
新页面 URL = 直链！
```

### 关键发现

| 发现 | 说明 |
|------|------|
| **按钮状态变化机制** | 解析成功后显示「下载」→ 点击后变成「点击下载」 |
| **必须点击两次** | 第一次触发获取直链，第二次打开直链页面 |
| **直链获取方式** | 新标签页的 URL 即是直链，无需额外解析 |
| **输入方式** | `fill()` 直接填入即可，不需要逐字 type |

### 元素定位（已验证）

| 元素 | Playwright 选择器 |
|------|------------------|
| 输入框 | `page.get_by_placeholder("请将复制的视频链接粘贴到此处，并点击开始按钮")` |
| 开始按钮 | `page.get_by_role("button", name="开始")` |
| 下载按钮（状态1） | `page.get_by_role("button", name="下载", exact=True)` |
| 点击下载按钮（状态2） | `page.get_by_role("button", name="点击下载")` |

### 直链 URL 格式

```
https://v{N}-default.365yg.com/{hash}/{hash}/video/tos/cn/.../?mime_type=video_mp4&...
https://v{N}-coldy.douyinvod.com/.../?mime_type=video_mp4&...
```

域名变体：`365yg.com`、`douyinvod.com`（抖音 CDN）

---

## 现有脚本问题分析

### 脚本路径
`skills/media-master/scripts/greenvideo_auto_v2.py`

### 问题：Headless 模式超时

脚本在独立启动的浏览器中（无论 headless 或可视化）超时失败，但在我控制的现有浏览器中每次都成功。

### 可能原因

1. **浏览器环境差异**：
   - 新浏览器没有历史 cookies/session
   - 网站可能检测到自动化浏览器
   
2. **频率限制**：
   - 同一链接反复解析可能触发网站限制
   
3. **输入方式问题**：
   - `fill()` 可能触发不同的 Vue 事件处理
   - 需要模拟真实的粘贴/输入行为

4. **网站检测自动化**：
   - 可能通过浏览器指纹检测
   - 可能检测鼠标移动轨迹

---

## 建议的解决方案

### 方案 1：使用持久化浏览器上下文

```python
# 保存 cookies/session，模拟真实用户
context = playwright.chromium.launch_persistent_context(
    user_data_dir="./browser_profile",
    headless=False,
    args=['--disable-blink-features=AutomationControlled']
)
```

### 方案 2：使用 CDP 连接已打开的浏览器

```python
# 连接到我已经打开的浏览器（已知可用）
browser = playwright.chromium.connect_over_cdp("ws://localhost:18800")
```

### 方案 3：模拟更真实的输入行为

```python
# 1. 先清空
input_box.click()
input_box.fill("")
# 2. 模拟粘贴（触发 Vue 的 paste 事件）
input_box.evaluate("el => { el.value = 'URL'; el.dispatchEvent(new Event('input', { bubbles: true })); }")
```

### 方案 4：增加反检测措施

```python
context = playwright.chromium.launch(
    headless=False,
    args=[
        '--disable-blink-features=AutomationControlled',
        '--disable-features=IsolateOrigins,site-per-process',
    ]
)
# 添加 stealth 脚本
page.add_init_script("""
    Object.defineProperty(navigator, 'webdriver', { get: () => undefined });
""")
```

---

## 测试链接

```
抖音视频: https://v.douyin.com/dQrFcMREc50/
```

---

## 输出要求

脚本应实现：

1. **单链接解析下载**
   ```python
   result = bot.parse_and_download("https://v.douyin.com/xxx/")
   # 返回: {'success': True, 'video_file': 'D:\\downloads\\标题.mp4', 'title': '...'}
   ```

2. **批量处理**
   ```python
   results = bot.batch_process(["url1", "url2", "url3"])
   ```

3. **下载目录**: `D:\downloads`

4. **文件命名**: 使用视频标题（去除特殊字符）

---

## 关键技术点

1. **按钮状态检测**：需要等待「下载」按钮出现后，点击并等待变成「点击下载」

2. **新页面监听**：使用 `context.expect_page()` 或轮询 `context.pages` 检测新打开的页面

3. **下载 Headers**：直链可能需要特定 Referer
   ```python
   headers = {
       "Referer": "https://www.douyin.com/",
       "User-Agent": "Mozilla/5.0 ..."
   }
   ```

4. **错误处理**：
   - 解析超时（链接无效/频率限制）
   - 下载失败（403 防盗链）

---

## 给 AI 编程代理的任务说明

### 任务目标

完善 `greenvideo_auto_v2.py` 脚本，使其能够稳定自动化解析下载视频。

### 你具备的能力

你可以自主操作浏览器测试脚本，这比我只能调用 browser API 更灵活。建议：

1. **先手动验证流程**：用你的浏览器操作能力，走一遍完整流程，确认元素和时机

2. **对比失败日志**：查看脚本失败时浏览器实际状态（截图/HTML）

3. **调试输入方式**：测试 `fill()` vs `type()` vs JavaScript 直接赋值，哪种能触发正确的 Vue 事件

4. **测试持久化上下文**：保存浏览器 profile，避免每次都是"新用户"

### 关键调试点

- 为什么独立浏览器超时？
- 输入后是否正确触发 Vue 事件？
- 是否需要特定的 cookies/session？
- 网站是否有自动化检测？

---

## 相关文件

- 脚本 v2: `skills/media-master/scripts/greenvideo_auto_v2.py`
- 脚本 v1: `skills/media-master/scripts/greenvideo_auto.py`
- TOOLS.md: `workspace/TOOLS.md`（包含网站信息和下载方法）

---

**祝调试成功！核心流程已验证可行，关键是解决浏览器环境差异问题。**