# Field report: Historia-globen, October 2026

Feedback from one consumer project, written at the maintainer's request. It describes
what worked, where work was wasted, and small generic rule changes that would have
prevented it. Nothing here is host policy; adopt, rewrite or reject each suggestion.

## The project and the runtime

- **Project:** [Historia-globen](https://github.com/DasKrokodil/historia-globen), an
  interactive history globe. Static ES modules without a build step, hand-formatted
  JSON data and a strict source policy: every fact must be checked against an opened
  source page, and only the human owner may mark content as human-reviewed.
- **Installed version:** v0.6.0 via `git subtree --squash`. The host policy is in
  `.orchestra/project.md`: bounded passes in PR mode, at most four PRs per pass, at
  most three workers, mandatory conductor review, and fact work on the conductor's model.
- **Runtime:** Claude Code in a cloud container. The conductor spawned workers as
  subagents, each in its own git worktree and port, and the user merged every PR.
  There was no `gh` CLI, and GitHub access went through MCP tools scoped to one
  repository.
- **Volume, 28–30 October:** roughly 80 PRs. About half were data PRs (wars, battles,
  sources and images), and the rest were UI, playback and tooling fixes. The project
  owner made the orchestra permanent after the first pass.

## What worked well

1. **Conductor review before every PR.** In the first pass the review found
   something in half of the deliveries. In the later war-data passes it caught a
   factual slip in roughly every third delivery. Examples: a casualty figure that was
   off by one compared with the source (28 instead of 29), a wording stronger than the
   source ("the largest" instead of "one of the largest"), a claim the source did not
   make ("first independent state in Latin America *and the Caribbean*"), and missing
   uncertainty markings. The cheapest effective check was to spot-check three or four
   numbers against the raw source text (for example Wikipedia's `action=raw`), rather
   than rereading the whole diff.
2. **Same model for fact work.** It was the right call. Workers on the conductor's
   model reliably refused to write from memory and documented where sources
   disagreed, instead of picking one silently.
3. **A shared brief file for a batch of similar issues.** For nine war issues the
   conductor wrote one brief file and gave each worker a short assignment that pointed
   to it. That saved a lot of prompt and kept the deliveries consistent (same fields,
   same tests, same report format).
4. **A merge rehearsal of all open PRs when the queue drained.** It found conflicts
   and cross-PR defects before the user hit them. Examples: two PRs that added the
   same source id, and items that broke a project invariant only once both PRs were
   merged.
5. **Workers report bugs in existing code instead of widening their scope.** Several
   workers found real playback bugs while testing data, for example an infinite
   restart loop. They described the bug and left the code alone. The conductor turned
   each one into an issue and a separate small PR.
6. **Bounded passes plus "area overrides the PR cap".** "Take the whole of version 3"
   ran cleanly in waves of three workers, with no idle polling.

## Where work was wasted, and suggested rule changes

### 1. Parallel data PRs appending to the same file

All workers inserted their records at the same natural spot (after the last related
record in `events.json` and `sources.json`). Every PR after the first conflicted, and
the conductor had to move blocks around by script before the PRs could merge in any
order.

**Suggestion (COORDINATION.md or WORKER.md):** when several workers add records to the
same append-mostly file, the conductor gives each worker a distinct insertion anchor
(for example "insert after record X") and states it in the brief. Changes at distinct
anchors merge cleanly in any order. After this rule was added to the brief, no
further data conflicts occurred.

### 2. Append-only logs always conflict on the last line

Every PR that recorded a decision appended a row to `docs/beslut.md` (the decision
log), and any two such PRs conflicted. The resolution is always "keep both, in merge
order". It cost a round trip with the user each time.

**Suggestion:** say explicitly that a conflict in an append-only log is resolved by
keeping both sides in merge order, and that the conductor may do this without asking.
Optionally recommend that hosts use one file per decision, or a marker the PR fills
in, if conflicts become frequent.

### 3. Workers that finish with background processes still running

Workers that started `python3 -m http.server` or a long Playwright run in the
background and then reported produced repeated "waiting on its own background work"
notifications. The conductor had to stop them explicitly. One worker kept a test
queue lock for minutes after its report.

**Suggestion (WORKER.md):** before returning, stop every background process you
started (servers, watchers, test runs) and release shared locks. The conductor stops a
worker that has reported but still has background work, instead of waiting for it.

### 4. Killing processes by pattern

`pkill -f "http.server"` in a worker's shell also matched the shell's own command line
and killed the test script (exit 144). The same pattern could kill another worker's
server.

**Suggestion (WORKER.md, isolation):** start servers as `cmd & P=$!` and stop them with
`kill $P`. Never kill processes by name pattern, since sibling workers run identical
commands.

### 5. Commit early, because sessions restart

A session restart killed a worker that had done all its research but not committed.
The work could be rebuilt from notes in the scratchpad, but only by chance.

**Suggestion (WORKER.md):** commit (and push, if the host allows) at each meaningful
milestone, not only at the end. A worker's progress should survive the loss of the
worker.

### 6. Personal data sent to external services

A worker once put the owner's e-mail address in the User-Agent of requests to an
external API, because the API asks for contact details. The project then added a
rule: never send the owner's name, e-mail or other personal data to external services.
Use the project name and repository URL instead.

**Suggestion (WORKER.md or ORCHESTRA.md, generic):** never send personal data from the
user's environment (name, e-mail address, account identifiers) to external services
in headers, URLs or request bodies. If a service needs a contact, use the project's
public identity.

### 7. Shared expensive verification: keep the queue short

One `flock` lock file for Playwright worked well and prevented flaky parallel browser
runs. When one worker ran a ten-minute end-to-end playback test, though, the other two
waited on the lock and the conductor's own check timed out.

**Suggestion:** next to the shared-queue rule, recommend a time budget per run (for
example under five minutes) and splitting long scenarios into several runs, so the
queue stays fair.

### 8. Blocked network hosts: report the final host

The environment's allowlist was updated after the conductor reported blocked domains,
but some sources stayed blocked. The bare domains were allowed, but `un.org` redirects
to `www.un.org` and `news.un.org`, which were not. That took one more round trip.

**Suggestion (host-neutral):** when reporting blocked sources, test the final URL after
redirects and report the exact hostnames, or a wildcard such as `*.un.org`.

### 9. Ordering dependencies between PRs

A few PRs could only be merged in a certain order: a fix that touched the same lines
as a feature, or two PRs that introduced the same source. Where the conductor wrote
the required order and the reason in each PR body, merging went smoothly. Where it
did not, the user hit a conflict.

**Suggestion (ISSUE-WORKFLOW.md or COORDINATION.md):** when PRs from the same pass
depend on each other, state the merge order and the reason in each affected PR body.
After each merge, the conductor refreshes the remaining branches.

### 10. A user-question channel that matches how the owner works

The workflow says to post user-only questions on the issue with the `user question`
label. That worked. The owner, however, often answered in chat ("kör som
rekommenderat", that is "go with the recommendation"), so the conductor had to mirror
the answer to the issue for traceability.

**Suggestion:** mention that an answer given in chat is recorded on the issue by the
conductor, with the date, so the decision remains findable.

## Smaller observations

- **Deferred tool loading:** the MCP tools for GitHub had to be loaded before every new
  kind of call (issues, PRs, labels). That is a runtime detail, but a short note in
  AI.md, "load the issue and PR tools at startup", would save a few turns.
- **Restricted environments:** a session limited to one repository cannot fork or open
  a PR in an upstream repository. This report had to be prepared locally and pushed
  through a fork that the owner created. INSTALL.md or CONTRIBUTING.md could mention
  this path for consumers who want to contribute back.
- **Labels for user tests:** the project used an extra label for findings from
  automated user tests (`användartest`). The owner approves a finding by removing the
  label, and the orchestra then builds it. It was a simple and effective way to keep
  the owner in control of what gets built. It might be worth describing as an optional
  intake pattern.

## Summary

The method held up under a large volume of small, verifiable changes. The mandatory
review paid for itself many times over. Most of the waste came from parallel workers
touching the same files, from processes outliving workers, and from round trips that
a single sentence in the right place would have prevented. All of the suggestions
above are small additions to existing sections, and none of them changes the
contract of the workflow.
