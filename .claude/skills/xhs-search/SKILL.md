---
name: xhs-search
description: Search Xiaohongshu (小红书/RED) for travel guides, restaurant reviews, product recommendations, and lifestyle tips.
---

# Xiaohongshu Search Skill

Search 小红书 notes from the command line. Wraps the `xhs` CLI (redbook-cli). Useful for travel guides, restaurant recommendations, product reviews, and real user experiences.

## Prerequisites (one-time setup)

The first time you use this skill, check if the CLI is installed:

```bash
which xhs
```

If not installed:

```bash
pip install redbook-cli
xhs init   # follow the prompts, skip proxy if in China, scan QR to login
```

`xhs init` will guide you through setup. After the first successful login, Chrome remembers the session — you don't need to log in every time.

## Every-session check

Before searching, verify the environment is ready:

```bash
xhs status
```

Three possible states:

| Status | What to do |
|---|---|
| `CDP: Chrome 未启动` | Run `xhs login --cdp` — opens Chrome, scan QR if needed |
| `CDP: Chrome 已启动` but search fails | Session may have expired, run `xhs login --cdp` to refresh |
| `CDP: Chrome 已启动` and search works | Ready to go |

If `xhs` is not installed at all, tell the user and fall back to WebSearch. Don't block on it.

## Search

```bash
xhs search "<关键词>" [--sort <排序>] [--type <类型>] [--time <时间>]
```

Options (all optional):

| Option | Values |
|---|---|
| `--sort` | `综合` (default), `最新`, `最多点赞`, `最多评论`, `最多收藏` |
| `--type` | `不限` (default), `图文`, `视频` |
| `--time` | `不限` (default), `一天内`, `一周内`, `半年内` |

### Search strategy

- **Start broad**, then narrow: first search with general terms, then refine based on what comes back
- **Prefer `--sort 最多点赞`** when looking for quality guides and recommendations — high-engagement notes are usually more reliable
- **Use `--time 半年内`** for time-sensitive info (current conditions, seasonal tips, new openings)
- **Batch multiple searches** — run 2-4 related searches in parallel to cover different angles
- **Read results contextually** — the search output includes titles, authors, like counts, and note types. Use this to judge credibility before referencing a result

### Reading note details

After a search, use short-index to read a specific note:

```bash
xhs read 1      # read the first result from the last search
xhs read 3      # read the third result
```

Note: short-index only works within the same session (results cached in memory). If it fails, re-run the search first.

### JSON output (for programmatic use)

```bash
xhs search "<关键词>" --json-output
```

Returns structured data with feed IDs, tokens, and metadata. Useful when another skill needs to parse results.

## When to use this skill

Good fits:
- Travel: route tips, accommodation reviews, restaurant recommendations, seasonal conditions
- Shopping: product comparisons, unboxing experiences, value-for-money picks
- Lifestyle: local events, hidden gems, DIY/craft inspiration
- Food: dish recommendations, restaurant rankings, regional specialties

Not a good fit:
- Breaking news or factual verification (use WebSearch instead)
- Official data like opening hours or ticket prices (check official sources)
- Anything requiring authoritative sources

## Limitations

- **CDP mode only** — needs Chrome running in the background. If Chrome isn't available, this skill can't work.
- **Session expiry** — login cookies expire after 12+ hours of inactivity. Re-run `xhs login --cdp` to refresh.
- **No official API** — relies on web automation via Chrome DevTools Protocol. Xiaohongshu may change their page structure at any time.
- **Chinese-language ecosystem** — search works best with Chinese keywords. English queries return sparse results.
- **Rate limiting** — don't fire off more than ~5 searches in quick succession. Add 2-3 second pauses between batches.
