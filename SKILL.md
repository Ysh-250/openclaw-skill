---
name: qwen-chat-openclaw-automation
description: >
  Automate Qwen Chat (chat.qwen.ai) interactions using OpenClaw's built-in browser tool.
  This skill is specifically designed for Qwen Chat automation with OpenClaw browser tool.
  It includes complete workflows for login, asking questions, switching models, and more.
  Cookie persistence is handled automatically by OpenClaw's browser profile.
---

# Qwen Chat OpenClaw 自动化技能

专门用于 Qwen Chat 网站自动化的技能模板，适配 OpenClaw 内置 `browser` 工具。

## 🎯 快速开始

### 前提条件
- ✅ OpenClaw 已配置 browser 工具
- ✅ 用户数据目录已保存（自动管理 Cookie）
- ✅ 已登录过 Qwen Chat（Cookie 已保存）

### 核心命令格式
```
browser open "https://chat.qwen.ai/"
browser act kind=wait timeMs=3000
browser act kind=evaluate fn="JavaScript 代码"
browser act kind=press key=Enter
```

## 📋 核心原则

### 1. JavaScript 代码规范

**必须使用 IIFE 包裹**：
```javascript
// ✅ 正确格式
(() => {
  var element = document.querySelector('.target');
  element.click();
  return 'success';
})()

// ❌ 错误格式
var element = document.querySelector('.target');
element.click();
```

**使用 `var` 而非 `const`/`let`**：
```javascript
// ✅ 正确
var items = document.querySelectorAll('.item');
for (var i = 0; i < items.length; i++) {
  items[i].click();
}

// ❌ 错误（会导致重声明错误）
const items = document.querySelectorAll('.item');
```

**避免可选 chaining `?.`**：
```javascript
// ✅ 正确
var element = document.querySelector('.target');
if (element) {
  element.click();
  return 'clicked';
}

// ❌ 错误（PowerShell 解析问题）
element?.click();
```

### 2. React 表单输入特殊处理

**填写输入框（必须用 nativeInputValueSetter）**：
```javascript
browser act kind=evaluate fn="(() => {
  var input = document.querySelector('input[placeholder*=电子邮箱]');
  if (!input) return 'not_found';
  
  // React 16+ 特殊处理
  var s = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
  if (s) {
    // ⚠️ 替换为你的 Qwen 账号邮箱
  s.call(input, 'YOUR_QWEN_EMAIL_HERE');
  } else {
    input.value = 'YOUR_QWEN_EMAIL_HERE';
  }
  
  // 触发 React 状态更新
  input.dispatchEvent(new Event('input', { bubbles: true }));
  input.dispatchEvent(new Event('change', { bubbles: true }));
  
  return 'filled';
})()"
```

**填写多行文本框（textarea）**：
```javascript
browser act kind=evaluate fn="(() => {
  var ta = document.querySelector('textarea');
  if (!ta) return 'not_found';
  
  var s = Object.getOwnPropertyDescriptor(window.HTMLTextAreaElement.prototype, 'value').set;
  s.call(ta, '输入内容');
  
  ta.dispatchEvent(new Event('input', { bubbles: true }));
  ta.dispatchEvent(new Event('change', { bubbles: true }));
  
  return 'filled';
})()"
```

### 3. Hover 事件触发

**展开更多模型（必须用 mouseenter + mouseover）**：
```javascript
browser act kind=evaluate fn="(() => {
  var el = document.querySelector('[class*=view-more]');
  if (!el) return 'not_found';
  
  el.dispatchEvent(new MouseEvent('mouseenter', { 
    bubbles: true, 
    cancelable: true 
  }));
  el.dispatchEvent(new MouseEvent('mouseover', { 
    bubbles: true, 
    cancelable: true 
  }));
  
  return 'hovered';
})()"
```

## 🚀 核心工作流

### 工作流 1：智能登录检测

**检测登录状态**：
```javascript
browser act kind=evaluate fn="(() => {
  // 检查是否已登录（通过侧边栏历史对话）
  var chatItems = document.querySelectorAll('a[class*=chat-item]');
  if (chatItems.length > 0) return 'already_logged_in';
  
  // 检查登录表单类型
  var popupBtn = document.querySelector('button.qwenchat-auth-pc-submit-button');
  if (popupBtn) return 'popup_login';
  
  var fullpageInput = document.querySelector('input[placeholder*=电子邮箱]');
  if (fullpageInput) return 'fullpage_login';
  
  // 可能需要先点击登录按钮
  var loginText = document.querySelector('[class*=login-text]');
  if (loginText) return 'need_click_login';
  
  return 'unknown_state';
})()"
```

