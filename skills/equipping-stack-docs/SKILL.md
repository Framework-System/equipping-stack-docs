---
name: equipping-stack-docs
description: Use when a design/spec is approved OR when starting substantial implementation work (a feature, a multi-file change) in an existing codebase whose stack has no local doc skills - discovers the stack (from the design, manifests, or code), generates project-local documentation skills with version-pinned docs via Context7
---

# Equipping Stack Docs

## Overview

Model training knowledge about frameworks and libraries goes stale fast — and it fails hardest in legacy codebases, where the model writes today's idioms against yesterday's APIs. Once the stack is known — named in an approved design, read from the project's manifests, or inferred from the code itself — generate one documentation skill per core technology, grounded in docs for the version the project actually uses, so implementation plans and every implementation subagent works from the project's real APIs instead of training memory.

**Announce at start:** "I'm using the equipping-stack-docs skill to generate up-to-date documentation skills for the stack."

<HARD-RULE>
You are an orchestrator. NEVER fetch documentation yourself — dedicated subagents do that. Raw documentation must never enter your context. And a Context7 failure of any kind NEVER blocks Step 6 (the hand-off).
</HARD-RULE>

## When to trigger autonomously (outside a design flow)

Trigger only for substantial implementation work — a new feature, a multi-file change, a migration — in an existing codebase whose stack has no `.claude/skills/<tech>-<major>-docs/` skills yet. Do NOT trigger for trivial work: a one-line fix, a config tweak, a typo. An over-eager skill is a skill your human partner learns to ignore. When in doubt whether the work is substantial, it is not — proceed with the task without interrupting.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Discover the stack** (design → manifests → code)
2. **Confirm the list** with your human partner
3. **Check for existing doc skills** in the project
4. **Dispatch one subagent per technology**
5. **Verify generated skills and commit**
6. **Hand off** (writing-plans when a design flow is active, or report back)

## Step 1: Discover the stack

Consult three sources in order, accumulating results:

1. **Approved design** (when one exists — the most recent design/spec file, e.g. under `docs/superpowers/specs/` when the superpowers plugin manages the flow, or wherever the project keeps specs): the technologies it names explicitly.
2. **Project manifests**: `package.json`, `pom.xml`, `build.gradle`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`, `Gemfile`, `*.csproj`. Manifests are the source of truth for **versions** — including for technologies the design named.
3. **Code inference** (only where no manifest covers the touched area): imports/usings in the affected files, jars in `lib/`, `<script src>` tags, characteristic config files (`web.xml`, `applicationContext.xml`, `struts.xml`). Mark inferred versions as approximate (`~2.x`). If nothing recognizable turns up, do not invent — ask in Step 2.

**Relevance filter (touched area):** keep only the technologies the files of this feature/task actually use — the web framework, the ORM, the module's central library. Typically 2–5 entries. Skip trivial utilities whose APIs are stable across versions (lodash, dotenv, uuid and the like). With no design (autonomous mode), scope the touched area by the requested task instead.

Versions come from the project. "latest" is only for a new project with no manifest.

## Step 2: Confirm with your human partner

Present the list as a table — technology, version (the project's real value; `~` marks approximate), source (`design` / `manifest` / `inferred from code`), one-line reason it needs current docs — and ask them to confirm, adjust, remove, or add entries. If code inference found nothing recognizable, ask for the stack here instead of guessing. Do NOT dispatch any subagent before they answer.

If they remove every entry, skip directly to Step 6.

## Step 3: Check for existing doc skills

For each confirmed technology, check whether `.claude/skills/<tech>-<major>-docs/` already exists in the project (from a previous feature in the same project). If it does, ask your human partner: keep it (skip regeneration) or regenerate it. Anonymous Context7 access has low rate limits — don't burn them regenerating skills that are still fresh.

## Step 4: Dispatch one subagent per technology

Dispatch all subagents in parallel, one per technology, each with the complete prompt template below (fill every `{placeholder}`). If any subagent reports persistent rate limiting, dispatch the remaining technologies sequentially instead, one at a time.

### Subagent prompt template

```
You are generating a project-local documentation skill for one technology,
using the Context7 REST API anonymously. Your final message must be ONLY a
status report (format at the end) — never paste raw documentation into it.

Technology: {tech}
Project version: {version, "~X" if approximate, or "latest"}
Project summary: {one line about what is being built}
Output file: {absolute project path}/.claude/skills/{tech-slug}-{major}-docs/SKILL.md
Topics to prioritize: {3-5 topics taken from the design or the requested task}

Steps:

