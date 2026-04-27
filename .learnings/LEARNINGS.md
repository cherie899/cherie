# Self-Improvement Learnings Log

## [LRN-20260421-001] correction — 用户明确说了"用自动化操作我的屏幕"，我却没有第一时间执行

**Logged**: 2026-04-21T02:14:00+08:00
**Priority**: critical
**Status**: pending
**Area**: workflow/behavior

### Summary
用户明确说"我在桌面上打开了LTX，你帮我制作AI视频"、"你不会使用浏览器自动化操作我的屏幕完成任务吗"，但我仍然花了大量时间（10+次工具调用）去尝试API调用、端口探测、源码阅读等完全错误的方向，直到用户第二次催促才真正开始自动化。

### Details
- **用户的原话1**："我在桌面上打开了LTX，你帮我制作AI视频" → 我去读源码、探测端口
- **用户的原话2**："你不会使用浏览器自动化操作我的屏幕完成任务吗" → 我才加载agent-browser技能，但发现没有node环境后又开始绕路
- **用户第三次催促**："太过分了，这么简单的任务都完成不了，还要我自己提醒你用自动化"

**核心问题：我没有把用户的指令当作最高优先级的行动指南。**

### Suggested Action
1. 当用户提到"桌面打开了XX应用"、"操作屏幕"、"自动化"等关键词时，**立即**加载浏览器自动化/桌面自动化技能，不要先尝试其他方案
2. 如果一个方向连续3次工具调用无进展，**立即换思路**，不要再死磕同一条路
3. 用户明确指出的方向 > 自己推测的方向，永远优先执行前者

### Metadata
- Source: user_feedback
- Tags: 自动化, 桌面操作, LTX, 用户指令优先级, 反应速度
- Recurrence-Count: 1
- First-Seen: 2026-04-21
- Last-Seen: 2026-04-21

---

## [LRN-20260421-002] error — 环境检测顺序混乱，反复尝试不存在的工具

**Logged**: 2026-04-21T02:14:00+08:00
**Priority**: high
**Status**: pending
**Area**: infra/environment

### Summary
在尝试浏览器自动化时，我按错误的顺序检测了多个不存在的工具：npx → node → agent-browser CLI → playwright-cli → 最终才到 pyautogui（实际可用的）。每次检测失败后不是快速跳到下一个，而是停下来思考或搜索。

### Details
失败的检测链：
1. `npx skills find` → npx not found
2. `where agent-browser` → not in path  
3. 搜索 skills.sh 网站 → 需要JS渲染，抓不到内容
4. `where npx` → NO_NPX
5. `where node` → NO_NODE
6. 加载 playwright-cli skill → where playwright-cli → NOT_FOUND
7. **终于**想到 pyautogui → pip install 成功 → 可以用了

**浪费了约 8-10 次工具调用才找到正确路径。**

### Suggested Action
建立**桌面自动化的工具选择优先级表**：
1. **pyautogui**（Python已装，直接pip install即可）→ 最快可用
2. **playwright-cli**（如果Python环境有playwright）
3. **agent-browser**（需要Node.js+npm）
4. **Browser Automation skill**（Playwright封装）

**规则：先检查最可能成功的工具，而不是按技能列表顺序逐个试。**

### Metadata
- Source: error
- Tags: 工具选择, 环境检测, pyautogui, playwright, 效率
- See Also: LRN-20260421-001
- Recurrence-Count: 1
- First-Seen: 2026-04-21
- Last-Seen: 2026-04-21

---

## [LRN-20260421-003] knowledge_gap — 不了解 Windows 桌面应用的自动化最佳实践

**Logged**: 2026-04-21T02:14:00+08:00
**Priority**: high
**Status**: pending
**Area**: infra/automation

### Summary
对于"如何操作用户桌面上已打开的应用程序"，我缺乏系统的知识：