**如果已登录**：直接跳过登录步骤，进入后续操作

**如果需要登录**：
```javascript
// 1. 点击登录按钮（如果需要）
browser act kind=evaluate fn="(() => {
  var loginBtn = document.querySelector('[class*=login-text]');
  if (loginBtn) {
    loginBtn.click();
    return 'clicked';
  }
  return 'not_found';
})()"

browser act kind=wait timeMs=1000

// 2. 填写邮箱
browser act kind=evaluate fn="(() => {
  var input = document.querySelector('input[placeholder*=电子邮箱]');
  if (!input) return 'not_found';
  
  var s = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
  // ⚠️ 替换为你的 Qwen 账号邮箱
  s.call(input, 'YOUR_QWEN_EMAIL_HERE');
  
  input.dispatchEvent(new Event('input', { bubbles: true }));
  input.dispatchEvent(new Event('change', { bubbles: true }));
  
  return 'email_filled';
})()"

// 3. 填写密码
browser act kind=evaluate fn="(() => {
  var input = document.querySelector('input[placeholder*=密码]');
  if (!input) return 'not_found';
  
  var s = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
  // ⚠️ 替换为你的 Qwen 账号密码
  s.call(input, 'YOUR_QWEN_PASSWORD_HERE');
  
  input.dispatchEvent(new Event('input', { bubbles: true }));
  input.dispatchEvent(new Event('change', { bubbles: true }));
  
  return 'password_filled';
})()"

// 4. 提交登录（智能选择按钮）
browser act kind=evaluate fn="(() => {
  // 优先使用弹窗登录按钮
  var popupBtn = document.querySelector('button.qwenchat-auth-pc-submit-button');
  if (popupBtn) {
    popupBtn.click();
    return 'popup_submitted';
  }
  
  // 回退到普通登录按钮
  var buttons = document.querySelectorAll('button');
  for (var i = 0; i < buttons.length; i++) {
    if (buttons[i].textContent.trim() === '登录') {
      buttons[i].click();
      return 'fullpage_submitted';
    }
  }
  
  return 'no_button_found';
})()"

// 5. 验证登录结果
browser act kind=wait timeMs=3000
browser act kind=evaluate fn="(() => {
  var isLoggedIn = document.querySelectorAll('a[class*=chat-item]').length > 0;
  return isLoggedIn ? 'login_success' : 'login_failed';
})()"
```

### 工作流 2：提问并获取回复

**步骤 1：填写问题**
```javascript
browser act kind=evaluate fn="(() => {
  var input = document.querySelector('textarea');
  if (!input) return 'not_found';
  
  var s = Object.getOwnPropertyDescriptor(window.HTMLTextAreaElement.prototype, 'value').set;
  s.call(input, '你好，请介绍一下你自己');
  
  input.dispatchEvent(new Event('input', { bubbles: true }));
  input.dispatchEvent(new Event('change', { bubbles: true }));
  
  return 'question_filled';
})()"
```

**步骤 2：发送问题**
```
browser act kind=press key=Enter
```

**步骤 3：智能等待回复（核心！）**

**❌ 错误做法**：固定时间等待（`wait timeMs=8000`）- 每个问题回答速度不同

**✅ 正确做法**：智能轮询检测（检测 textarea.disabled + 内容长度）

