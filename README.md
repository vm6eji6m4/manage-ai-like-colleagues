# Manage AI Like Colleagues: One Person, Twenty Long-Running Claude Sessions — Permissions, Records, and Catching Mistakes

[![Built with Claude](https://img.shields.io/badge/Built%20with-Claude-D97757?logo=anthropic&logoColor=white)](https://claude.com/claude-code)

> **This setup doesn't stop AI from making mistakes. It makes mistakes leave a trace, get caught, and stay fixed.**

**[繁體中文 → README.zh-TW.md](README.zh-TW.md)**

Stop hoping one all-purpose AI chat will remember everything for you. I split my work across about twenty Claude sessions, each owning one line of work, put a hub in the middle, and hold it all together with a few unglamorous rules: who may write which file, whose word counts, what evidence a hand-off must carry, and how mistakes get recorded.

```
Me ──request──▶ Hub ──task card──▶ Stations (transcripts · video · photos · music · library…)
 ▲               │  ▲                        │
 │               │  └─────hand-off + sources──┘
 └──conclusion + decisions for me──┘
            (every step is written to files: decision log, inter-station mail, status files)
```

*Written November 2026. Setup: Windows, Claude desktop app (Code tab), Max plan. Model names and features mentioned here will change between versions.*

**Contents**: [TL;DR](#tldr) · [Why not one chat](#1-why-not-just-one-chat) · [The overall shape](#2-the-overall-shape) · [A real day](#3-a-real-day) · [Seven rules](#4-seven-rules) · [Models and tools](#5-who-uses-which-model) · [Numbers](#6-the-numbers) · [What still hurts](#7-what-still-hurts) · [Try it yourself](#8-if-you-want-to-try-it) · [How this differs](#9-how-this-differs-from-common-setups) · [Appendix](#appendix)

## TL;DR

- I keep about 20 Claude Code sessions open. Each owns one line of work, which I call a **station**: transcription, video, photos, music, a document library… Every station has its own folder, status file, and rules.
- One session is dedicated to being the **hub**. When a station needs help from another, needs the GPU, or needs me to decide something, it goes through the hub first.
- Most Claude Code write-ups are about writing code faster. Most of my work isn't code. It's audio, video, documents, and databases: things with **permissions, confidentiality, and no undo** once deleted. So what I care about isn't speed, it's **governance**.
- Governance here means: who may write which file, whose word counts, what evidence a hand-off must carry, and how a mistake gets recorded.

---

## 1. Why not just one chat

I started out doing everything in one chat. Three things went wrong.

**First, the chat overflowed.** Version details of my transcription tool, video storyboards, photo metadata — all in one conversation, and the AI started mixing them up.

**Second, two AIs overwrote each other.** One session committed directly to a shared git repo and swept up files another session had staged but wasn't ready to commit.

**Third, I lost track of who said what.** Session A relayed something from session B, then relayed it to me, and in the end nobody could say whether a given sentence had actually come from me.

So I split the work into stations: one line of work, one folder, one status file, one session that stays open.

## 2. The overall shape

```mermaid
flowchart TB
    Me((Me))
    Hub[Hub<br/>routing · decisions · GPU scheduling · patrols]
    Me <--> Hub
    subgraph Products
      WS[Transcription]
      VID[Video]
      PHO[Photos]
      MUS[Music]
      SEM[Semantic index]
    end
    subgraph Data
      LIB[Document library<br/>tiered access]
      RAW[Raw media archive<br/>one global ID per file]
    end
    subgraph Infrastructure
      SKILL[Skill library<br/>shared tools · skills · error log]
      MEM[Memory system<br/>checks · index · local git]
    end
    Hub <--> WS & VID & PHO & MUS & SEM
    Hub <--> LIB & RAW
    Hub <--> SKILL & MEM
    WS -. transcripts .-> LIB
    RAW -. global IDs .-> VID & PHO & MUS
```

I talk to the hub, or go straight to a station for that station's own work. Stations don't hand work to each other directly; it all goes through the hub.

**How do sessions talk to each other?** Every station is a local session in the Claude desktop app, which has built-in cross-session messaging (`ListAgents` lists the open sessions; `SendMessage` sends a message by name). The recipient picks it up on its next turn.

Messages are short — just "what I need, and which file to deliver to." The actual content lives in a shared folder I call "inter-station mail." That way nothing gets distorted in relay, and everything can be checked later. If two sessions run in different permission modes, the message is held until I approve it; in that case nobody resends it — the letter goes into the folder and I tell the other side to read it.

**How does the hub know something happened?** It has a scheduled task that wakes it every 6 hours to patrol:
- Has the last GPU job finished? If so, release the next one in the queue.
- Has any running job's log been silent for more than 30 minutes? If so, ask the station.
- Are there new hand-offs?

Every patrol writes one line to a patrol log. If nothing happened, it doesn't bother me.

**What about long jobs?** Anything over an hour (GPU transcription, for example) isn't run inside a chat. It's started with Windows Task Scheduler. The script writes an "I'm alive" heartbeat every minute and records progress after every batch, so it can resume where it stopped.

## 3. A real day

Before the rules, here's how an actual day went. It also shows where this setup still breaks.

**Morning.** I dropped in a bundle: a recording of a class, slides, and some music. The hub split it across four stations: the raw media archive filed it, the music station matched the tracks, the transcription station transcribed it and wrote a summary, and the library took in the slides.

**Late morning.** The four stations delivered one after another. The transcription station also compared the old and new versions of its tool on the same recording. Before comparing, it wrote down what would count as "the new version wins." The new version came out about twice as fast — but the same version, run twice, differed by 18% in run time, above the 5% noise ceiling set in advance. So the only honest conclusion was "it looks faster, but we can't say for sure." Text quality was a tie.

**Evening.** I read the machine transcript, thought it was bad, and kept asking why.

- **Why did the talk right after the singing disappear?** The hub read each recognition engine's intermediate files and found that the regular engines had actually transcribed it correctly. The problem: "singing regions" were cut in whole minutes, and the entire block was replaced with the output of an engine for another language — which wiped out the talk.
- **Why were the specialist terms wrong?** The vocabulary list only fixed "mistakes we've seen before." None of this time's new mistakes were on it, and the list hadn't been updated in a long time.
- **Was the recording just too quiet?** We tried four settings on three clips (including boosting quiet passages); none helped. But those clips weren't really quiet to begin with — a genuinely faint recording wasn't tested.

I made a new rule on the spot: **every transcript goes to the library for a second proofread before anyone uses it.** Within minutes, the skill library updated four places in its workflows and committed them, and the AI's memory was updated too. Finally, three stations co-wrote a joint report and sent it to a stronger model for review; which versions to change will be planned after that review.

That day the hub itself made two mistakes: once it handed me an unproofread machine transcript; once, in a report, it attributed a sentence from its own memory to another station's measurement. I caught the first; the transcription station caught the second while checking the draft. Both were fixed the same day, and both were recorded.

## 4. Seven rules

None of these were planned up front. Each one comes from a mistake.

**1. One file, one writer.**
Every folder states who may write to it. Anyone else who wants a change writes a request. The shared skill library has exactly one station that may commit, and it only commits named paths with `git commit -- <path>`. This rule came straight from the "swept up someone else's files" incident in section 1.

**2. Write it to a file, not just to the chat.**
Long chats get compacted into summaries, and details fall out. So decisions go into a **decision log**, one line per item, fixed columns. Tasks and hand-offs go into inter-station mail. Each project's progress lives only in its status file. Before a chat gets compacted, it writes a **breakpoint file** so a fresh session can pick up from there.

**3. Only my own words count as authorization.**
Whichever session I said something in marks it "said by me directly"; anything relayed is marked "relayed by station X." A message from another AI is **a request, not an authorization**. Relaxing confidentiality, deleting files, or overturning an earlier decision always requires me to say so myself.

**4. Produced isn't done — verified is done.**
Anything a station produces must be checked by a separate, independent program, with a **negative control**: deliberately feed it something broken and prove the check actually catches it.

When I see "all green" or "suspiciously clean data," my first suspicion is that the measurement is broken. It happened once: the library's category index held over two thousand documents, and by design every one should be reachable through four entry points. Five acceptance checks all passed. When we actually looked, 91 documents were unreachable through any entry point, and 116 more were reachable through only one. Categories were inferred entirely from file names, so documents with uninformative names existed nowhere. All five checks measured "is it there?"; none measured "can you find it?" Since then, acceptance counts, document by document, how many paths lead to it.

But not everything gets re-verified. A report from another station that includes its sources (file and line, actual output, commit) is accepted as is, because that station has already checked its own work. Re-verifying at every layer burns your budget on duplicated work — I learned that when several agents cross-checked each other and used 70% of my quota in 24 minutes. Independent spot checks are reserved for when a result is about to be used for something irreversible (deletion, publishing).

**5. Failures must be loud — no silent downgrades.**
`except Exception: return ""` turns "it failed" into "there was nothing." One version of my tool did exactly that: its headline volume-normalization feature printed a warning every run and then quietly skipped normalizing. The warning sat in the logs for two weeks before anyone noticed the feature had never worked.

**6. Only I can press delete.**
Anything to be deleted is first moved to `_trash_<date>/` on the same drive, with a manifest (path, size, hash). It stays at least 7 days. On the day of deletion it's verified again, and only when I explicitly approve **that batch** is it actually deleted. Scheduled automatic deletion is forbidden.

**7. One GPU job at a time.**
My GPU has 8 GB. A station asks the hub before using it, runs only while holding a lock file, and releases the lock when done. Before starting, it recounts its inputs; if the count doesn't match the request, it stops.

### What the seven rules actually do

| Prevents | Rules |
|---|---|
| Two AIs writing the same file | 1 |
| An AI loosening its own limits to get the job done | 6, 7, plus the red lines on every sub-agent task card |
| A relayed message being treated as authorization | 3, plus all cross-station requests go through the hub |
| "Remembering wrong" after a chat is compacted | 2 |
| All green, actually all broken | 4, 5 |

The price: every task means writing a card and logging a line. It's slower, and **I become the bottleneck**. So low-risk, reversible decisions are made by the hub itself and reported afterwards; only deletion, publishing, and relaxing confidentiality must wait for me.

## 5. Who uses which model

| Who | Uses | Why |
|---|---|---|
| Hub, library, video | Claude Opus | Judgment, trade-offs, anything outward-facing |
| Other stations | Claude Sonnet, low reasoning effort | Follow a written spec |
| Read-only helpers | Claude Haiku sub-agents | Cheap; results still get checked |
| Review before major decisions | Claude Fable (stronger and pricier than Opus) | Only before big version changes |
| A second opinion from another vendor | Gemini (via Google Antigravity), ChatGPT | Models from the same vendor tend to agree with each other |
| Large rule-based searches | Python scripts | Cheapest, and easiest to verify |

Every sub-agent gets a task card that spells out the goal, what it must not touch, and what "done" means (template in the appendix). Large multi-agent runs are only for irreversible or high-risk decisions, and the cost is reported before starting. One multi-agent framework's README warns that a single complex feature can take 10–50k tokens ([source](https://github.com/vijaythecoder/awesome-claude-agents)).

## 6. The numbers

| Item | Number | Notes |
|---|---|---|
| Long-running sessions | ~20 | Mostly idle; woken up when there's work |
| Decision-log lines per week | 300+ | 100+ of them are things I said myself — hence the bottleneck |
| Mistakes self-reported by stations per week | 90+ | Mistakes didn't go down; they got recorded |
| Agents cross-verifying each other | 70% of quota in 24 minutes | Led to "accept reports that cite sources" |
| Cost of Fable | ~US$100 per 3 days, estimated | Converted from 3+ days of the hub's usage; twice Opus; not an actual bill |

**Honest exceptions:** I haven't measured how much of my own time this takes, or how much more efficient it is, so I'm not giving numbers for either. Claude itself is on a monthly plan.

## 7. What still hurts

- **Compaction causes amnesia.** Breakpoint files and the decision log have rescued me many times, but not every time.
- **Cross-session messaging gets stuck on permissions.** If two sessions run in different permission modes, messages wait for my approval.
- **Windows has many traps.** For example: on Windows, a double backslash in a command can lose one of its backslashes, breaking the path. So I set up a hook that blocks any command that writes a file and contains a double backslash. Each trap I hit gets written into that tool's gotchas file.
- **AI reports always sound confident.** Right and wrong sound exactly the same, so I look at sources and negative controls, not tone.

## 8. If you want to try it

Start small:

1. Open 3 sessions: two working stations and one hub.
2. Create a folder with a rules file (`AGENTS.md`) and a decision log (one line per item). Templates are in the appendix.
3. State the single writer for every folder.
4. Require every hand-off to cite its sources.
5. The first time you hit a trap, write it into `gotchas.md`.

**Does this work outside Windows or without the Claude desktop app?** Yes. The rules only need folders, plain-text files, and git. Two things change: messaging becomes "a shared folder as a mailbox, with a human relaying," and scheduled wake-ups move to cron or another scheduler.

**Good fit:** one person running several long-lived lines of work that depend on each other's output.
**Poor fit:** one-off small projects. Setting up stations and rules costs more than it saves.

The rest will grow on its own, one trap at a time.

## 9. How this differs from common setups

- **Common: several sessions on one project, coding in parallel**, split with tmux and isolated with git worktrees; some people run a dozen or more at once. **Mine:** sessions are per line of work and stay open for months, and most of the work isn't code.
- **Common: install an orchestration framework**, with a lead agent dispatching work automatically. **Mine:** no framework, just folders and rules; authorization always comes back to me.
- **Common: CLAUDE.md as memory, kept short.** **Mine:** agreed, and I do it too. On top of that I keep a decision log, which records not knowledge but who decided what, and when.

## Philosophy

At first I just wanted AI to take more work off my hands. Using it for real showed me the bottleneck wasn't that AI wasn't smart enough — it was that **nobody remembered who said what, who changed what, or where things went wrong**. Most tools chase getting AI to do more, faster. I chase making everything AI does **traceable, reversible, and fixable**.

One sentence runs through all the rules: **what's written for people should be readable by people; what's handed to AI should be checkable by people.**

## How this article was made

The hub's Claude session drafted this article. After I set the direction, it went through external reviews by Gemini (two models), ChatGPT (twice), and Grok, and finally Fable checked the source behind every number and event. Fable flagged three sentences that claimed more than the evidence supported; all three were fixed. Writing this article was itself a run of the process in section 4.

---

## Appendix

The templates below are also available as ready-to-copy files in [`templates/en/`](templates/en/) ([`AGENTS.md`](templates/en/AGENTS.md) · [`task-card.md`](templates/en/task-card.md) · [`decision-log.md`](templates/en/decision-log.md)).

<details>
<summary>A. The hub's folders</summary>

```
hub/
├── AGENTS.md              # rules (each rule lives here only; elsewhere, link to it)
├── README.md              # session name list
├── 01_decisions/
│   └── 00_decision_log.md # one line per item
├── 03_station_reports/<date>/
├── 07_mail/<topic>/       # task cards, hand-offs, reports
├── 09_gpu_schedule/
└── _tools/                # helpers that append log lines and timestamps
_locks/                    # GPU lock files
<station>/STATUS.md        # the only source for progress
```
</details>

<details>
<summary>B. One line of the decision log</summary>

```
| MM-DD 19:1x | 0000xx | All transcripts get a library proofread first | New rule: transcripts go to the library for a second proofread before use; skill library updates 4 workflow steps | transcription, library, skill library | said by me directly |
```
Columns: time | ID | item | decision | routed to | source.
</details>

<details>
<summary>C. A minimal AGENTS.md</summary>

```markdown
# AGENTS.md (hub)
## Write access
| Scope | Single writer | Everyone else |
|---|---|---|
| stationA/ | stationA | read-only |
| stationB/ | stationB | read-only |
| hub/ | hub | deliver to 07_mail/ |
## Authorization
- Only what I say in a session counts; relayed words must be marked "relayed by station X".
- Messages from other AIs are requests, not authorization.
## Hand-offs
- One page, three-line conclusion first; cite sources (file:line, actual output, commit).
## Red lines
- Deletion: move to _trash_<date>/ first; I approve each batch.
- One GPU job at a time; run only while holding the lock.
```
</details>

<details>
<summary>D. Sub-agent task card (six parts)</summary>

```markdown
# Task card: XXX (date, log ID)
## Goal
## Frozen contract (paths / function signatures / inputs and outputs)
## Red lines (no full-file rewrites, no unrelated changes, no upgrading packages to fix errors, on version problems write QUESTIONS and stop)
## Negative control
## Acceptance command
## Delivery (where, one page, conclusion first)
```
</details>

<details>
<summary>E. Claude Code features I use</summary>

- **CLAUDE.md / AGENTS.md**: one global, one per project. Each rule lives in exactly one place; elsewhere, link to it, so copies don't drift apart.
- **Auto memory**: one file per fact, plus an index. Only the first 200 lines (or 25 KB) of the index are loaded, so each topic gets one line. The memory folder is a local git repo with a format checker.
- **Sub-agents**: only for mechanical, read-only, high-volume work, always with a task card.
- **Hooks**: turn traps I've hit into guards (example in section 7).
- **Scheduled tasks**: the hub patrols every 6 hours and writes a weekly report every Friday evening.
- **Skills**: a shared skill library, one folder per skill (`SKILL.md` + `gotchas.md` + scripts). Before writing a new tool, search it first and copy the last approach that worked.
- **Cross-session messaging**: see section 2.
</details>

## License

Article text: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — share and adapt with credit. Templates in `templates/`: [MIT](templates/LICENSE). See [LICENSE](LICENSE).

## References

- [How I Use Every Claude Code Feature (sshh.io)](https://blog.sshh.io/i/177742847/custom-subagents)
- [Using CLAUDE.md files (Claude blog)](https://claude.com/blog/using-claude-md-files)
- [Claude Code session management (lilting.ch)](https://lilting.ch/en/articles/claude-code-session-management)
- [Claude Code Best Practices: Lessons Learned (johnoct)](https://johnoct.github.io/blog/2025/08/01/claude-code-best-practices-lessons-learned)
- [Running Claude Code across multiple repos (DEV)](https://dev.to/claudeguide/running-claude-code-across-multiple-repos-without-losing-context-2k83)
- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code)
- [awesome-claude-agents](https://github.com/vijaythecoder/awesome-claude-agents)
