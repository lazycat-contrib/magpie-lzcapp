# magpie-lzcapp

[Magpie](https://github.com/yetone/magpie) 的懒猫微服打包：把各家大模型接到一处，给 Codex / Claude Code / OpenCode 等 agent 提供统一网关（兼容 OpenAI / Anthropic / Gemini），并带一个网页控制台。

- 上游镜像：`ghcr.io/yetone/magpie`（官方商店交付走 `delivery.mode: lazycat`，由 Action 转存到 `registry.lazycat.cloud` 后改写 manifest）
- 一个容器两种角色：镜像默认 `serve` 只跑网关；这里用 `magpie web --addr 0.0.0.0:3430 --no-open`，控制台与网关一起跑
- 路由：`/` → 控制台（3430），`/v1`、`/v1beta` → 网关（3425）；网关的 OpenAI / Anthropic / Gemini 路径都落在这两个前缀下
- 网关监听地址不覆盖（沿用镜像默认 `MAGPIE_ADDR=0.0.0.0:3425`），微服入口才能把 `/v1`、`/v1beta` 转发进来
- `user: root`：镜像以 nonroot 运行，而 `/lzcapp/var/config` 由平台以 root 创建，用 root 启动保证配置可写
- `public_path` 只放 `/v1`、`/v1beta`：agent / CLI 没法过微服登录，这两条路径由网关自己的共享 Key 把关；控制台仍留在微服账号鉴权之后
- Web Key：安装向导参数，默认 `sk-magpie-lazycat-web`。entry 不渲染模板参数、也不该写死 Key，所以改用 inject：请求阶段按 `web_key` 的值给控制台上游补 `Cookie: magpie_web_3430=<key>`，打开应用即进控制台；另有一条 `on: response` 兜底，把未带 Key 的 401 改写成带 Key 的 303。改 Key 无需同步入口
- 数据（账号、API Key、登录态）持久化在 `/lzcapp/var/config`

## 安装后要做的两步

1. 打开控制台（点启动器入口或直接打开应用域名即可，服务端会自动补上 Web Key）。需要手动登录时用 `https://<域名>/?k=<Web Key>`。
2. Settings → Share on local network 打开共享，记下生成的 `sk-magpie-…` Key，然后让 agent 连 `https://<域名>/v1`（OpenAI 兼容）或 `https://<域名>`（Anthropic / Gemini），API Key 填共享 Key。没打开共享前网关接受任意 Key，这是上游设计，所以请先完成这一步。
