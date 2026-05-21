# Solana 极速解密 / Solana Speed Demons

An interactive single-page course on **Solana validator internals at wire speed** — how Firedancer, Samba, Delorean, and pmm-sim each carve a different optimization out of the 400ms slot. Self-contained HTML, no framework.

中文教程，7 章，从验证节点的全貌一路深入到私有 AMM 解剖：

| # | Module | Topic |
|---|---|---|
| 01 | 验证节点 | The Validator |
| 02 | Firedancer：C 语言引擎 | Firedancer — the C engine |
| 03 | 交易的旅程 | Transaction journey |
| 04 | Samba：极速 MEV | Samba — MEV at wire speed |
| 05 | Delorean：时光机 | Delorean — the time machine |
| 06 | 全局视野 | The bigger picture |
| 07 | pmm-sim：私有 AMM 解剖 | pmm-sim — private AMM dissected |

Companion long-form essay: [aileena.xyz/blog/wire-speed](https://aileena.xyz/blog/wire-speed)

## Build

```bash
bash build.sh
```

Concatenates `_base.html + modules/*.html + _footer.html` into `index.html`. Open in any browser.

## Structure

```
_base.html        Shell, nav, fonts, accent variables
_footer.html      Closing tags
modules/*.html    Chapter content, assembled in numeric order
briefs/*.md       Pre-writing teaching-arc briefs for each module
main.js           Scroll progress, nav-dot active states
styles.css        Layout, typography, animations
build.sh          One-line concat
```

Styling: teal accent (`#2A7B9B`), Bricolage Grotesque + DM Sans + JetBrains Mono via [loli.net](https://fonts.loli.net) mirror.

## Deploy

Static. Drop the directory on any host — GitHub Pages, Vercel, S3, anywhere that serves HTML.