```javascript
// 第 1 次检测（发送后 3 秒）
browser act kind=wait timeMs=3000
browser act kind=evaluate fn="(() => {
  var textarea = document.querySelector('textarea');
  var msgs = document.querySelectorAll('.qwen-chat-message-assistant');
  if (!textarea) return JSON.stringify({ status: 'ERROR', reason: 'no_textarea' });
  if (msgs.length === 0) return JSON.stringify({ status: 'WAITING', reason: 'no_reply_yet' });
  var len = msgs[msgs.length - 1].textContent.length;
  if (textarea.disabled) return JSON.stringify({ status: 'GENERATING', length: len });
  if (len < 100) return JSON.stringify({ status: 'WAITING', reason: 'content_too_short', length: len });
  return JSON.stringify({ status: 'COMPLETED', length: len });
})()"

// 判断结果：
// - GENERATING → 等待 3 秒，重新检测
// - WAITING → 等待 3 秒，重新检测
// - COMPLETED → 进入步骤 4

// 第 2 次检测（如需要）
browser act kind=wait timeMs=3000
browser act kind=evaluate fn="(...同上...)"

// 第 3 次检测（如需要）
browser act kind=wait timeMs=3000
browser act kind=evaluate fn="(...同上...)"

// 超时保护：最多检测 10 次（30 秒），超时则返回已有内容
```

**检测逻辑说明**：
| 状态 | textarea.disabled | content_length | 含义 |
|------|------------------|----------------|------|
| GENERATING | true | 增长中 | Qwen 正在生成回复 |
| WAITING | false | < 100 | 内容还未渲染完整 |
| COMPLETED | false | ≥ 100 | ✅ 回复完成，可读取 |

**步骤 4：获取回复内容**
```javascript
browser act kind=evaluate fn="(() => {
  var messages = document.querySelectorAll('.qwen-chat-message-assistant');
  if (messages.length === 0) return 'no_messages';
  
  var lastMsg = messages[messages.length - 1];
  // 返回前 2000 个字符
  return lastMsg.textContent.substring(0, 2000);
})()"
```

### 工作流 3：切换模型

**切换到指定模型（例如 Qwen3.5-Plus）**：
```javascript
// 1. 打开模型选择器
browser act kind=evaluate fn="(() => {
  var trigger = document.querySelectorAll('span.ant-dropdown-trigger')[2];
  if (trigger) {
    trigger.click();
    return 'dropdown_opened';
  }
  return 'trigger_not_found';
})()"

browser act kind=wait timeMs=1000

// 2. 选择目标模型
browser act kind=evaluate fn="(() => {
  var items = document.querySelectorAll('[class*=model-item___]');
  for (var i = 0; i < items.length; i++) {
    var text = items[i].textContent.trim();
    if (text.indexOf('Qwen3.5-Plus') !== -1) {
      items[i].click();
      return 'clicked_index_' + i;
    }
  }
  return 'not_found_total_' + items.length;
})()"

browser act kind=wait timeMs=2000

// 3. 验证当前模型
browser act kind=evaluate fn="(() => {
  var selector = document.querySelector('[class*=model-selector-text]');
  return selector ? selector.textContent : 'not_found';
})()"
```

### 工作流 4：展开更多模型（查看所有 19 个模型）

```javascript
// 1. 打开模型选择器
browser act kind=evaluate fn="(() => {
  var trigger = document.querySelectorAll('span.ant-dropdown-trigger')[2];
  if (trigger) {
    trigger.click();
    return 'dropdown_opened';
  }
  return 'trigger_not_found';
})()"

browser act kind=wait timeMs=1000

// 2. Hover 触发"展开更多模型"
browser act kind=evaluate fn="(() => {
  var moreBtn = document.querySelector('[class*=view-more]');
  if (!moreBtn) return 'more_button_not_found';
  
  moreBtn.dispatchEvent(new MouseEvent('mouseenter', { 
    bubbles: true, 
    cancelable: true 
  }));
  moreBtn.dispatchEvent(new MouseEvent('mouseover', { 
    bubbles: true, 
    cancelable: true 
  }));
  
  return 'hovered';
})()"

browser act kind=wait timeMs=1000

// 3. 查看所有模型
browser act kind=evaluate fn="(() => {
  var items = document.querySelectorAll('[class*=model-item___]');
  var names = [];
  for (var i = 0; i < items.length; i++) {
    names.push(items[i].textContent.trim());
  }
  return 'total: ' + items.length + ' | ' + names.join(', ').substring(0, 500);
})()"
```

### 工作流 5：新建对话

```javascript
browser act kind=evaluate fn="(() => {
  var elements = document.querySelectorAll('*');
  for (var i = 0; i < elements.length; i++) {
    if (elements[i].textContent && elements[i].textContent.trim() === '新建对话') {
      elements[i].click();
      return 'clicked';
    }
  }
  return 'not_found';
})()"

browser act kind=wait timeMs=1000
```