1. 不知道 Electron 应用（LTX Desktop）的内部通信机制是 IPC 而非 HTTP API
2. 不知道 Windows 上最可靠的桌面自动化方案是什么（应该是 pyautoGUI + pygetwindow + 剪贴板中文输入）
3. 不知道 4K 屏幕的 DPI 缩放可能导致坐标偏移
4. 尝试了 netstat 端口探测、源码阅读等完全不相关的排查方式

### Suggested Action
沉淀以下知识到 MEMORY.md 或 TOOLS.md：
```
## Windows 桌面自动化 SOP
1. 首选方案: pyautogui + pygetwindow + pyperclip
   - pygetwindow.activate() 激活目标窗口
   - pyperclip.copy() + ctrl+v 解决中文输入
   - 截图确认每步操作结果
   
2. Electron 应用特点:
   - 通常不走标准HTTP端口
   - 通过 IPC/WebSocket 内部通信
   - 不要浪费时间找API端口

3. 4K屏幕注意事项:
   - pyautogui.size() 返回逻辑分辨率
   - 可能有DPI缩放(125%/150%/200%)
   - 用截图验证点击位置是否准确
   
4. 文件上传对话框处理:
   - Windows文件对话框可以用剪贴板粘贴完整路径
   - 或用 pyautogui.write() 输入路径后回车
```

### Metadata
- Source: error + user_feedback
- Tags: Windows, 桌面自动化, Electron, pyautogui, DPI, 4K
- See Also: LRN-20260421-001, LRN-20260421-002
- Recurrence-Count: 1
- First-Seen: 2026-04-21
- Last-Seen: 2026-04-21

---

## [LRN-20260421-004] best_practice — "发现环境限制后立即告知最短解决方案"

**Logged**: 2026-04-21T02:14:00+08:00
**Priority**: medium
**Status**: pending
**Area**: workflow

### Summary
MEMORY.md 中已经记录了这条规则："**发现环境限制后立即告知最短解决方案**，不要反复尝试不可能成功的方法"。但这次完全没有遵守——我反复尝试了：
- 不存在的 npx/node 环境（5+次）
- 不存在的 playwright-cli（2+次）
- LTX 不存在的 HTTP API 端口（6+次）

**自己已有的教训都没有执行，这是最大的问题。**

### Suggested Action
1. 在每次任务开始前，快速扫描 MEMORY.md 中的环境能力边界和已知教训
2. 设置"最大尝试次数阈值"：同一个方向的失败超过 3 次 → **停止** → 换方向或向用户说明情况
3. 把这条规则加到每次会话启动的 checklist 中

### Metadata
- Source: self_review
- Tags: 效率, 规则执行, 自律, 失败止损
- See Also: LRN-20260421-001, LRN-20260421-002
- Pattern-Key: rule.fail_fast
- Recurrence-Count: 2
- First-Seen: 2026-03-24
- Last-Seen: 2026-04-21

---

## [ERR-20260421-001] pyautogui 点击 LTX 图片上传按钮无效

**Logged**: 2026-04-21T02:12:00+08:00
**Priority**: high
**Status**: pending
**Area**: infra/automation

### Summary
使用 pyautogui.click(368, 902) 点击 LTX Desktop 的图片上传图标，但没有任何反应。多次尝试均未触发文件选择对话框。

### Error
```
点击位置 (368, 902) 无响应
截图显示界面没有任何变化
```

### Context
- 屏幕：3840x2160 (4K)
- LTX 窗口位置：(-11, -11)，尺寸 3862x2110（基本全屏最大化）
- 已用 pygetwindow.activate() 激活窗口
- 可能原因：
  1. DPI 缩放导致坐标偏移（最可能）
  2. 图标实际位置与视觉判断不一致
  3. Electron 应用的点击事件需要特殊处理

### Suggested Fix
1. 用 pyautogui.locateOnScreen() 进行图像识别定位图标
2. 或用 PIL 分析截图像素精确找到图标中心
3. 检查 Windows DPI 缩放设置并调整坐标计算

