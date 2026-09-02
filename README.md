# ladder

个人自用的代理分流规则，同时维护 Clash 与 Quantumult X 两套配置。

参考来源：墨鱼 <https://github.com/ddgksf2013>

> 重要声明：版权归原作者所有，此处的修改只供个人使用。

## 目录结构

| 路径 | 说明 |
| --- | --- |
| `shellclash/ShellClash.ini` | Clash 配置（subconverter 外部配置），本仓库的基准 |
| `shellclash/*.list` | Clash 语法的规则列表 |
| `quanx/QuantumultX.conf` | Quantumult X 配置 |
| `quanx/rules/*.list` | Quantumult X 语法的规则列表 |
| `quanx/icon/` | 自定义策略图标 |

## 维护约定

`shellclash` 是基准，`quanx` 向它看齐。

本仓库只保存有自定义内容的列表。广告拦截、微信、TikTok、苹果、电报、流媒体、ASN.China 等未经修改的列表一律直接引用上游链接，不再保存快照——否则快照会逐渐过期，而且两侧容易漂移。

两侧的自定义列表内容保持一致，只有规则类型的前缀不同，因此可以直接 diff 校验：

```bash
# 校验某个列表两侧是否一致（应无输出）
diff <(sed -E 's/^HOST(-SUFFIX|-KEYWORD)?,/DOMAIN\1,/' quanx/rules/AI.list) shellclash/AI.list
```

修改规则时，先改 `shellclash/` 下的列表，再用下面的命令同步到 `quanx/rules/`：

```bash
for f in AI Crypto CryptoHK Feishu CustomProxy CustomDirect GoogleVoice; do
  sed -E 's/^DOMAIN(-SUFFIX|-KEYWORD)?,/HOST\1,/' "shellclash/$f.list" > "quanx/rules/$f.list"
done
```

### 语法对照

| Clash | Quantumult X |
| --- | --- |
| `DOMAIN` | `HOST` |
| `DOMAIN-SUFFIX` | `HOST-SUFFIX` |
| `DOMAIN-KEYWORD` | `HOST-KEYWORD` |
| `IP-CIDR` / `IP-ASN` / `USER-AGENT` | 同名，无需转换 |
| `DST-PORT` | 无对应类型，无法移植 |

Quantumult X 支持的规则类型只有 `HOST`、`HOST-SUFFIX`、`HOST-WILDCARD`、`HOST-KEYWORD`、`USER-AGENT`、`IP-CIDR`、`IP6-CIDR`、`GEOIP`、`IP-ASN`。因此 `shellclash/Port.list`（`DST-PORT,22`）只存在于 Clash 侧。

### 策略对照

| ShellClash | Quantumult X |
| --- | --- |
| `🛑 广告拦截` | `reject` |
| `🎯 全球直连` | `direct` |
| `🏃‍♂️ 全球加速` | `全球加速` |
| `🌍 国外媒体` | `国际媒体` |
| `🐟 漏网之鱼` | `兜底分流` |
| `🤖 人工智能` / `📲 电报消息` / `🍎 苹果服务` / `💰 Crypto` / `🪙 CryptoHK` / `🇺🇲 美国节点` | 同名 |

`声田音乐`（Spotify）与 `哔哩哔哩` 是 Quantumult X 侧独有的策略，Clash 侧没有对应项。

## 规则顺序

两侧都按同样的顺序组织，顺序即优先级：

1. **规则修正与自定义覆盖** —— `Unbreak`、`CustomDirect`、`CustomProxy`。必须置顶，否则会被后面的广告拦截规则抢先命中，起不到修正误杀的作用。
2. **广告拦截** —— 引用上游 AdRules 与 ACL4SSR BanAD。
3. **分流策略** —— 微信、飞书、TikTok、AI、流媒体、苹果、电报等。其中飞书必须排在 TikTok 之前：两者共用 `snssdk.com`、`ttwebview.com`、`bytedapm.com`，而 TikTok 国际版用的是 `isnssdk.com`，不受影响。
4. **Crypto** —— `Crypto.list` 走可选节点，`CryptoHK.list` 强制走香港节点（与实名身份相关）。
5. **直连与兜底** —— ASN.China、GEOIP CN，最后 FINAL 兜底。

规则列表里应避免使用过宽的 `DOMAIN-KEYWORD` 与通用后缀。例如 `DOMAIN-KEYWORD,gemini` 会把加密货币交易所 `gemini.com` 一并吃掉，`DOMAIN-SUFFIX,googleapis.com` 会把 Play 商店、Firebase、地图等全部卷入 AI 策略。

## Crypto 相关改动

相对原作者配置，这里把 crypto 单独拆了出来：

- `Crypto.list` → `💰 Crypto` / `Crypto` 策略，可在台湾、香港、日本、狮城、美国节点间切换。
- `CryptoHK.list` → `🪙 CryptoHK` / `CryptoHK` 策略，只允许香港节点。

同时把全球加速的规则中与 crypto 重复的部分交由上面两个列表处理。