1. Resolve the library ID (NO Authorization header — always anonymous):
   curl -s "https://context7.com/api/v2/libs/search?libraryName={tech}&query={primary use}"
   The response is JSON: {"results":[{"id","title","trustScore","benchmarkScore","versions",...}]}.
   Pick the canonical entry: official org repo (e.g. /vercel/next.js), trustScore >= 9,
   highest benchmarkScore among candidates. If no result has trustScore >= 9 or nothing
   plausibly matches, STOP and report status "not-found".

   Version ladder (project version vs the entry's "versions" array):
   a. Exact project version listed -> use the versioned ID: {id}/v{version}.
   b. Exact absent but others listed -> use the listed version closest to the project's,
      preferring the SAME MAJOR; with no same-major entry, the nearest available.
      This is a VERSION DELTA — note both versions for steps 2-4.
   c. Nothing versioned available -> use the unversioned ID. This is also a VERSION
      DELTA (resolved = latest docs); treat it with the strictest evidence rule below.

2. Fetch each priority topic (one call per topic, sleep 3 between calls):
   curl -s --max-time 30 "https://context7.com/api/v2/context?libraryId={id}&query={topic}"
   The response is plain text: snippet blocks with a "### Title" heading, a "Source:" URL,
   prose, and code, separated by dashed lines.
   When there is a VERSION DELTA, fetch FIRST one extra topic:
   "migration guide breaking changes since {project version}" — it is the evidence base
   for delta annotations.
   On HTTP 429: sleep 15, retry once. If it still fails, stop fetching and generate the
   query-only variant (step 4b) with reason "Context7 unreachable".

3. Distill — do NOT paste raw docs. For each topic extract: current idiomatic patterns
   (with short code samples), exact API signatures, and anything that changed in recent
   major versions. Target ~150 lines total for the snapshot.

   With a VERSION DELTA, distill AGAINST the project's version:
   - Annotate a pattern with "[not in {project major}]" ONLY when the fetched docs
     evidence it — "since v" notes, migration guides, changelog entries.
   - No evidence -> OMIT the annotation. NEVER fill the gap from training memory:
     a confidently wrong delta note is worse than a generic doc, because it carries
     the authority of a generated skill.
   - If the fetched docs describe practically a different product from the project's
     version (e.g. AngularJS 1.x vs modern Angular), STOP: report status "version-gap"
     and write the query-only variant (step 4b) with reason "version gap".

4. Write the skill to the output file, creating directories as needed:

   ---
   name: {tech-slug}-{major}-docs
   description: Use when writing or reviewing {tech} {project major} code in this project - current conventions, APIs and patterns for v{project version}
   ---

   # {Tech} {project version} — Current Docs (generated {YYYY-MM-DD} via Context7)

   ## Essential conventions (snapshot)
   {distilled content, organized by topic}

   ## Before implementing anything outside this snapshot
   Fetch live docs (anonymous, low rate limit — use sparingly):
   curl -s "https://context7.com/api/v2/context?libraryId={resolved id}&query=<specific topic>"

   With a VERSION DELTA, add this block right under the H1:

   > VERSION DELTA: project uses {project version}; docs resolved from {resolved version}
   > (closest available). Patterns below are annotated — anything marked
   > [not in {project major}] must not be used.

   The description always names the PROJECT's version, so the skill auto-triggers for
   the code the project actually contains.

4b. Query-only variant (rate limited, offline, or version gap): write the same file but
   with the header line "> NOTE: generated without snapshot ({reason}). Use the live
   query below and filter results against your project version ({project version})."
   and no snapshot section.

5. Report back EXACTLY this format and nothing else:
   status: generated | query-only | not-found | version-gap
   library-id: {resolved id or "-"}
   topics: {comma-separated list or "-"}
```

## Step 5: Verify generated skills and commit

For each technology, verify the output file: it exists, the frontmatter has `name` and `description`, and the body is non-empty. If a file is missing or empty, mark that technology as failed — do not retry more than once.

Commit the generated skills to the project's git repository (they are project artifacts, versioned like any other).

Report one line per technology to your human partner: `generated` / `generated with version delta` / `kept existing` / `query-only (rate limited)` / `version-gap (docs too far from project version)` / `not found on Context7` / `failed`.

## Step 6: Hand off

- **A design flow is active** (an approved design/spec triggered this skill): if a writing-plans skill is available (e.g. `superpowers:writing-plans`), invoke it and tell it which documentation skills now exist (exact skill names) so the plan references them in the tasks that touch each technology. With no planning skill available, report the generated skills and let your human partner drive the next step.
- **Invoked autonomously (no design):** report the generated skills to your human partner and return control to the work that triggered you. Do not force a planning flow.

## Failure handling

| Failure | Behavior |
|---|---|
| Technology not found on Context7 | Report it, generate nothing for it, continue |
| Persistent 429 after backoff | Subagent writes the query-only variant, orchestrator reports it |
| No network | Same as persistent 429 |
| Code inference finds nothing recognizable | Ask your human partner for the stack in Step 2 instead of inventing |
| Version gap (docs too far from the project's version to annotate reliably) | Subagent writes the query-only variant with the version warning, orchestrator reports it |
| Subagent output file missing/empty | Retry that subagent once, then report `failed` and continue |

Whatever happens with Context7, Step 6 always runs.