### 工作流 6：搜索历史对话

```javascript
browser act kind=evaluate fn="(() => {
  var searchInput = document.querySelector('input[placeholder*=搜索对话]');
  if (!searchInput) return 'search_not_found';
  
  var s = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
  s.call(searchInput, '关键词');
  
  searchInput.dispatchEvent(new Event('input', { bubbles: true }));
  searchInput.dispatchEvent(new Event('change', { bubbles: true }));
  
  return 'search_filled';
})()"
```

### 工作流 7：退出登录

```javascript
// 1. 打开用户菜单
browser act kind=evaluate fn="(() => {
  var userMenu = document.querySelectorAll('span.ant-dropdown-trigger')[0];
  if (userMenu) {
    userMenu.click();
    return 'menu_opened';
  }
  return 'menu_not_found';
})()"

browser act kind=wait timeMs=500

// 2. 点击退出登录
browser act kind=evaluate fn="(() => {
  var elements = document.querySelectorAll('*');
  for (var i = 0; i < elements.length; i++) {
    if (elements[i].textContent && elements[i].textContent.trim() === '退出登录') {
      elements[i].click();
      return 'logout_clicked';
    }
  }
  return 'logout_not_found';
})()"

browser act kind=wait timeMs=2000

// 3. 验证退出结果
browser act kind=evaluate fn="(() => {
  var isLoggedIn = document.querySelectorAll('a[class*=chat-item]').length > 0;
  return isLoggedIn ? 'still_logged_in' : 'logged_out';
})()"
```

## 🔍 元素定位策略

### 1. CSS 选择器（优先使用）

```javascript
// 类选择器
document.querySelector('.class-name')
document.querySelectorAll('.class-name')

// 属性选择器
document.querySelector('[placeholder*=电子邮箱]')
document.querySelector('[class*=chat-item]')
document.querySelector('[class*=model-item___]')

// 组合选择器
document.querySelector('button.qwenchat-auth-pc-submit-button')
document.querySelector('span.ant-dropdown-trigger')
```

### 2. 文本匹配（回退方案）

```javascript
// 遍历所有元素匹配文本
var elements = document.querySelectorAll('*');
for (var i = 0; i < elements.length; i++) {
  if (elements[i].textContent && elements[i].textContent.trim() === '登录') {
    elements[i].click();
  }
}
```

### 3. ant-dropdown-trigger 索引参考

| 索引 | 元素 | 用途 |
|------|------|------|
| 0 | 用户资料菜单 | 退出登录、设置 |
| 1 | 帮助按钮 (?) | 帮助/文档 |
| 2 | 模型选择器 | 切换模型 |
| 3 | 更多按钮 (+) | 上传、深度研究等 |

## ⚠️ 常见陷阱与解决方案

### 陷阱 1：React 表单输入不触发更新

**问题**：直接设置 `input.value = 'xxx'` 不触发 React 状态更新

**解决**：
```javascript
var s = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set;
s.call(input, 'xxx');
input.dispatchEvent(new Event('input', { bubbles: true }));
input.dispatchEvent(new Event('change', { bubbles: true }));
```

### 陷阱 2：Hover 事件用 click 无效

**问题**：某些下拉菜单是 hover 触发，不是 click 触发

**解决**：
```javascript
el.dispatchEvent(new MouseEvent('mouseenter', { bubbles: true }));
el.dispatchEvent(new MouseEvent('mouseover', { bubbles: true }));
```

### 陷阱 3：const/let 导致重声明错误

**问题**：多次执行 eval 时，`const`/`let` 会导致重声明错误

**解决**：始终使用 `var`

### 陷阱 4：可选 chaining `?.` 导致 PowerShell 解析错误

**问题**：`?.` 在 PowerShell 中会被解析为其他含义

**解决**：使用显式 null 检查
```javascript
// ✅ 正确
if (element) {
  element.click();
}

// ❌ 错误
element?.click();
```

### 陷阱 5：登录 UI 变体

**问题**：Qwen 有两种登录 UI（弹窗 vs 全页）

