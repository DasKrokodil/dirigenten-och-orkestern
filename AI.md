# Repository AI policy

This is the host policy for maintaining `solwation/dirigenten-och-orkestern` itself.
Consumers use their own host policy. Respond to the maintainer in Swedish; retain
the established language of the files being edited.

## Follow the orchestra

Read this file, [ORCHESTRA.md](ORCHESTRA.md) and
[ISSUE-WORKFLOW.md](ISSUE-WORKFLOW.md) before making changes. Apply the orchestra
to this repository's own maintenance, including a short delivery plan, deliberate
resource use, scoped verification and one conductor review.

The agent receiving the request acts as conductor and integration owner. It may
implement directly and review its own diff for small cohesive changes. Use an
optional worker only when independent work justifies the handoff, with a maximum
of two agents including the conductor. Use [WORKER.md](WORKER.md) for assignments.
Respect platform limits on delegation and model selection; disclose unavailable
controls once rather than pretending they were used.

Each maintainer request starts a single scoped task. Queue polling and idle watching
are disabled unless explicitly requested. For an authorized queue session, also read
[COORDINATION.md](COORDINATION.md) and establish its scope and timer settings.

## Track actual work in GitHub

Use the current authoritative Issueflow guide as resolved by ISSUE-WORKFLOW.md;
record its revision in the issue or delivery note once per session.

- Reuse a matching issue or create one before implementation, with the expected
  result, acceptance criteria, short plan and integration owner.
- Set exactly one canonical priority label: `priority: high`, `priority: medium`
  or `priority: low`. Default to medium and briefly explain the priority.
- Apply `in-progress` only while actively working. Keep ownership separate from
  work status; never remove another active owner's labels.
- Remove all recognized active-work labels when pausing, handing off or finishing.
- Record the delivered revision, checks, limitations and remaining acceptance in
  the issue before closing it. Close only after acceptance and verification pass
  and the requested change has reached `origin/main`.

## Commit and push requested changes

The maintainer has authorized agents to commit and push completed requested changes
directly to `main` in this repository. Treat implementation, verification, commit
and push as one delivery; do not stop with local-only edits or ask again for routine
publishing approval. A later explicit instruction can change this scope.

Fetch before integration and preserve concurrent work. If upstream has advanced,
integrate it with merge ancestry intact (for example,
`git rebase --rebase-merges origin/main` for unpublished local commits), inspect the
combined changes and repeat affected checks. Push normally; never force-push or
bypass branch protection. If direct push is blocked, preserve the work and report
the actual blocker.

For documentation changes, check the diff, relative links and consistency with the
shared rules. Exercise distribution changes in a disposable repository as required
by the README. Do not create a release or move tags unless requested.

Verify that the delivered commit is on remote `main`. Report the outcome, GitHub
links, revision, checks and any unmet acceptance accurately.
