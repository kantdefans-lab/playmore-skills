# CNKI Skills for Claude Code

[English](#english) | [涓枃](#涓枃)

| WeChat Official Account (鍏紬鍙? | WeChat Group (寰俊缇? | Discord |
|:---:|:---:|:---:|
| <img src="qrcode_for_gh_a1c14419b847_258.jpg" width="200"> | <img src="0320.jpg" width="200"> | [Join Discord](https://discord.gg/tGd5vTDASg) |
| 鏈潵璁烘枃瀹為獙瀹?| 鎵爜鍔犲叆浜ゆ祦缇?| English & Chinese |

---

<a id="english"></a>

## English

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills that let Claude interact with [CNKI (涓浗鐭ョ綉)](https://www.cnki.net) through Chrome DevTools MCP.

Search papers, browse journals, check indexing status, resolve open full-text links, and export to Zotero 鈥?all from the Claude Code CLI.

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Chrome browser (login handled manually for downloads)
- [Zotero](https://www.zotero.org/) desktop app (optional, for export)
- Python 3 (optional, for Zotero push script)

### Skills

| Skill | Description | Invocation |
|-------|-------------|------------|
| `cnki-search` | Keyword search with structured result extraction | `/cnki-search 浜哄伐鏅鸿兘` |
| `cnki-advanced-search` | Filtered search: author, journal, date, source category (SCI/EI/CSSCI/鍖楀ぇ鏍稿績/CSCD) | `/cnki-advanced-search CSSCI 浜哄伐鏅鸿兘 2020-2025` |
| `cnki-parse-results` | Re-parse an existing search results page | `/cnki-parse-results` |
| `cnki-navigate-pages` | Pagination and sort order | `/cnki-navigate-pages next` |
| `cnki-paper-detail` | Extract full paper metadata (abstract, keywords, etc.) | `/cnki-paper-detail` |
| `cnki-journal-search` | Find journals by name, ISSN, or CN | `/cnki-journal-search 璁＄畻鏈哄鎶 |
| `cnki-journal-index` | Check journal indexing status and impact factors | `/cnki-journal-index 璁＄畻鏈哄鎶 |
| `cnki-journal-toc` | Browse issue table of contents | `/cnki-journal-toc 璁＄畻鏈哄鎶?2025 01鏈焋 |
| `cnki-download` | Resolve open full-text link (PDF preferred) | `/cnki-download` |
| `cnki-export` | Push to Zotero or output GB/T 7714 citation | `/cnki-export zotero` |

### Agent

**`cnki-researcher`** - orchestrates all 10 skills with link-first behavior by default. For bulk or noisy link extraction, it can delegate to `cnki-link-agent` and return concise "title + link" output.

### Installation

#### 1. Install Chrome DevTools MCP server

```bash
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

#### 2. Install CNKI skills

```bash
git clone https://github.com/cookjohn/cnki-skills.git
cd cnki-skills
cp -r skills/ agents/ .claude/
```

Or add to an existing project:

```bash
git clone https://github.com/cookjohn/cnki-skills.git /tmp/cnki-skills
cp -r /tmp/cnki-skills/skills/ your-project/.claude/skills/
cp -r /tmp/cnki-skills/agents/ your-project/.claude/agents/
鈹溾攢鈹€ cnki-researcher.md              # Agent: search/sort orchestration
鈹斺攢鈹€ cnki-link-agent.md              # Agent: dedicated CNKI link extraction
```

---

<a id="涓枃"></a>

## 涓枃

| 鍏紬鍙?| 寰俊浜ゆ祦缇?| Discord |
|:---:|:---:|:---:|
| <img src="qrcode_for_gh_a1c14419b847_258.jpg" width="200"> | <img src="0320.jpg" width="200"> | [鍔犲叆 Discord](https://discord.gg/tGd5vTDASg) |
| 鏈潵璁烘枃瀹為獙瀹?| 鎵爜鍔犲叆浜ゆ祦缇?| 涓嫳鏂囦氦娴?|

璁?[Claude Code](https://docs.anthropic.com/en/docs/claude-code) 閫氳繃 Chrome DevTools MCP 鎿嶄綔 [涓浗鐭ョ綉 (CNKI)](https://www.cnki.net) 鐨勬妧鑳介泦銆?

鏀寔璁烘枃妫€绱€佹湡鍒婃祻瑙堛€佹敹褰曟煡璇€丳DF 涓嬭浇銆佸鍑哄埌 Zotero 绛夊姛鑳斤紝鍏ㄩ儴鍦?Claude Code 鍛戒护琛屼腑瀹屾垚銆?

### 鍓嶇疆瑕佹眰

- 宸插畨瑁?[Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Chrome 娴忚鍣紙涓嬭浇鍔熻兘闇€鎵嬪姩鐧诲綍鐭ョ綉锛?
- [Zotero](https://www.zotero.org/) 妗岄潰绔紙鍙€夛紝鐢ㄤ簬瀵煎嚭锛?
- Python 3锛堝彲閫夛紝鐢ㄤ簬 Zotero 鎺ㄩ€佽剼鏈級

### 鎶€鑳藉垪琛?

| 鎶€鑳?| 鍔熻兘 | 璋冪敤鏂瑰紡 |
|------|------|----------|
| `cnki-search` | 鍏抽敭璇嶆绱紝杩斿洖缁撴瀯鍖栫粨鏋?| `/cnki-search 浜哄伐鏅鸿兘` |
| `cnki-advanced-search` | 楂樼骇妫€绱細浣滆€呫€佹湡鍒娿€佹椂闂淬€佹潵婧愮被鍒紙SCI/EI/CSSCI/鍖楀ぇ鏍稿績/CSCD锛?| `/cnki-advanced-search CSSCI 浜哄伐鏅鸿兘 2020-2025` |
| `cnki-parse-results` | 閲嶆柊瑙ｆ瀽褰撳墠鎼滅储缁撴灉椤?| `/cnki-parse-results` |
| `cnki-navigate-pages` | 缈婚〉涓庢帓搴?| `/cnki-navigate-pages next` |
| `cnki-paper-detail` | 鎻愬彇璁烘枃璇︾粏淇℃伅锛堟憳瑕併€佸叧閿瘝绛夛級 | `/cnki-paper-detail` |
| `cnki-journal-search` | 鎸夊悕绉般€両SSN銆丆N 鍙锋煡鎵炬湡鍒?| `/cnki-journal-search 璁＄畻鏈哄鎶 |
| `cnki-journal-index` | 鏌ヨ鏈熷垔鏀跺綍鎯呭喌鍜屽奖鍝嶅洜瀛?| `/cnki-journal-index 璁＄畻鏈哄鎶 |
| `cnki-journal-toc` | 娴忚鏈熷垔鐩綍 | `/cnki-journal-toc 璁＄畻鏈哄鎶?2025 01鏈焋 |
| `cnki-download` | 涓嬭浇璁烘枃 PDF/CAJ | `/cnki-download` |
| `cnki-export` | 鎺ㄩ€佸埌 Zotero 鎴栬緭鍑?GB/T 7714 寮曠敤 | `/cnki-export zotero` |

### 鏅鸿兘浣?

**`cnki-researcher`** 鈥?缁熶竴璋冨害鍏ㄩ儴 10 涓妧鑳姐€傝嚜鍔ㄥ鐞嗛獙璇佺爜妫€娴嬶紙鏆傚仠骞舵彁绀虹敤鎴锋墜鍔ㄥ畬鎴愶級銆佹爣绛鹃〉绠＄悊锛屾敮鎸?妫€绱?鈫?绛涢€?鈫?瀵煎嚭鍒?Zotero"绛夊鍚堝伐浣滄祦銆?

### 瀹夎鏂规硶

#### 1. 瀹夎 Chrome DevTools MCP 鏈嶅姟鍣?

```bash
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

#### 2. 瀹夎 CNKI 鎶€鑳?

```bash
git clone https://github.com/cookjohn/cnki-skills.git
cd cnki-skills
cp -r skills/ agents/ .claude/
```

娣诲姞鍒板凡鏈夐」鐩細

```bash
git clone https://github.com/cookjohn/cnki-skills.git /tmp/cnki-skills
cp -r /tmp/cnki-skills/skills/ your-project/.claude/skills/
cp -r /tmp/cnki-skills/agents/ your-project/.claude/agents/
```

#### 3. 鍚姩 Chrome 杩滅▼璋冭瘯

```bash
# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222

# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222
```

#### 4. 鍚姩 Claude Code

```bash
claude
```

鎶€鑳藉拰鏅鸿兘浣撲細鑷姩鍔犺浇銆傝緭鍏?`/cnki-search 娣卞害瀛︿範` 楠岃瘉鏄惁姝ｅ父銆?

### 宸ヤ綔鍘熺悊

鎵€鏈夋妧鑳介€氳繃 Chrome DevTools MCP 鐨?`evaluate_script` 寮傛鎵ц JavaScript锛屾棤闇€鎴浘璇嗗埆鎴?OCR銆傛瘡涓搷浣滀粎闇€ 1-2 娆″伐鍏疯皟鐢紙瀵艰埅 + 鎵ц鑴氭湰锛夛紝蹇€熶笖绋冲畾銆?

鏍稿績璁捐锛?
- **鍗曟寮傛鑴氭湰** 鈥?鍙栦唬澶氭楠ょ殑 snapshot 鈫?click 鈫?wait_for 妯″紡
- **鐩存帴瀵艰埅浼樹簬鐐瑰嚮閾炬帴** 鈥?鐭ョ綉閾炬帴浼氭墦寮€鏂版爣绛鹃〉锛岀洿鎺ュ鑸伩鍏嶆爣绛鹃〉绠＄悊寮€閿€
- **鎵归噺瀵煎嚭** 鈥?浠庢悳绱㈢粨鏋滈〉涓€娆℃€у鍑哄绡囪鏂囷紝鏃犻渶閫愮瘒杩涘叆璇︽儏椤?
- **楠岃瘉鐮佹劅鐭?* 鈥?妫€娴嬪埌鑵捐婊戝潡楠岃瘉鐮佹椂鑷姩鏆傚仠锛岀瓑寰呯敤鎴锋墜鍔ㄥ鐞?

---

## License

MIT