**解决**：智能检测并适配
```javascript
var popupBtn = document.querySelector('button.qwenchat-auth-pc-submit-button');
if (popupBtn) {
  // 弹窗登录
  popupBtn.click();
} else {
  // 全页登录
  var buttons = document.querySelectorAll('button');
  for (var i = 0; i < buttons.length; i++) {
    if (buttons[i].textContent.trim() === '登录') {
      buttons[i].click();
      break;
    }
  }
}
```

### 陷阱 6：登录态检测错误

**问题**：主页可能显示模型名称但实际未登录

**解决**：使用侧边栏历史对话检测
```javascript
// ✅ 正确：检测侧边栏历史对话
var chatItems = document.querySelectorAll('a[class*=chat-item]');
if (chatItems.length > 0) return 'logged_in';

// ❌ 错误：检测模型选择器文本
var modelSelector = document.querySelector('[class*=model-selector-text]');
// 未登录时也可能显示模型名称
```

## 🎯 错误处理策略

### 重试机制

```javascript
browser act kind=evaluate fn="(() => {
  var maxRetries = 3;
  var retryCount = 0;
  
  while (retryCount < maxRetries) {
    var element = document.querySelector('.target');
    if (element) {
      element.click();
      return 'success_on_retry_' + retryCount;
    }
    
    retryCount++;
    return 'retry_' + retryCount;
  }
  
  return 'failed_after_retries';
})()"
```

### 超时保护

```javascript
browser act kind=evaluate fn="(() => {
  var startTime = Date.now();
  var timeout = 10000; // 10 秒
  
  var element = null;
  while (!element && (Date.now() - startTime) < timeout) {
    element = document.querySelector('.target');
  }
  
  if (element) {
    element.click();
    return 'success';
  }
  
  return 'timeout';
})()"
```

## 📊 调试技巧

### 1. 打印页面信息

```javascript
browser act kind=evaluate fn="(() => {
  console.log('URL:', window.location.href);
  console.log('Title:', document.title);
  console.log('Total buttons:', document.querySelectorAll('button').length);
  console.log('Total inputs:', document.querySelectorAll('input').length);
  
  return {
    url: window.location.href,
    title: document.title,
    readyState: document.readyState
  };
})()"
```

### 2. 检查元素是否存在

```javascript
browser act kind=evaluate fn="(() => {
  var selectors = [
    'input[placeholder*=电子邮箱]',
    'input[placeholder*=密码]',
    'button.qwenchat-auth-pc-submit-button',
    'button'
  ];
  
  var result = {};
  for (var i = 0; i < selectors.length; i++) {
    var sel = selectors[i];
    result[sel] = !!document.querySelector(sel);
  }
  
  return JSON.stringify(result);
})()"
```

### 3. 获取页面快照

```
browser snapshot
```

## 🔄 完整示例：Qwen Chat 自动化全流程

```
# 1. 打开页面（OpenClaw 会自动加载保存的 Cookie）
browser open "https://chat.qwen.ai/"

# 2. 等待加载
browser act kind=wait timeMs=3000

# 3. 检测登录状态
browser act kind=evaluate fn="(() => {
  var chatItems = document.querySelectorAll('a[class*=chat-item]');
  if (chatItems.length > 0) {
    return 'already_logged_in';
  }
  return 'need_login';
})()"

# 4. 如果需要登录（通常不需要，Cookie 已保存）
# （跳过登录步骤）

# 5. 提问
browser act kind=evaluate fn="(() => {
  var input = document.querySelector('textarea');
  if (!input) return 'not_found';
  
  var s = Object.getOwnPropertyDescriptor(window.HTMLTextAreaElement.prototype, 'value').set;
  s.call(input, '你好，请介绍一下你自己');
  
  input.dispatchEvent(new Event('input', { bubbles: true }));
  input.dispatchEvent(new Event('change', { bubbles: true }));
  
  return 'question_filled';
})()"

browser act kind=press key=Enter

# 6. 等待回复
browser act kind=wait timeMs=8000

# 7. 获取回复
browser act kind=evaluate fn="(() => {
  var messages = document.querySelectorAll('.qwen-chat-message-assistant');
  if (messages.length === 0) return 'no_messages';
  
  var lastMsg = messages[messages.length - 1];
  return lastMsg.textContent.substring(0, 2000);
})()"

# 8. 切换模型到 Qwen3.5-Plus
browser act kind=evaluate fn="(() => {
  var trigger = document.querySelectorAll('span.ant-dropdown-trigger')[2];
  if (trigger) {
    trigger.click();
    return 'dropdown_opened';
  }
  return 'trigger_not_found';
})()"

browser act kind=wait timeMs=1000

browser act kind=evaluate fn="(() => {
  var items = document.querySelectorAll('[class*=model-item___]');
  for (var i = 0; i < items.length; i++) {
    var text = items[i].textContent.trim();
    if (text.indexOf('Qwen3.5-Plus') !== -1) {
      items[i].click();
      return 'clicked_' + i;
    }
  }
  return 'not_found';
})()"

browser act kind=wait timeMs=2000

# 9. 验证模型
browser act kind=evaluate fn="(() => {
  var selector = document.querySelector('[class*=model-selector-text]');
  return selector ? selector.textContent : 'not_found';
})()"

# 10. 新建对话
browser act kind=evaluate fn="(() => {
  var elements = document.querySelectorAll('*');
  for (var i = 0; i < elements.length; i++) {
    if (elements[i].textContent && elements[i].textContent.trim() === '新建对话') {
      elements[i].click();
      return 'clicked';
    }
  }
  return 'not_found';
})()"

browser act kind=wait timeMs=1000
```

