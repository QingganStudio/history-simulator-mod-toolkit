---
title: 修改游戏样式
description: 使用安全的 data-mod-class 选择器编写和验证 custom.css
---

# 修改游戏样式

> 适用范围：当前 `custom.css` 安全编译器；最近代码核对：2026-10-14；验证层级：样式预览 + 打包版自动注入。

## 打开样式编辑器

从 Mod Maker 工具区打开样式编辑器。缺少 `custom.css` 时会自动创建；尚未首次保存的作品会在保存 Mod 时写入根目录。编辑器会保留顶部规则说明。

启动时锁定的每个已启用 Mod 只要存在合法且不超过 1MB（准确说是 1 MiB，即 1,048,576 个 UTF-8 字节）的 `custom.css`，游戏就会自动编译并注入；不存在、超限或编译后没有有效规则时不注入。

![样式编辑器显示稳定选择器说明和预览入口](/screenshots/stage-4/08-style-editor-warning.webp)

*图：先使用界面提供的稳定选择器提示和预览入口；编译结果仍须在预览中检查。*

## 查找稳定选择器

1. 打开 Preview。打包后的 Preview 始终启用 React Grab，可以直接从渲染结果中选择元素。
2. 选中目标元素并复制；复制内容会保留完整 `outerHTML`，包括真实的 `data-mod-class` 和运行时属性。
3. 读取 DOM 中真实存在的 `data-mod-class` 完整稳定名，在样式编辑器编写：

```css
[data-mod-class~="实际稳定名"] {
  color: #f5d58a;
}
```

不要从源码文件名或视觉名称猜稳定名。选择器被空格、`>`、`+`、`~` 分成多段时，每一段都必须包含自己的 `[data-mod-class~="..."]` 锚点。

## 使用 React Grab 让 AI 协助修改

在 Preview 中使用 React Grab 复制目标元素后，可以把剪贴板内容粘贴给 AI。复制内容包含：

- 一段提示 AI 先读取当前工程 `custom.css` 顶部 `@mod-maker custom-css-guide` 说明的提示词；
- 选中元素的完整 DOM，包括 `data-mod-class` 和其他运行时属性；
- React Grab 中填写的用户修改需求。

AI 需要读取当前工程的 `custom.css`，保留已有规则和顶部说明，再根据复制的 DOM 生成最小修改。复制内容只反映渲染时的 DOM，不提供可靠的 JSX、TypeScript 或源码位置；需要查看当前计算样式、祖先节点或页面状态时，再使用开发者工具辅助确认。

如果没有填写修改需求，AI 只能识别目标元素，无法判断希望调整的效果。建议同时说明目标、期望效果和必要的范围，例如“将这个按钮的背景改为深红色，只影响当前页面”。

## 支持的写法

可以使用 `:hover`、`:focus`、伪元素、`@media` 和 `@keyframes`。普通规则编译后会自动增加 `!important`。优先修改颜色、字号、边框等局部表现，避免移动、隐藏或覆盖关键按钮和交互。

## 单位与响应式转换

CSS 声明值中的小写 `px` 会在预览和游戏启动时，按 1920×1080 基准自动转换为响应式尺寸。例如：

```css
[data-mod-class~="home__map"] {
  width: 200px;
  margin: 12px 0;
}
```

其中 `200px` 会转换为等价的 `min(calc(200 / 1920 * 100vw), calc(200 / 1080 * 100vh))`。磁盘文件仍保留作者写入的小写 `px`，`0px` 会规范为 `0`。这条规则适用于 `width`、`height`、间距、字号、`transform` 等声明值。

需要固定像素时使用大写 `PX`，例如 `1PX`；大写 `PX` 不会被转换。`%`、`vw`、`vh`、`rem`、`em`、`fr`、`calc()`、`var()` 等值会保持原样。媒体查询条件、选择器、注释和 `url()` 中的 `px` 也不会转换。`custom.css` 中不要写 `rp()` 或 UnoCSS 的 `w-rp-*` 类名。

`opacity`、`z-index`、`font-weight` 和无单位的 `line-height` 使用纯数字，不要为了响应式转换给这些属性添加 `px`。

## 安全限制

