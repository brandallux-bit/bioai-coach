# bioai-coach

The coach's rules for Claude Code. Open Claude Code in this repository and talk to your coach: it logs and answers through the BioAI platform's tools, signed in as you.

**It holds no data and no platform code.** Your chart lives on the platform (bioaicoach.com). This repository holds only:

- `CLAUDE.md`: the coach's rules, loaded in every session.
- `.claude/skills/today/`: how to log a day, save what you tell it, and answer questions about it.
- `.claude/skills/train-today/`: what to train today, from what you actually did lately.
- `.claude/skills/program-design/`: the next block, whether you're ready to go up, and how your lifts are doing.
- `.claude/skills/weekly-review/`: how your last finished week went: the facts, one verdict, one thing for next week.
- `.claude/skills/nutrition-targets/`: should you eat more or less: maintenance from what you ate and what your weight did, your floor, and the calories and protein a rate comes to.
- `.claude/skills/intake/`: setting up a new person: what you want, how you live, what is safe for you, and your first plan, saved as you say it and adopted only on your yes.
- `.claude/skills/plan-change/`: changing a day or the plan: draft, stage, review, show, and save only on your yes.
- `.claude/agents/`: read-only reviewers the coach runs on a plan change: the five standing ones, `nutrition`, `strength`, `recovery`, `adherence` and `red-team`, and one `specialist` that takes on any brief saved in your chart. They read your chart and save nothing.
- `.claude/settings.json`: the platform's tools pre-allowed, file and shell tools denied, and the start-of-session hook.
- `.claude/hooks/session-key.mjs`: prints a fresh key prefix at the start of each session, so a resent save is never saved twice.

## Before the first session

You need to have been invited. Start at `https://www.bioaicoach.com/start`: sign in, press Start, and follow the steps it shows you.

1. Connect the coach once at claude.ai: Settings, Connectors, Add custom connector, named `BioAI`, with the URL `https://www.bioaicoach.com/api/mcp`. Sign in by the emailed link if asked, and press Allow.
2. Get this folder: download the zip from the Start page, or use the phone steps below.

## Opening it

- **Desktop:** open Claude Code in the folder you downloaded.
- **Phone:** sign in with GitHub at claude.ai/code and start a session on `brandallux-bit/bioai-coach`. If the session can't reach the platform, add `bioaicoach.com` to that environment's allowed domains.

Then say hello. Its first step is always to read your chart summary. After that, tell it a meal, a weight, a workout, or ask "what's left today?".

## Changing it

Don't edit this repository directly. It's published from the platform repository's `workspace/` folder, where it's tested: make the change there, run `pnpm workspace:publish <this folder>` from the platform repository, then commit and push here.
