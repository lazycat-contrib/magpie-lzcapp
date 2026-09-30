# magpie-lzcapp

[Magpie](https://github.com/yetone/magpie) 的懒猫微服打包：把各家大模型接到一处，给 Codex / Claude Code / OpenCode 等 agent 提供统一网关（兼容 OpenAI / Anthropic / Gemini），并带一个网页控制台。

- 上游镜像：`ghcr.io/yetone/magpie`（官方商店交付走 `delivery.mode: lazycat`，由 Action 转存到 `registry.lazycat.cloud` 后改写 manifest）
- 一个容器两种角色：镜像默认 `serve` 只跑网关；这里用 `magpie web --addr 0.0.0.0:3430 --no-open`，控制台与网关一起跑
- 路由：`/` → 控制台（3430），`/v1`、`/v1beta` → 网关（3425）；网关的 OpenAI / Anthropic / Gemini 路径都落在这两个前缀下
- 网关监听地址不覆盖（沿用镜像默认 `MAGPIE_ADDR=0.0.0.0:3425`），微服入口才能把 `/v1`、`/v1beta` 转发进来
- `user: root`：镜像以 nonroot 运行，而 `/lzcapp/var/config` 由平台以 root 创建，用 root 启动保证配置可写
- 控制台访问控制：**不做安装向导参数**，`MAGPIE_WEB_KEY` 固定为 `sk-magpie-lazycat-web`，启动器入口直接带这个 Key（`/?k=sk-magpie-lazycat-web`）。原因：entry 不渲染模板参数；写死 Key 又无法与用户自设值保持一致；而 `ctx` 注入只在懒猫客户端链路上生效，覆盖不了浏览器直连等入口
- `public_path` 只放 `/v1`、`/v1beta`：agent / CLI 没法过微服登录，这两条路径由网关自己的共享 Key 把关；控制台留在微服账号鉴权之后，那个固定 Key 只是 magpie 自己的一道门槛
- 数据（账号、API Key、登录态）持久化在 `/lzcapp/var/config`

## 使用

1. 打开控制台：点启动器入口；浏览器直连时用 `https://<域名>/?k=sk-magpie-lazycat-web`（带一次后写入 400 天 Cookie）
2. Settings → Share on local network 打开共享，记下生成的 `sk-magpie-…` Key，然后让 agent 连 `https://<域名>/v1`（OpenAI 兼容）或 `https://<域名>`（Anthropic / Gemini），API Key 填共享 Key。没打开共享前网关接受任意 Key，这是上游设计，所以请先完成这一步
