---
name: skill-health-checkup
version: 1.0.0
description: "Use when two skills fire for the same request, or a skill never fires at all — scans every installed skill, maps trigger overlaps and blind spots, delivers one health report with fixes. 当多个技能被同一句话触发、或某个技能从不触发时使用——扫描全部已装技能，找出触发重叠与盲区，输出一份带修复建议的体检报告。"
---

# Skill Health Checkup

Your skills fight each other for attention — or never get any. This skill runs a full physical exam on every installed skill and ships one report: who overlaps whom, who is dead weight, who has a broken description.

## Startup

1. Locate the skills directory (try in order: `~/.openclaw/skills/`, `<project>/.claude/skills/`, `~/.claude/skills/`; if none exists, ask the user where skills live)
2. Run the deterministic scan:
   ```bash
   python3 scripts/scan_skills.py --dir <skills-dir> --json
   ```
3. Python unavailable → read each `SKILL.md` frontmatter manually (name + description only)
4. Report language: match the user's language

## The Exam — 5 checks

### 1. Trigger Overlap (the headline check)
For every pair of skills, compare their `description` trigger phrases:
- **RED** — two skills explicitly claim the same scenario (e.g. both say "Use when the user asks to optimize a prompt")
- **YELLOW** — partial overlap: same verbs, different objects ("write a report" vs "write documentation")
- **GREEN** — no conflict
Verdict rule: for every RED pair, name the exact sentence a user could type that would wake both skills. No hypotheticals — the sentence must be plausible.

### 2. Dead Zones
Ask the user for their 5 most frequent task types (or infer from conversation). Check which tasks have **no skill covering them**. A dead zone is not an error — it is expansion space. List top 3 with a one-line skill idea for each.

### 3. Description Fitness
For each installed skill:
- description shorter than 60 chars → trigger accuracy at risk (**RED**)
- no "Use when"-style trigger phrase → the model must guess when to fire (**YELLOW**)
- describes features ("generates reports") instead of scenarios ("use when memory contains stale numbers") → propose a rewrite (**YELLOW**)

### 4. Duplicates
Same core function installed twice — detect by name similarity plus description similarity (≥60% keyword overlap in the trigger phrase). **RED**. Ask before suggesting removal.

### 5. Bloat
Skills that are large (many reference files) AND plausibly never triggered = dead weight loading cost. **YELLOW**, suggest review — never delete without asking.

## Report

Follow `references/report-template.md`. Rules:
- Overlaps: one line per skill pair, with the wake-both sentence quoted
- Fitness: one line per skill with the verdict and the fix
- End with **Top 3 fixes by impact** — the three edits that most improve trigger accuracy

## Fix Protocol

- For RED overlaps: propose a boundary sentence for each skill ("You handle X; the other handles Y"), then edit both descriptions
- For fitness issues: offer a rewritten description, show the exact before/after
- Never modify a skill without showing the user the precise diff first
- Re-run check 1 after any description edit to confirm the conflict is gone
