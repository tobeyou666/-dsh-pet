# 本地说明：「多个桌宠」现象与单只配置

> 本文档记录**本机（本镜像使用者的机器）**的配置与排查结论，**不是上游缺陷**，也不包含任何代码改动。
> 对应的上游行为是设计使然。

## 现象

应用 dsh-pet 后出现**多个桌宠，且各自功能不尽相同**。

## 根因：不是 bug，是 `display: "both"` 的效果

内置默认配置（`dsh-pet/assets/config.jsonc`）只有 **1 只**宠物（`id: main`），
但它的 `display` 是 `"both"` —— 含义是**在下面两个不同表面各渲染一遍**：

| 表面 | 渲染者 | 过滤条件 |
|---|---|---|
| 浏览器 overlay | `src/client/app.ts` 注册 `shell.overlay` → `PetMulti` → 每只一个 `PetCard` | `src/client/pet.ts` 按 `isWebVisible(p.display)` |
| 桌面 Electron 窗 | `runtime/electron-helper/main.js` 的 `createPetWindows()`（每宠一个局部小窗口） | `src/host/index.ts` 按 `isDesktopVisible(p.display)` |

所以"只配了 1 只却看到 2 个"是这个默认值的直接结果。

### 为什么"功能不尽相同"

两个表面的**基础能力来自同一份共享代码**（`src/shared/menu.ts` 菜单树、`displays`、
`physics`、`chat`、`whisper`、`balance`），差异只在右键菜单根项：

| 菜单项 | 浏览器 overlay | 桌面 Electron 窗 |
|---|---|---|
| 打开网站 | ✗ | ✓ |
| 查看余额 | ✗ | ✓ |
| 碎碎念 / 对话 / 回到初始位置 | ✓ | ✓ |
| 动作 → 分类 → 具体动画 | ✓ | ✓ |

这是作者**有意为之**，`src/client/pet.ts` 中有明确注释：浏览器端不提供「打开网站」
（用户已经在网页里）与「查看余额」（已由 `/balance` 命令实现）。

### 额外的放大因素

若同时运行多个 DSH 实例（例如 `dsh web` 服务 + `dsh-desktop-windowos` 桌面端），
**每个实例都会各自渲染一份浏览器 overlay**，于是宠物数量会进一步增加：

```
1 个 dsh web 实例        → overlay 1 只 + 桌面窗 1 只 = 2 只
+ 桌面端（另一个实例）    → overlay 再 1 只           = 3 只
```

这是"每个 DSH 实例独立渲染插件"的必然结果，不属于 dsh-pet 缺陷。

## 本机的处理方式

写入用户覆盖层 `$DSH_HOME/dsh-pet/main-config.json`（插件设计的可编辑层，只需写要改的字段，
其余自动从 `assets/config.jsonc` 补齐）：

```json
{
  "pets": [
    { "id": "main", "name": "蓝毛小女仆", "display": "desktop" }
  ]
}
```

`display` 可选值为 `web`（仅浏览器）/ `desktop`（仅桌面）/ `both`（两者）/ `none`（都不显示）。

### 修改方式（二选一）

1. **图形界面**：DSH 设置 → 桌宠配置 → 显示位置 → 选择目标值。
   设置页与宠物是否可见**无关**（`settings.section` 是无条件注册的），
   该下拉会写入 `/dsh-pet-7340/config`，host 侧随即重启桌面 helper 适配。
2. **直接编辑文件**：改 `main-config.json` 的 `display`，**刷新页面即生效**
   （`readAllConfig` 每次请求重读文件，无需重启 DSH）。

### 一个客观边界

"只看到一只"与"宠物出现在网页里"**互斥** —— 这是两个独立渲染表面的物理事实，
任何配置都无法让同一只宠物既只显示一次、又同时具备两个表面的全部入口。
因此本机的取舍是：默认只留桌面窗（菜单更全），需要网页侧能力时再把 `display` 切回去。

## 验证方式

用插件自身的 `readAllConfig` 校验合并结果，而不是手写一份 JSON 去猜：

- `display` 正确变为 `desktop`
- **动画池、物理参数、文案等条目级字段与「无用户配置」基线逐字节一致**（覆盖层没写的字段不会被清空）
