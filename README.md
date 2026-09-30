# magpie-lzcapp

[Magpie](https://github.com/yetone/magpie) 的懒猫微服打包：把各家大模型接到一处，给 Codex / Claude Code / OpenCode 等 agent 提供统一网关（兼容 OpenAI / Anthropic / Gemini），并带一个网页控制台。

- 上游镜像：`ghcr.io/yetone/magpie`（官方商店交付走 `delivery.mode: lazycat`，由 Action 转存到 `registry.lazycat.cloud` 后改写 manifest）
- 一个容器两种角色：镜像默认 `serve` 只跑网关；这里用 `magpie web --addr 0.0.0.0:3430 --no-open`，控制台与网关一起跑
- 路由：`/` → 控制台（3430），`/v1`、`/v1beta` → 网关（3425），网关的三种 API 路径都落在这两个前缀下
- 控制台有自带 Key（安装向导里的 Web Key，默认 `sk-magpie-lazycat-web`），`?k=` 打开一次后写 Cookie；启动器入口已带 Key
- 数据（账号、API Key、登录态）持久化在 `/lzcapp/var/config`
- 镜像默认把网关绑在 `0.0.0.0:3425` 且对外接受任意 Key，这里用 `MAGPIE_ADDR=127.0.0.1:3425` 压回回环：控制台里打开 Settings → Share on local network 之前 `/v1` 不对外服务，打开后监听自动切到 `0.0.0.0` 并要求共享 Key

`public_path` 只对 `/v1`、`/v1beta` 关闭微服账号密码鉴权（agent/CLI 没法过微服登录），网关自身的共享 Key 是这两条路径的鉴权；控制台仍在微服鉴权之后。

## 安装后要做的两步

1. 打开控制台（启动器入口，或 `https://<域名>/?k=<Web Key>`）
2. Settings → Share on local network 打开共享，记下生成的 `sk-magpie-…` Key，然后让 agent 连 `https://<域名>/v1`（OpenAI 兼容）或 `https://<域名>`（Anthropic / Gemini），API Key 填共享 Key
