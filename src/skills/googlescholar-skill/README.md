# Google Scholar Skills for Claude Code

[English](#english) | [涓枃](#涓枃)

| WeChat Official Account (鍏紬鍙? | WeChat Group (寰俊缇? | Discord |
|:---:|:---:|:---:|
| <img src="qrcode_for_gh_a1c14419b847_258.jpg" width="200"> | <img src="0320.jpg" width="200"> | [Join Discord](https://discord.gg/tGd5vTDASg) |
| 鏈潵璁烘枃瀹為獙瀹?| 鎵爜鍔犲叆浜ゆ祦缇?| English & Chinese |

---

<a id="english"></a>

## English

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills that let Claude interact with [Google Scholar](https://scholar.google.com) through Chrome DevTools MCP.

Search papers, track citations, get full-text links, and export to Zotero 鈥?all from the Claude Code CLI.

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Chrome browser with remote debugging enabled
- [Zotero](https://www.zotero.org/) desktop app (optional, for export)
- Python 3 (optional, for Zotero push script)

### Skills

| Skill | Description | Invocation |
|-------|-------------|------------|
| `gs-search` | Keyword search with structured result extraction | `/gs-search deep learning` |
| `gs-advanced-search` | Filtered search: author, journal, date range, exact phrase, title-only | `/gs-advanced-search author:Einstein after:2020 relativity` |
| `gs-cited-by` | Find papers that cite a given paper via data-cid | `/gs-cited-by 0qfs6zbVakoJ` |
| `gs-fulltext` | Resolve open full-text link (PDF or publisher page) | `/gs-fulltext 0qfs6zbVakoJ` |
| `gs-navigate-pages` | Pagination for search results | `/gs-navigate-pages next` |
| `gs-export` | Export to Zotero via BibTeX extraction | `/gs-export 0qfs6zbVakoJ` |

### Agent

**`gs-researcher`** - orchestrates all 6 skills with link-first behavior by default. For bulk or noisy link extraction, it can delegate to `gs-link-agent` and return concise "title + link" output.

### Installation

#### 1. Install Chrome DevTools MCP server

```bash
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

#### 2. Install Google Scholar skills

```bash
git clone https://github.com/cookjohn/gs-skills.git
cd gs-skills
cp -r skills/ agents/ .claude/
```

Or add to an existing project:

```bash
git clone https://github.com/cookjohn/gs-skills.git /tmp/gs-skills
cp -r /tmp/gs-skills/skills/ your-project/.claude/skills/
cp -r /tmp/gs-skills/agents/ your-project/.claude/agents/
鈹溾攢鈹€ gs-researcher.md                # Agent: search/sort orchestration
鈹斺攢鈹€ gs-link-agent.md                # Agent: dedicated Scholar link extraction
```

---

<a id="涓枃"></a>

## 涓枃

| 鍏紬鍙?| 寰俊浜ゆ祦缇?| Discord |
|:---:|:---:|:---:|
| <img src="qrcode_for_gh_a1c14419b847_258.jpg" width="200"> | <img src="0320.jpg" width="200"> | [鍔犲叆 Discord](https://discord.gg/tGd5vTDASg) |
| 鏈潵璁烘枃瀹為獙瀹?| 鎵爜鍔犲叆浜ゆ祦缇?| 涓嫳鏂囦氦娴?|

璁?[Claude Code](https://docs.anthropic.com/en/docs/claude-code) 閫氳繃 Chrome DevTools MCP 鎿嶆帶 [Google Scholar (璋锋瓕瀛︽湳)](https://scholar.google.com) 鐨勬妧鑳介泦銆?

鏀寔璁烘枃鎼滅储銆佸紩鐢ㄨ拷韪€佸叏鏂囪幏鍙栥€佸鍑哄埌 Zotero 绛夊姛鑳斤紝鍏ㄩ儴鍦?Claude Code 鍛戒护琛屼腑瀹屾垚銆?

### 鍓嶇疆瑕佹眰

- 宸插畨瑁?[Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Chrome 娴忚鍣紙闇€寮€鍚繙绋嬭皟璇曪級
- [Zotero](https://www.zotero.org/) 妗岄潰鐗堬紙鍙€夛紝鐢ㄤ簬瀵煎嚭锛?
- Python 3锛堝彲閫夛紝鐢ㄤ簬 Zotero 鎺ㄩ€佽剼鏈級

### 鎶€鑳藉垪琛?

| 鎶€鑳?| 鎻忚堪 | 璋冪敤鏂瑰紡 |
|------|------|----------|
| `gs-search` | 鍏抽敭璇嶆悳绱紝杩斿洖缁撴瀯鍖栫粨鏋?| `/gs-search deep learning` |
| `gs-advanced-search` | 楂樼骇鎼滅储锛氫綔鑰呫€佹湡鍒娿€佹椂闂淬€佺簿纭煭璇€佷粎鏍囬 | `/gs-advanced-search author:Einstein after:2020 relativity` |
| `gs-cited-by` | 寮曠敤杩借釜锛氭煡鎵惧紩鐢ㄤ簡鏌愮瘒璁烘枃鐨勬墍鏈夋枃鐚?| `/gs-cited-by 0qfs6zbVakoJ` |
| `gs-fulltext` | 鍏ㄦ枃鑾峰彇锛歅DF銆丏OI銆丼ci-Hub銆佸嚭鐗堝晢閾炬帴 | `/gs-fulltext 0qfs6zbVakoJ` |
| `gs-navigate-pages` | 鎼滅储缁撴灉缈婚〉 | `/gs-navigate-pages next` |
| `gs-export` | 閫氳繃 BibTeX 瀵煎嚭鍒?Zotero | `/gs-export 0qfs6zbVakoJ` |

### 鏅鸿兘浣?

**`gs-researcher`** 鈥?缁熶竴璋冨害鍏ㄩ儴 6 涓妧鑳姐€傝嚜鍔ㄦ娴嬮獙璇佺爜锛堟殏鍋滃苟鎻愮ず鐢ㄦ埛鎵嬪姩瀹屾垚锛夛紝鏀寔"鎼滅储 鈫?寮曠敤杩借釜 鈫?瀵煎嚭鍒?Zotero"绛夊鍚堝伐浣滄祦銆?

### 瀹夎姝ラ

#### 1. 瀹夎 Chrome DevTools MCP 鏈嶅姟鍣?

```bash
claude mcp add chrome-devtools -- npx -y chrome-devtools-mcp@latest
```

#### 2. 瀹夎 Google Scholar 鎶€鑳?

```bash
git clone https://github.com/cookjohn/gs-skills.git
cd gs-skills
cp -r skills/ agents/ .claude/
```

鎴栨坊鍔犲埌宸叉湁椤圭洰锛?

```bash
git clone https://github.com/cookjohn/gs-skills.git /tmp/gs-skills
cp -r /tmp/gs-skills/skills/ your-project/.claude/skills/
cp -r /tmp/gs-skills/agents/ your-project/.claude/agents/
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

鎶€鑳藉拰鏅鸿兘浣撲細鑷姩鍔犺浇銆傝緭鍏?`/gs-search deep learning` 楠岃瘉鏄惁姝ｅ父宸ヤ綔銆?

### 宸ヤ綔鍘熺悊

鎵€鏈夋妧鑳介€氳繃 Chrome DevTools MCP 鐨?`evaluate_script` 寮傛鎵ц JavaScript锛屾棤闇€鎴浘璇嗗埆鎴?OCR銆傛瘡涓妧鑳戒粎闇€ 1-2 娆″伐鍏疯皟鐢紙瀵艰埅 + 鎵ц鑴氭湰锛夛紝蹇€熶笖绋冲畾銆?

鏍稿績璁捐锛?
- **绾?DOM 瑙ｆ瀽** 鈥?Google Scholar 鏃犲叕寮€ API锛屾墍鏈夋暟鎹€氳繃 CSS 閫夋嫨鍣ㄦ彁鍙?
- **`data-cid` 浣滀负涓婚敭** 鈥?闆嗙兢 ID 璐┛鎵€鏈夋妧鑳斤紝鐢ㄤ簬寮曠敤杩借釜銆佸鍑哄拰浜ゅ弶寮曠敤
- **鍗曟寮傛鑴氭湰** 鈥?鍙栦唬澶氭楠?snapshot 鈫?click 鈫?wait_for 妯″紡
- **navigate_page 鑾峰彇 BibTeX** 鈥?缁曡繃 `scholar.googleusercontent.com` 鐨?CORS 闄愬埗
- **楠岃瘉鐮佹劅鐭?* 鈥?妫€娴嬪埌 Google Scholar 楠岃瘉鐮佹椂鑷姩鏆傚仠锛岀瓑寰呯敤鎴锋墜鍔ㄥ畬鎴?

---

## License

MIT

