# Self-Improvement Errors Log

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