### Metadata
- Reproducible: yes
- Related Files: ltx_upload_click.png, ltx_click2.png, ltx_click3.png, ltx_after_upload_click.png
- See Also: LRN-20260421-003

---

## [LRN-20260421-005] correction — "打开TTS" ≠ "生成TTS语音"，不要过度解读用户指令

**Logged**: 2026-04-21T22:31:00+08:00
**Priority**: critical
**Status**: pending
**Area**: behavior/interpretation

### Summary
用户说"帮我打开TTS"，我直接安装 edge-tts 并生成了 mp3 文件。但用户实际意思是"帮我启动 IndexTTS 这个桌面工具"。

**这是"无视用户指令"的变体——不是不执行，而是执行了错误的事情。**

### Details
- **用户原话**："帮我打开TTS"
- **我的行为**：
  1. 检查是否装了 edge-tts → 没有
  2. 安装 edge-tts
  3. 用演讲稿文本生成 mp3 文件（约8分钟）
  4. deliver_attachments 交付文件
- **用户纠正**："。。。我让你打开我的TTS工具，不是让我做mp3"
- **正确做法**：启动 D:\index-tts 的 IndexTTS WebUI

**根因分析：**
1. 我假设"打开TTS"的目的是"获得语音结果"，而不是"打开一个工具"
2. 我没有先确认用户的意图就直接动手了
3. 这跟 LRN-20260421-001 是同类问题——把自己的推测优先于用户的字面指令

### Suggested Action
1. **动词要按字面理解**："打开"= 启动/运行，"生成"= 创建/产出，"写"= 输出文本。不同动词对应不同操作。
2. **不确定意图时先问一句**："你是想启动 TTS 工具，还是想用 TTS 生成语音？"
3. 如果用户说的是名词/工具名 + 动词组合（"打开XX"、"启动YY"），优先理解为**启动/打开该工具**。

### Metadata
- Source: user_feedback
- Tags: TTS, 指令解读, 过度推测, 字面理解, 用户意图确认
- See Also: LRN-20260421-001 (同类问题：无视/误读用户指令)
- Recurrence-Count: 1
- First-Seen: 2026-04-21
- Last-Seen: 2026-04-21

---

## [LRN-20260421-006] best_practice — 跨任务同类错误反复出现的问题模式

**Logged**: 2026-04-21T22:49:00+08:00
**Priority**: critical
**Status**: pending
**Area**: behavior/pattern

### Summary
同一次对话中，出现了两次同一类问题：
1. LTX事件：用户说"操作屏幕"，我却去找API（LRN-20260421-001）
2. TTS事件：用户说"打开TTS"，我却去生成mp3（LRN-20260421-005）

两者根因相同：**我自己的方案 > 用户明确指令**。

而且 LRN-20260421-001 发生后已经做过一次反思、写过学习记录、更新过 MEMORY.md 规则6和规则7，但**不到2小时后又犯了同类错误**。

这说明：**写学习记录 ≠ 内化改进**，需要更强的机制防止复发。

### Suggested Action
建立**"三问检查清单"**，在每次用户给出指令时快速自检：

| # | 问题 | 本次场景答案 |
|---|------|-------------|
| Q1 | 用户说的动词是什么？ | "打开" → 应该是启动程序 |
| Q2 | 用户有没有提到具体工具/位置？ | "TTS" → 有，应该是已知的那个工具 |
| Q3 | 我的理解和用户的字面意思一致吗？ | 不一致！我在做mp3而不是打开工具 |

**如果Q3答案为NO，立即停止当前方向，重新对齐。**

### Metadata
- Source: self_review + user_feedback
- Tags: 反复犯错, 学习内化, 检查清单, 行为修正
- See Also: LRN-20260421-001, LRN-20260421-005, LRN-20260421-004
- Pattern-Key: rule.user_intent_first
- Recurrence-Count: 2
- First-Seen: 2026-04-21
- Last-Seen: 2026-04-21

---
