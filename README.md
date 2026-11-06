# emos-auto-public

[![emos-AutoCheckIn](https://github.com/YYluck06/emos-auto-public/actions/workflows/emos-AutoCheckIn.yml/badge.svg)](https://github.com/YYluck06/emos-auto-public/actions/workflows/emos-AutoCheckIn.yml)
[![emos-StatusWatch](https://github.com/YYluck06/emos-auto-public/actions/workflows/emos-StatusWatch.yml/badge.svg)](https://github.com/YYluck06/emos-auto-public/actions/workflows/emos-StatusWatch.yml)

[emos](https://emos.best)（somebyte 生态的公益影音服）日常任务 —— **公开 runner 库**。

> 本库**没有一行业务代码**：只有 GitHub Actions workflow + 一份非敏感状态（`emos/`）。
> 真正的脚本、文档、签到的历史状态都在**私有库** [`YYluck06/emos-auto-core`](https://github.com/YYluck06/emos-auto-core)，
> 运行时用 `actions/checkout` + 只读 PAT 把它拉到 `app/` 再执行。

## 两个 workflow

| workflow | cron（UTC） | 北京时间 | 做什么 |
| --- | --- | --- | --- |
| `emos-AutoCheckIn` | `40 0 * * *` | 08:40 | 每日签到（+1~5 萝卜）→ 快照 → 回写状态 → 可选 Telegram |
| `emos-StatusWatch` | `0 13 * * *` | 21:00 | 只采集快照，萝卜到 800 / 3000 / 100 门槛时提醒 |

两个都可以在 Actions 页面手动 `Run workflow`。

## 需要配的 Secrets

| Secret | 必填 | 说明 |
| --- | --- | --- |
| `CORE_PAT` | ✅ | 能**只读**克隆私有库 `emos-auto-core` 的 PAT（fine-grained: Contents = Read） |
| `EMOS_TOKEN` | ✅ | emos 面板 → 点左上角「欢迎」→「当前密钥」，形如 `4005_xxxxxxxxxxxxxxxx` |
| `TG_BOT_TOKEN` | ⬜ | 想收 Telegram 通知就填 bot token |
| `TG_CHAT_ID` | ⬜ | 同上，你的 chat id |
| `EMOS_PROXY` | ⬜ | **出口 IP**：`http://user:pass@1.2.3.4:8080` / `socks5h://1.2.3.4:1080` / `1.2.3.4:8080`；留空=直连 |

可选仓库变量：`EMOS_MOTTO`（签到寄语，≤10 字）、`EMOS_PROXY`（当默认出口，secret 优先）。

## 代理出口（可以每个库单独配）

- 想换出口 IP 就给本库加 `EMOS_PROXY`，**只影响本库**——想「库 A 走节点 1、库 B 走节点 2」，在各自的库里配各自的，workflow 不用改。
- 支持 `http` / `https` / `socks5` / `socks5h` / `socks4` / `socks4a`，裸 `host:port` 按 `http://` 处理。
- `hysteria2://` `vless://` `tuic://` `ss://` 这类分享链接 curl 不认：先用 sing-box 转成本地 socks5 再填。
- **配了代理却连不通 → 任务直接失败（exit 3），不会偷偷直连**，避免用 runner 的真实 IP 打站。
- 每次运行都会打印并记录出口 IP（`emos/state.json` 的 `egress` 字段、Actions 摘要里的「出口」一行）。

## 状态文件

```
emos/time.txt       # 最近一次运行的一行摘要
emos/state.json     # 最近一次快照（萝卜/境界/观影资格/邀请名额/卡槽）
emos/status.json    # StatusWatch 产出的完整快照 + 告警
```

`EMOS_TOKEN` 永远不会落盘到本库，也不写进任何文件。

## 为什么公开库/私有库要拆开

1. **秘密与代码隔离**：公开库只放 workflow，账号密钥统一放 secrets；私有库放实现细节（接口、坑、备选路径）。
2. **换账号 = 换 secret**，不用改一行代码。
3. **失败互不牵连**：后续要接第二个 emos 账号，直接再开一个公开库 `emos-auto-public-2`，各跑各的 cron，互不影响。

## 设计来源

- `jyucoeng/auto_scripts`：一站点一 workflow、每站 `time.txt` 记上次运行、分站 README。
- 本机上一个项目 `alishushu-renew-core` + `alishushu-renew-1..4`：私有核心库 + 多个公开 runner，状态回写私有库。

细节文档全部在私有库的 `docs/` 里。