- 禁止裸标签、全局 class、ID、`*`、`html`、`body` 等未锚定选择器；
- 未知稳定名会使整条选择器失效；
- 设置页面的稳定名受保护，命中后规则被拒绝；
- 所有 `@import` 会被移除；
- `url(...)` 只允许 `data:` 内联资源，http、https、相对路径和协议相对 URL 会被中和；
- 单文件不得超过 1MB（1 MiB，即 1,048,576 个 UTF-8 字节）。

## 1MB 限制在哪一层生效

| 环节 | 当前行为 |
|---|---|
| 编辑器保存（Electron 与独立浏览器） | 样式编辑器的“保存并预览”和整包保存的样式阶段统一经过样式写入入口：按 UTF-8 字节计算，超限时写入失败并提示；无 Electron 桥接时经 File System Access API 直接写盘，同样拦截 |
| 预览编译 | 有独立的源码长度保护并返回超限告警；预览成功不等于已经通过磁盘文件的 UTF-8 字节检查 |
| 游戏启动 | 对启动锁定安装目录中的实际文件再次检查字节数；超限时整份样式都不注入 |
| Workshop 整目录预检 | 对根目录实际文件复核 1MB 上限，超限报 `custom-css` 错误并阻断上传；写法层问题（`@import`、非 `data:` 的 `url()`、未锚定顶层选择器）不检查，由运行时安全编译处理 |

因此 Workshop 目录预检通过只代表样式在 1MB 以内；预检不检查写法层问题——`@import`、外部 URL、未锚定选择器等由运行时安全编译分别剥离、中和、整条丢弃，稳定名拼写、受保护页面等编译层行为都要靠预览验证。

## 保存并预览

点击“保存并预览”，检查预览窗口效果和编译告警。规则无效时按顺序检查：CSS 语法、稳定名拼写、每段是否有锚点、是否命中受保护页面、是否使用外部 URL。

如果 AI 或外部工具直接修改了磁盘上的 `custom.css`，文件写入本身不会自动通知已经打开的 Preview。写入完成后必须通过 CDP 连接 Mod Preview（不是游戏主窗口）：先访问 `http://127.0.0.1:9333/json/list`，从 `webSocketDebuggerUrl` 连接 Preview；在 Preview 已打开并 ready 的前提下，通过 `Runtime.evaluate` 执行桥接发送：

```js
const message = {
  type: 'preview:custom-css-source',
  payload: { source: `完整的 custom.css 内容` },
};
await cdp.send('Runtime.evaluate', {
  expression: `window.modmakerNative.sendPreviewMessage(${JSON.stringify(message)})`,
  awaitPromise: true,
  returnByValue: true,
});
```

如果 CDP 工具已经处于 Preview 页面上下文，也可以直接执行 `window.modmakerNative.sendPreviewMessage(message)`。写入文件后必须发送完整源码，不能只发送文件路径。

也可以关闭未保存的样式草稿后，在样式编辑器点击“预览”，让编辑器重新读取磁盘文件。仅刷新 Preview，或只发送 `preview:asset-changed`，不能替代 `preview:custom-css-source` 的样式注入。发送后等待渲染完成，通过 CDP 检查 `#mod-preview-custom-css` 的内容、目标元素的 computed style 和控制台告警；若样式编辑器存在未保存草稿，点击预览会优先使用内存草稿，不会覆盖它。

整包保存先写结构化快照，再单独写 `custom.css`。无论走游戏内 Electron 入口还是独立浏览器的 File System Access API，样式写入前都会统一按 UTF-8 字节检查 1MB 上限，超限时写入失败并提示；若样式在这一步因超限或权限失败，前面的结构化文件不会自动回滚——把目录视为部分更新，缩小样式并重新完整保存，不要直接上传。

游戏只读取启动时锁定的安装目录。若当前测试的是本地导入 Mod，修改原工程样式后要先保存，再删除已有托管副本、重新导入、启用并重启；只重启不会把原工程变化同步进托管副本。最后在打包版逐页检查不同分辨率、悬停/焦点、弹窗和关键按钮；不要只在一个静态截图上判断安全。

下一步：[游戏规则与界面配置](/advanced/game-and-ui-config)。
