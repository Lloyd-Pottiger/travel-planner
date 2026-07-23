---
name: xhs-search
description: Search Xiaohongshu (小红书/RED) for travel guides, restaurant reviews, product recommendations, and lifestyle tips.
---

# Xiaohongshu Search Skill

Search 小红书 notes from the command line via [OpenCLI](https://github.com/jackwener/opencli). Useful for travel guides, restaurant recommendations, product reviews, and real user experiences.

## Prerequisites (one-time setup)

Check if OpenCLI is installed:

```bash
which opencli
```

If not installed:

```bash
npm install -g @jackwener/opencli
```

Then verify the Browser Bridge Chrome extension is connected:

```bash
opencli doctor
```

If it shows `[MISSING] Extension: not connected`, follow the setup guide at https://github.com/jackwener/opencli — download the extension from Releases, load it unpacked in `chrome://extensions/`, then re-run `opencli doctor`.

Login to Xiaohongshu:

```bash
opencli xiaohongshu login
```

This opens Xiaohongshu in your existing Chrome — scan QR with the app. Login persists across sessions.

## Every-session check

```bash
opencli doctor
```

If daemon and extension are both OK, you're ready. If not, run `opencli daemon restart` then `opencli xiaohongshu login`.

## Search

```bash
opencli xiaohongshu search "<关键词>" [options]
```

Common options:

| Option | Description |
|---|---|
| `-f json` | JSON output (for programmatic use) |
| `-f csv` | CSV output |
| `-f table` | Table output (default, human-readable) |

### Search strategy

- **Start broad**, then narrow: first search with general terms, then refine based on what comes back
- **Batch multiple searches** — run 2-4 related searches in parallel to cover different angles
- **Read results contextually** — the output includes titles, authors, engagement counts. Use this to judge credibility
- **For deeper context**, read individual notes with `opencli xiaohongshu note <note-id>`

## Other useful commands

```bash
opencli xiaohongshu note <note-id>        # Read full note content
opencli xiaohongshu comments <note-id>    # Read comments on a note
opencli xiaohongshu user <user-id>        # View a user's public notes
opencli xiaohongshu feed                  # Browse home feed
```

## When NOT to use

- Breaking news or factual verification (use WebSearch instead)
- Official data like opening hours or ticket prices (check official sources)

## Limitations

- **Requires Chrome** — the Browser Bridge extension runs in Chrome. If Chrome isn't available, fall back to WebSearch.
- **Session expiry** — Xiaohongshu login may expire. Re-run `opencli xiaohongshu login` to refresh.
- **No official API** — relies on browser automation of your logged-in session. Xiaohongshu may change their page structure over time.
- **Chinese-language ecosystem** — search works best with Chinese keywords. English queries return sparse results.

## Completion

Search is complete when: results have been returned, key findings are contextualized for the caller's needs, and any follow-up searches suggested by the initial results have been run (or explicitly deferred by the user).
