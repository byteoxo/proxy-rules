# proxy-rules

Curated domain and traffic rules for **Clash**, **Surge**, **Stash**, and other proxy clients that accept plain-text rule lists. Each file is a focused snippet you include in your own config and attach to a policy (`PROXY`, `DIRECT`, `REJECT`, or a proxy group).

Rules target common developer, AI, and daily-use services — especially hosts that are slow or unstable when accessed directly from mainland China.

## Quick start

### Clash / Clash Meta (rule-provider)

```yaml
rule-providers:
  proxy-rules-openai:
    type: http
    behavior: classical
    url: https://raw.githubusercontent.com/byteoxo/proxy-rules/main/surge/ai-chat/openai.txt
    path: ./ruleset/surge/ai-chat/openai.txt
    interval: 86400

rules:
  - RULE-SET,proxy-rules-openai,PROXY
  - MATCH,DIRECT
```

Use a `file` provider instead of `http` if you clone this repo locally.

### Surge

```ini
[Rule]
RULE-SET,https://raw.githubusercontent.com/byteoxo/proxy-rules/main/surge/development/development.txt,PROXY
RULE-SET,https://raw.githubusercontent.com/byteoxo/proxy-rules/main/surge/cn/wechat.txt,DIRECT
FINAL,DIRECT
```

### Stash

Stash supports the same `RULE-SET` URL pattern as Surge for classical rule lists.

## Repository layout

All rule files are now organized under `surge/`.

| Path | Purpose |
|------|---------|
| `surge/ai-chat/openai.txt` | OpenAI / ChatGPT and related CDNs, APIs, and infra |
| `surge/ai.txt` | Other AI services (e.g. Mistral) |
| `surge/cursor.txt` | Cursor IDE domains and macOS process rule |
| `surge/zed.txt` | Zed editor |
| `surge/arc_browser.txt` | Arc browser |
| `surge/tradingview.txt` | TradingView |
| `surge/russia.txt` | `.ru` TLD suffix |
| `surge/development/` | Package registries, Git hosts, language ecosystems |
| `surge/cn/` | China domestic services — typically paired with `DIRECT` |
| `surge/social-media/` | Discord, Bluesky, etc. |
| `surge/crypto/` | Wallet and Web3 tooling |

### `surge/development/`

| File | Contents |
|------|----------|
| `development.txt` | **Umbrella** — registries, CDNs, containers, IDEs, and common install hosts |
| `github.txt` / `gitlab.txt` | Git hosting (subset) |
| `git_platform.txt` | Bitbucket, SourceForge, Codeberg, sr.ht |
| `node.txt` | npm, yarn, pnpm, js CDNs |
| `bun.txt` / `deno.txt` | Bun and Deno / JSR |
| `rust.txt` / `go.txt` / `python.txt` / `java.txt` | Language-specific registries |
| `lovart.txt` | Lovart AI |

Pick the umbrella file for broad coverage, or include only the slices you need to avoid rule bloat.

### `surge/cn/`

These lists are meant for **direct** (non-proxy) routing:

| File | Contents |
|------|----------|
| `wechat.txt` | WeChat / Weixin (from [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)) |
| `12306.txt` | China Railway / 12306 (from blackmatrix7) |
| `feishu.txt` | Feishu / Lark |
| `package.txt` | Domestic mirrors (e.g. `npmmirror.com`) |

Place `surge/cn/*` rules **above** your catch-all proxy rules so domestic traffic stays local.

## Rule format

Files use the classical syntax shared by Clash, Surge, and Stash:

```
DOMAIN-SUFFIX,example.com
DOMAIN,exact.example.com
DOMAIN-KEYWORD,keyword
PROCESS-NAME,/Applications/SomeApp.app/
IP-CIDR,1.2.3.4/32,no-resolve
IP-CIDR6,2001:db8::/32,no-resolve
IP-ASN,13335,no-resolve
```

- Lines starting with `#` are comments.
- Rule lines do **not** include a policy suffix — you choose `PROXY` / `DIRECT` when wiring the file into your config.
- Prefer `DOMAIN-SUFFIX` for stable apex domains; use `DOMAIN` for specific hostnames or CDN edges.
- Add `no-resolve` on IP-based rules to skip DNS resolution (recommended).

## Recommended policy pairing

| Rule set | Typical policy | Why |
|----------|----------------|-----|
| `surge/ai-chat/openai.txt`, `surge/ai.txt`, `surge/cursor.txt`, `surge/development/*` | `PROXY` | Foreign or blocked / slow services |
| `surge/cn/wechat.txt`, `surge/cn/12306.txt`, `surge/cn/package.txt` | `DIRECT` | Domestic services and mirrors |
| `surge/cn/feishu.txt` | `DIRECT` | Local app + `.cn` domains |
| `surge/social-media/*`, `surge/crypto/*` | `PROXY` | Usually accessed via proxy |

Adjust to your network — these are conventions, not requirements.

## Updating

Subscribe to raw GitHub URLs with a daily `interval` (Clash) or refresh Surge/Stash rule sets periodically. For air-gapped or pinned setups, clone the repo and point providers at local paths.

## Contributing

See [AGENT.md](./AGENT.md) for file conventions, naming, and how to add or split rule sets. Pull requests and issue reports for missing domains are welcome.

## License

Rule snippets adapted from third-party lists retain attribution in file headers where applicable. See individual files for source credits.
