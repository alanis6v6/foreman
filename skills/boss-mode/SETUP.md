# boss-mode 安裝與使用

平常打 `claude`，一切照舊。需要工頭模式時改打 `claude-boss`，再在對話裡打 `/boss-mode 任務內容`。

| 模式 | 終端機 | 主模型 | advisor | 子代理 |
|---|---|---|---|---|
| 平常 | `claude` | 你原本的設定 | 你原本的設定 | 不會被叫 |
| 工頭 | `claude-boss` | Sonnet | Opus（可改成 Fable） | researcher 用 Haiku、reviewer 用 Opus |

為什麼要分兩步：advisor 只能在**啟動時**用 `--advisor` 指定（只對那一次有效），或用 `/advisor` 設定（會永久存進設定檔）。skill 沒辦法自己開關 advisor，所以由 `claude-boss` 負責啟動、`/boss-mode` 負責工作規則。

## 安裝

需要 Claude Code v2.1.98 以上（advisor 功能）。先 `claude update`。

**1. 複製 skill 和子代理**

```bash
git clone https://github.com/alanis6v6/foreman.git
mkdir -p ~/.claude/skills ~/.claude/agents
cp -r foreman/skills/boss-mode ~/.claude/skills/
cp foreman/skills/boss-mode/agents/*.md ~/.claude/agents/
```

`~/.claude/agents/` 裡如果已經有同名的 `researcher.md` 或 `reviewer.md`，`cp` 會蓋掉。有的話先改名再複製。

**2. 加入 `claude-boss` 啟動指令**

zsh 用 `~/.zshrc`，bash 用 `~/.bashrc`。整段貼進終端機：

```bash
cat >> ~/.zshrc << 'EOF'

# boss-mode：工頭模式啟動指令（https://github.com/alanis6v6/foreman）
claude-boss() {
  local advisor="${BOSS_ADVISOR:-opus}"
  if [[ -n "$BOSS_SWITCH_AT" ]] && (( $(date +%s) >= BOSS_SWITCH_AT )); then
    advisor="${BOSS_ADVISOR_AFTER:-opus}"
  fi
  command claude --model "${BOSS_MODEL:-sonnet}" --advisor "$advisor" "$@"
}
EOF
source ~/.zshrc
```

不設任何變數的話，就是 Sonnet 主模型加 Opus advisor，所有付費方案都能直接用。

**3. 確認**

```bash
claude-boss
```

啟動後應該會看到 `Advisor Tool (experimental) is on`。打 `/` 選單裡會有 `boss-mode`。

在 `/advisor` 選單裡**只看不按 Enter**。按下 Enter 會把選擇永久存進設定檔，之後一般的 `claude` 也會掛上 advisor。

## 設定（選用）

把下面這幾行加進 `~/.zshrc`，放在 `claude-boss` 那段**之前**：

| 變數 | 預設 | 用途 |
|---|---|---|
| `BOSS_MODEL` | `sonnet` | 工頭模式的主模型 |
| `BOSS_ADVISOR` | `opus` | advisor 模型 |
| `BOSS_SWITCH_AT` | 不設 | 到這個時間（Unix 秒數）自動換 advisor |
| `BOSS_ADVISOR_AFTER` | `opus` | 換過去的 advisor |

**用 Fable 當 advisor：**

```bash
export BOSS_ADVISOR=fable
```

Fable 在部分方案是用 usage credits 計費，第一次使用要先在 Claude Code 裡執行 `/model fable` 同意計費，否則 `claude-boss` 一啟動就會報錯退出。

**限時優惠期間用 Fable，到期自動換 Opus：**

```bash
export BOSS_ADVISOR=fable
export BOSS_SWITCH_AT=$(date -j -f "%Y-%m-%d %H:%M" "2026-11-17 15:00" +%s)   # macOS
# Linux：export BOSS_SWITCH_AT=$(date -d "2026-11-17 15:00" +%s)
export BOSS_ADVISOR_AFTER=claude-opus-5-5
```

時間依你電腦的時區計算。建議比截止時間提早一點：advisor 是在啟動時決定的，截止前開的 session 過了截止時間還是會繼續用原本的 advisor。session 開著跨過截止時間的話，在裡面打 `/advisor opus` 手動切換（這一下會存進設定檔，用完記得 `/advisor off`）。

**advisor 要比主模型強或一樣強**，不然不會掛上去。Sonnet 5.5 主模型可以接 Fable、Opus 5 以上、Sonnet 5.5。完整配對表見官方文件：https://code.claude.com/docs/en/advisor

## 使用

```bash
cd 你的專案
claude-boss
```

```
/boss-mode 幫我把登入頁改成 OAuth
```

**想先讓強模型親自跟你討論方向**（advisor 不會跟你對話，它只把建議丟給主模型，按 `Ctrl+O` 才看得到）：

```
/model fable          ← 或 /model opus
Shift+Tab             ← plan mode，只討論、不動檔案
（討論到滿意）
「把定案的方案寫成 PLAN.md」
/model sonnet
/boss-mode 照 PLAN.md 執行
```

先存檔再換模型。對話中途換模型會讓快取失效，所以討論完一次換就好，不要來回切。

## 移除

```bash
rm -r ~/.claude/skills/boss-mode
rm ~/.claude/agents/researcher.md ~/.claude/agents/reviewer.md
```

再打開 `~/.zshrc`，刪掉 `# boss-mode` 開頭的那段，以及你加的 `BOSS_` 變數。