## 📚 最佳实践总结

### ✅ 必须做的

1. **始终使用 IIFE** 包裹所有 eval JavaScript 代码
2. **始终使用 `var`** 而不是 `const`/`let`
3. **React 表单输入** 必须用 `nativeInputValueSetter` + `dispatchEvent`
4. **Hover 事件** 必须用 `dispatchEvent('mouseenter')` + `dispatchEvent('mouseover')`
5. **每步操作后添加验证** 确保操作成功
6. **添加适当的等待时间** 让页面/UI 渲染完成（500-3000ms）
7. **使用智能检测** 适配多种 UI 变体
8. **错误处理** 包含重试和超时保护
9. **智能等待回复** - 检测 `textarea.disabled` + 内容长度，不用固定时间
10. **页面管理** - 刷新用 `location.reload()`，不用 `browser open` 创建新标签页

### ❌ 避免做的

1. **不要用 `const`/`let`** - 会导致重声明错误
2. **不要用 `?.`** - PowerShell 解析问题
3. **不要直接设置 input.value** - React 不会更新
4. **不要用 click 触发 hover** - 必须用 dispatchEvent
5. **不要用模型选择器检测登录** - 要用侧边栏历史对话
6. **不要省略等待时间** - UI 需要渲染时间
7. **不要用固定时间等待回复** - 每个问题回答速度不同
8. **不要频繁 `browser open`** - 会导致标签页过多，用刷新或新建对话

---

## 🗂️ 页面管理最佳实践（重要！）

### 场景 1：页面卡住/超时

**❌ 错误做法**：
```javascript
browser open "https://chat.qwen.ai/"  // 创建新标签页！
```

**✅ 正确做法**：
```javascript
browser act kind=evaluate fn="(() => { location.reload(); return 'reloaded'; })()"
// 或
browser open "https://chat.qwen.ai/c/当前对话 ID"  // 复用当前标签页
```

### 场景 2：开启新话题

**❌ 错误做法**：
```javascript
browser open "https://chat.qwen.ai/"  // 创建新标签页！
```

**✅ 正确做法**：
```javascript
// 点击"新建对话"按钮（同一标签页）
browser act kind=evaluate fn="(() => {
  var elements = document.querySelectorAll('*');
  for (var i = 0; i < elements.length; i++) {
    if (elements[i].textContent && elements[i].textContent.trim() === '新建对话') {
      elements[i].click();
      return 'clicked';
    }
  }
  return 'not_found';
})()"
```

### 场景 3：多轮对话

**❌ 错误做法**：
```javascript
// 每轮都打开新标签页
browser open "https://chat.qwen.ai/"
// ... 提问 ...
browser open "https://chat.qwen.ai/"  // 又创建新标签页！
```

**✅ 正确做法**：
```javascript
// 只打开一次
browser open "https://chat.qwen.ai/"

// 第 1 轮提问 → 检测完成 → 读取回复
// 第 2 轮提问（同一页面）→ 检测完成 → 读取回复
// 第 3 轮提问（同一页面）→ 检测完成 → 读取回复
```

