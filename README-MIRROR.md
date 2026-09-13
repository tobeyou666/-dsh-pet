# 镜像说明 / Mirror notice

本仓库是 **[PC2005-cloud/dsh-pet](https://github.com/PC2005-cloud/dsh-pet)** 的镜像（mirror），
不是原创项目。全部代码、素材与文档的著作权归原作者 **PC2005-cloud** 所有，以 **MIT License** 授权
（见 `LICENSE`）。

- 上游：https://github.com/PC2005-cloud/dsh-pet
- npm：https://www.npmjs.com/package/dsh-pet
- 已安装版本：`dsh-pet@0.2.8`

## 本镜像相对上游的差异

只有一个提交：

| 提交 | 内容 |
|---|---|
| `fix: drop defunct @deepseek-ai/dsh-client-runtime from client deps` | `dsh-pet/package.json`：从 `dsh.client.inject` 与 `peerDependencies` 移除 `@deepseek-ai/dsh-client-runtime`（2 行删除） |

其余内容与上游 `main` 一致。

### 为什么移除这个包

`@deepseek-ai/dsh-client-runtime` 在 npm 上**止于 `0.1.1-rc.2`**（最后一次发布变更时间
2026-08-21），DSH 0.1.5 已用 `@deepseek-ai/dsh-client-modules` 取代它 —— 这一点可由当前
DSH 核心包的声明交叉验证：`dsh-cordis-client-runner` 注入的是 `dsh-client-modules`，
且**核心包中没有任何一个注入 `dsh-client-runtime`**。

上游作者在
[issue #50](https://github.com/PC2005-cloud/dsh-pet/issues/50)
中已确认 0.1.5 移除了旧 API 并做了适配迁移
（提交 `fix(notify): 系统通知改走 host 转发通道适配 DSH 0.1.5 事件 API`）；
本改动是同一轮迁移的**遗漏项**。

### 这不是运行时故障

客户端模块加载器对 `inject` 中无法对应到启动图行的条目**静默跳过**
（`@deepseek-ai/dsh-client-modules/lib/client.js`），而 dsh-pet 的客户端 bundle 实测只
`require("react")` 与 `react/jsx-runtime`。因此本改动**不改变任何运行时行为**，
属于清理失效声明。

## 本地配置不在本仓库

本机为避免桌宠重复显示所做的 `display` 配置是**个人偏好**，保存在
`$DSH_HOME/dsh-pet/main-config.json`，**不属于上游源码，也未包含在本仓库中**。
其成因与说明见 `dsh-pet/LOCAL-NOTES.zh-CN.md`。
