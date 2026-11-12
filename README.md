# 自建订阅模板（已部署到 .2，2026-11-11）

## 现行基准（部署前实测，已纠正）
现行源配置是 subconverter 模板「Aethersailor 规则 重度分流版 **故障转移**」生成的版本（UCI config_subscribe[1]，当日 15:31 刚重新生成）：63 组（35 fallback / 17 url-test / 11 select，主组为 `🚀 故障转移`，**没有** 🚀手动选择）、63 条 rules（其中 3 条指向故障转移）、6 个远程 rule-providers。
注意：`~/workspace/fallback-fix/` 里 2026-11-10 的备份件是另一个模板变体「Custom_Clash_Full」（select 版），与现行不是同一套，不要混用（旧草稿已存为 `mihomo-template-select-variant.bak.yaml`）。

## 部署结果（2026-11-11 15:46 验收通过）
- 部署版 = 现行源配置逐项对齐 + 内置注入：`🇺🇸 美国UI` + 5 个 `🛟 兜底·*`（原本靠覆写脚本运行时注入，现写进源文件；覆写脚本保留、幂等不重复）+ `⭐ 自定义` 组 + 2 条 Custom_User 规则 + 2 个本地 file 型 rule-providers。
- 落点：直接替换 `/etc/openclash/config/123.yaml`（config_path 不变），并把 `config_subscribe[1].enabled` 置 0（LuCI 可见可改；该订阅原本 auto_update=0，只有点更新才会覆盖）。节点 provider 每小时刷新不受影响。
- 运行配置验收：70 组（= 部署前 69 组 + ⭐自定义，无重复）、90 条 rules（88 + 2 自定义）、providers 含 Custom_User 两个；API 实测 ⭐自定义 alive、now=🚀故障转移；经 7890：gstatic/google 204、chatgpt 200、netflix 302、baidu 直连 200；DNS：google 回 fake A 198.18.2.182 + fake AAAA 2001:2::62a，baidu 回真实国内地址。

## 文件
- `mihomo-template.yaml` —  canonical 模板（节点 URL 为占位符）。
- `live-20261111/deploy-123.yaml` — 已部署原件（含真实节点订阅地址，勿外传）；同目录另有部署前源/运行配置快照与部署后运行配置。
- `rule_provider/Custom_User_Domain.yaml` / `Custom_User_Classical.yaml` — 自定义规则（.2 上在 `/etc/openclash/rule_provider/`），空 payload 起步。

## 自定义怎么改
往两个 Custom_User 文件的 payload 里加域名/classical 规则即可，目标组默认 `⭐ 自定义`（select，现指向 🚀故障转移，可在 API/LuCI 改选 自动选择/全球直连）。改完重启 OpenClash 生效。

## 回滚
.2 上：`config/123.yaml.bak-selftmpl-20261111`（源）、`123.yaml.bak-selftmpl-20261111`（运行）、`/etc/config/openclash.bak-selftmpl-20261111`（UCI，含 subscribe[1] 原 enabled=1）。恢复源文件 + UCI 后重启 OpenClash 即回部署前状态。

## 遗留提醒
若日后在 LuCI 重新启用 config_subscribe[1] 并点更新订阅，源文件会被上游故障转移模板重新覆盖：兜底组靠覆写脚本仍会自动回来，但 ⭐自定义与 Custom_User 会丢，需要按 deploy-123.yaml 重新合并。

## 订阅转换模板（GitHub 链接版，2026-11-11 新增）
基于 Aethersailor「重度分流版」原版 ini 二次定制：上游 63 条 ruleset / 63 个分组逐条保留，仅新增 `⭐ 自定义` 组和 1 条自定义 ruleset（指向本仓库 `rule/Custom_User_Classical.yaml`，在 GitHub 改名单即生效）；ruleset 总数 64，仍在转换后端 max_allowed_rulesets=64 上限内。
- 故障转移版（现用）：https://raw.githubusercontent.com/duoluo1234/clash/main/Custom_Clash_Full_Fallback_Custom.ini
- 手动选择版：https://raw.githubusercontent.com/duoluo1234/clash/main/Custom_Clash_Full_Custom.ini
用法：订阅转换时把 config 参数换成上面的链接即可（OpenClash 订阅转换的自定义模板同理）。
2026-11-11 瘦身：jin 订阅实际只有港/美/日/新/台有节点，已删除 8 个空国家组（韩/加/英/法/德/荷/土/俄）的定义与全部成员引用——空组在 fallback 链里报存活、可能抢在自动选择之前把流量直连放出；分组 64→56。同步适用于 .2 现行原生模板（mihomo-template.yaml，70→62 组）与 .2 已部署源配置（备份 .bak-slim-20261111，重启验收：62 组/90 规则、⭐ 自定义在、韩国组 404、gstatic 204/chatgpt 200/baidu 200）。
2026-11-12 调整（jin 点选）：两份 ini 同步删除 🌐 其他地区、🏠 家宽节点、🐢 低倍率节点三组的定义与全部引用；新增 📌 单节点选择（select，`.*` 展开全部节点逐个可选），挂入 ⭐ 自定义，手动版另挂入 🚀 手动选择。分组 56→54（故障转移版）/ 58→54（手动版，含 2 条注释稿）。已推送后在 .2 触发订阅更新验收（见下行记录）。
