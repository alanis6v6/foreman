# foreman

**一個總指揮，帶一群比較便宜的工人做事。** 給 Claude Code（或任何讀得懂 Markdown skill 的 agent）用的九個小 skill。

[English below](#english)

這套是我們自己每天在用的做法整理出來的：一個主 agent 跟使用者討論、寫工單、驗收；背景的工人改檔、build、截圖；另一家模型查資料；最貴的模型只在少數設計工作上出想法。踩過的坑都寫在每個 skill 裡當例子。

## 九個 skill

| skill | 做什麼 | 什麼時候叫 |
|---|---|---|
| [`route-work`](skills/route-work/SKILL.md) | 省流分工：誰做什麼、用哪個等級的模型、工單多大；總指揮不讀長文，交給只讀工單的無頭工人 | 要派工、用量燒很快 |
| [`brief`](skills/brief/SKILL.md) | 把使用者的一句話變成工人做得對的工單：先討論、原話不轉述、翻成數字；送出前讓零上下文的「金魚」讀一次抓漏洞 | 派工之前 |
| [`project-docs`](skills/project-docs/SKILL.md) | 每個 repo 四份文件（需求、骨架、計畫、技債）＋三四個功能重構一輪 | 開新專案、接手舊專案 |
| [`verify`](skills/verify/SKILL.md) | 說「做完了」之前：對清單、讀 diff、要證據 | 收到工人回報之後 |
| [`design-handoff`](skills/design-handoff/SKILL.md) | 設計交接：可以點的草圖、說出「那一下」、幫效果取名字、美術指導跟施工分開 | 介面改版、動態效果 |
| [`slim-context`](skills/slim-context/SKILL.md) | 精簡 token：量開機讀多少、分必讀／浮現／需要才讀、規則跟紀錄分開、日誌滾動；上下文過半自己摘要、自己換窗 | 用量燒很快、agent 越用越笨 |
| [`chapter-digest`](skills/chapter-digest/SKILL.md) | 歸納章節重點：先逐字摘錄、再用自己的話講、接到讀者身上、留一個問題；寫成文章再逐條對原文 | 一起讀書、整理文件、改寫官方文章 |
| [`nightly-dream`](skills/nightly-dream/SKILL.md) | 每日做夢：收工後便宜的模型只讀重看當天對話，列出沒記到／過時矛盾／答應沒做，隔天主 agent 決定怎麼改 | agent 常忘記講過的事、答應的沒人做 |
| [`boss-mode`](skills/boss-mode/SKILL.md) | 工頭模式：Sonnet 寫程式、更強的 advisor（Opus 或 Fable）在定案前／錯誤重複時／完成前出主意，Haiku 子代理跑腿搜尋、Opus 子代理審查。平常的 `claude` 不受影響，要用時打 `claude-boss` 啟動。安裝見 [`SETUP.md`](skills/boss-mode/SETUP.md) | 任務長、方向很重要，又不想全程用最貴的模型 |

每個都可以單獨用，彼此不互相引用。

## 最省的三個做法

1. **總指揮不讀長文，交給無頭工人**（`route-work`）：要改檔、掃 repo、讀 log，就寫工單開一個全新的無頭工人（例如 `claude -p`）。工人只讀工單、不讀整段對話，回一段短結果；總指揮的上下文只留決定跟驗收。
   官方：「Keep the expensive model's context for the decisions, and let cheaper subagents do the verbose reading.」
   https://claude.dev/blog/which-claude-model-should-you-build-with/
2. **上下文過半自己摘要換窗、每晚做夢**（`slim-context`、`nightly-dream`）：過 50% 就把重點寫進檔案、開新對話，不拖著一大段逐字稿；每天收工讓小模型重讀當天對話，只列出沒記到、過時、答應沒做的。
   官方：https://www.anthropic.com/research/components-of-agent-memory
3. **送出前讓金魚讀一次**（`brief`）：總指揮是大象，什麼都記得，所以看不到自己漏寫什麼。工單送出前給一個零上下文的模型讀，看不懂的先補，比工人做兩次便宜。
   原文：https://github.com/google/elephant-goldfish-model

## 安裝

```bash
git clone https://github.com/alanis6v6/foreman.git
# 全部裝到個人設定（每個專案都能用）
cp -r foreman/skills/* ~/.claude/skills/
# 或只裝到某個專案
cp -r foreman/skills/brief your-project/.claude/skills/
```

在 Claude Code 裡打 `/brief`、`/verify` 這樣叫。

`boss-mode` 多兩步：把子代理複製到 `~/.claude/agents/`，並在 shell 加一段 `claude-boss` 啟動指令。照 [`skills/boss-mode/SETUP.md`](skills/boss-mode/SETUP.md) 做。

## 裝了會怎樣

- **只有你叫它才會動**：每個 skill 都設了 `disable-model-invocation: true`。想讓模型自己判斷什麼時候用，刪掉那一行。
- **沒有 hook、不連網、不跑任何腳本**：全部是 Markdown。唯一的例外是 `boss-mode` 的 `claude-boss`，那是你自己貼進 shell 設定檔的一小段函式，repo 不會替你執行。
- **很短**：每個幾百字，叫一次不會吃掉大半上下文。
- 不會規定你「凡事都要先走 skill」，也不會叫 agent 中途不准問人。

## 語言

skill 本體是繁體中文。模型讀得懂，你也可以直接改成你習慣的說法。

## 授權

MIT

---

## English

**One foreman, a crew of cheaper workers.** Nine small, independent skills for Claude Code (or any agent that reads Markdown skills).

Distilled from how we actually work every day: a lead agent talks with the user, writes work orders and checks results; background workers edit files, build and take screenshots; another vendor's model does research; the most expensive model only gives design direction on a few jobs. Real mistakes we made are written into each skill as examples.

| skill | what it does |
|---|---|
| `route-work` | Who does what, which model tier, how big a work order should be. The lead never does the long reading: headless workers see only the work order. Comes with a `ROLES.md` table you fill in. |
| `brief` | Turn one sentence from the user into a work order a worker gets right: discuss first, quote the user verbatim (never paraphrase imagery), translate every line into numbers. Before sending, a zero-context "goldfish" reads it and flags gaps (Elephant & Goldfish). |
| `project-docs` | Four docs per repo (PRD, architecture, plan, tech debt), an audit brief for existing repos, and a refactor round every 3–4 features. |
| `verify` | Before saying "done": check the acceptance list, read the diff, demand evidence, surface what wasn't verified. |
| `design-handoff` | Clickable sketches first, name the one moment that moves, build a shared vocabulary of effects, separate the art director from the builder, verify motion with video. |
| `slim-context` | Measure what the agent reads at startup, split memory into always / recalled / on-demand, separate rules from logs, roll the journal, read only the lines you need. Past 50% context, save what matters and hand off to a fresh session. |
| `chapter-digest` | Summarize a chapter or long document: verbatim excerpts with locations first, then your own words, a link to the reader, one genuine question; check every claim against the source before publishing. |
| `nightly-dream` | After hours, a cheap read-only model re-reads the day's conversation against the memory files and lists what wasn't saved, what's stale or contradictory, and what was promised but not done. The lead agent decides what to change. |
| `boss-mode` | Foreman mode: Sonnet builds, a stronger advisor (Opus or Fable) weighs in before committing to a plan, when an error recurs, and before declaring done; a Haiku subagent does the searching and an Opus subagent reviews. Plain `claude` stays unchanged; launch with `claude-boss`. Setup in `skills/boss-mode/SETUP.md`. |

**Three biggest savings**

1. **The lead never does the long reading** (`route-work`). Edits, repo scans and log reading go out as work orders to fresh headless workers (e.g. `claude -p`) that see only the order, not the conversation. "Keep the expensive model's context for the decisions, and let cheaper subagents do the verbose reading." https://claude.dev/blog/which-claude-model-should-you-build-with/
2. **Hand off at 50% context, dream at night** (`slim-context`, `nightly-dream`). Past half the window, save what matters to files and start a fresh session; after hours, a small model re-reads the day and lists what wasn't saved, what's stale, and what was promised but not done. https://www.anthropic.com/research/components-of-agent-memory
3. **Let a goldfish read the brief first** (`brief`). The lead remembers everything, so it can't see what it forgot to write down. A zero-context model reads each work order before it ships and flags the gaps. https://github.com/google/elephant-goldfish-model

**Install**: copy the folders in `skills/` into `~/.claude/skills/` (all projects) or `<project>/.claude/skills/` (one project). Invoke with `/brief`, `/verify`, etc. `boss-mode` also needs its subagents copied to `~/.claude/agents/` and a small `claude-boss` shell function; see `skills/boss-mode/SETUP.md`.

**Safe by default**: manual invocation only (`disable-model-invocation: true`), no hooks, no network, no scripts, a few hundred words each. The one exception is `boss-mode`'s `claude-boss`: a short shell function you paste into your own shell config; the repo never runs it.

**Language**: the skill bodies are written in Traditional Chinese. Models read them fine; translate if you prefer.

**License**: MIT