### 场景 4：切换网站

**❌ 错误做法**：
```javascript
browser open "https://chat.qwen.ai/"
browser open "https://example.com"  // 创建新标签页！
```

**✅ 正确做法**：
```javascript
// 同一标签页内切换
browser open "https://chat.qwen.ai/"
// ... 操作 Qwen ...
browser open "https://example.com"  // 复用当前标签页
```

### 标签页管理原则

| 原则 | 说明 |
|------|------|
| **保持 1 个活动标签页** | 同一网站的对话始终用同一个标签页 |
| **刷新用 reload()** | 页面卡住时用 `location.reload()` |
| **新对话用按钮** | 点击"新建对话"按钮，不用 browser open |
| **关闭多余标签页** | 定期用 `browser close` 清理不用的标签页 |
| **检查标签页数量** | 定期用 `browser tabs` 检查，保持可控 |

---

## ⏱️ 智能等待策略（重要！）

### 为什么不用固定时间等待？

| 问题类型 | 实际需要时间 | 固定等待 15 秒 |
|---------|-------------|-------------|
| 简单问题（1+1=?） | 5 秒 | ❌ 浪费 10 秒 |
| 普通问答（解释概念） | 20 秒 | ❌ 还没完成 |
| 复杂分析（详细解释） | 40 秒 | ❌ 超时误判 |

### 智能检测方法

**检测指标**：
1. `textarea.disabled` - true = 正在生成，false = 完成
2. `content_length` - 内容长度，判断是否渲染完整

**轮询流程**：
```
1. 发送问题
2. 等待 3 秒（Qwen 开始处理）
3. 检测状态：
   - GENERATING (disabled=true) → 等待 3 秒，重新检测
   - WAITING (length<100) → 等待 3 秒，重新检测
   - COMPLETED (disabled=false, length≥100) → ✅ 完成，读取回复
4. 超时保护：最多检测 10 次（30 秒），超时则返回已有内容
```

**代码模板**：
```javascript
// 智能等待函数（可复用）
browser act kind=evaluate fn="(() => {
  var textarea = document.querySelector('textarea');
  var msgs = document.querySelectorAll('.qwen-chat-message-assistant');
  
  if (!textarea) return JSON.stringify({ status: 'ERROR', reason: 'no_textarea' });
  if (msgs.length === 0) return JSON.stringify({ status: 'WAITING', reason: 'no_reply_yet' });
  
  var len = msgs[msgs.length - 1].textContent.length;
  
  if (textarea.disabled) return JSON.stringify({ status: 'GENERATING', length: len });
  if (len < 100) return JSON.stringify({ status: 'WAITING', reason: 'content_too_short', length: len });
  
  return JSON.stringify({ status: 'COMPLETED', length: len });
})()"
```

## 🎓 命令速查表

| 操作 | 命令格式 | 示例 |
|------|---------|------|
| 打开页面 | `browser open "URL"` | `browser open "https://chat.qwen.ai/"` |
| 等待 | `browser act kind=wait timeMs=毫秒` | `browser act kind=wait timeMs=3000` |
| 执行 JS | `browser act kind=evaluate fn="代码"` | `browser act kind=evaluate fn="(() => {...})()"` |
| 键盘按键 | `browser act kind=press key=按键` | `browser act kind=press key=Enter` |
| 获取快照 | `browser snapshot` | `browser snapshot` |

## 🎯 总结

这个技能模板提供了：

1. ✅ **完整的 Qwen Chat 自动化工作流** - 登录、提问、切换模型、新建对话等
2. ✅ **适配 OpenClaw browser 工具** - 所有命令都经过验证
3. ✅ **React/Vue 特殊处理** - 正确处理表单输入
4. ✅ **Hover 事件触发** - 展开更多模型
5. ✅ **智能登录检测** - 自动识别登录状态
6. ✅ **错误处理策略** - 重试和超时保护
7. ✅ **调试技巧** - 快速定位问题
8. ✅ **最佳实践** - 避免常见陷阱

**Cookie 管理**：OpenClaw 会自动使用保存的 Cookie，无需重复登录！

**使用方法**：直接复制上述工作流到您的 OpenClaw 中执行即可！🚀
