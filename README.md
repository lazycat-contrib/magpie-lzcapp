# magpie-lzcapp

[Magpie](https://github.com/yetone/magpie) 的懒猫微服打包：把各家大模型接到一处，给 Codex / Claude Code / OpenCode 等 agent 提供统一网关（兼容 OpenAI / Anthropic / Gemini），并带一个网页控制台。

- 上游镜像：`ghcr.io/yetone/magpie`（官方商店交付走 `delivery.mode: lazycat`，由 Action 转存到 `registry.lazycat.cloud` 后改写 manifest）
- 一个容器两种角色：镜像默认 `serve` 只跑网关；这里用 `magpie web --addr 0.0.0.0:3430 --no-open`，控制台与网关一起跑
- 路由：`/` → 控制台（3430），`/v1`、`/v1beta` → 网关（3425）；网关的 OpenAI / Anthropic / Gemini 路径都落在这两个前缀下
- 网关监听地址不覆盖（沿用镜像默认 `MAGPIE_ADDR=0.0.0.0:3425`），微服入口才能把 `/v1`、`/v1beta` 转发进来
- `user: root`：镜像以 nonroot 运行，而 `/lzcapp/var/config` 由平台以 root 创建，用 root 启动保证配置可写
- 控制台 Key：安装向导参数 `web_key`（默认 `sk-magpie-lazycat-web`）。magpie 的控制台必须带 `?k=<Key>`（或对应 Cookie `magpie_web_3430`），而启动器 entry 不能渲染模板参数，所以用 `on: request` 的 inject 在请求 `/` 时 303 到 `/?k=<web_key>`，客户端打开即自动带上；`/v1*`、`/api/*`、静态资源不碰
- `public_path: [/]`：整个应用关闭微服账号鉴权（agent / CLI / 手机不必先过微服登录）。控制台改由 Web Key 把门——客户端打开由 inject 自动带上，其他入口用 `/?k=<Web Key>`；网关由控制台的「Share on local network」共享 Key 把门，没打开共享前网关接受任意 Key（上游设计）
- 数据（账号、API Key、登录态）持久化在 `/lzcapp/var/config`

## 使用

1. 打开控制台：懒猫客户端点启动器入口即可（inject 会把 Key 注入到地址）；其他浏览器/工具手动访问 `https://<域名>/?k=<Web Key>`，带一次后写 400 天 Cookie
2. Settings → Share on local network 打开共享，记下生成的 `sk-magpie-…` Key，然后让 agent 连 `https://<域名>/v1`（OpenAI 兼容）或 `https://<域名>`（Anthropic / Gemini），API Key 填共享 Key。没打开共享前网关接受任意 Key，这是上游设计，所以请先完成这一步

注：`ctx` 注入只在懒猫客户端链路上生效，其他入口覆盖不到——那些入口用手动 `?k=` 那条路。
