### Triage

**You own the verdict, not the fix.** For "triage issue N", "is this real", "what do we do with this one", and reports filed as GitHub issues. The deliverable is a classification, a severity you can defend, and a disposition. Stop before the fix.

A triage that restates the issue body has done nothing. The value is the three things the reporter could not know. Whether it is already filed. Whether the code really behaves that way. Whether anything can reach it today.

1. Read the whole issue, comments included. `gh issue view <n> --json number,title,body,state,labels,author,createdAt,comments,url`. Capture expected behavior, observed behavior, version and environment, trigger and frequency, and any error signature. Inspect the attachments. When one cannot be read, say so in the verdict instead of inventing what it shows.
2. Dedupe before you reason. `gh search issues --repo <owner>/<repo> --json number,title,state "<term>"`, run separately on the error signature, the area, and the symptom. A confident duplicate ends the triage. Name the live issue and stop. Link a plausible match as uncertain and keep going. A long-closed match is a regression lead, not a duplicate.
3. Trace the cause far enough to route, never far enough to fix. Route through the **how** skill for the path from the reported action to the observed result, and the **why** skill when the report says the behavior used to work. Mark every hypothesis as a hypothesis. If the code cannot be read, say cause tracing was unavailable rather than guessing an owner.
4. Check reachability before you set severity. Find the call site, the route, the flag, the gate. A defect nothing reaches today is real and not urgent. Say both, and name what would wake it. A severity with no reachability line behind it is a guess.
5. Classify into one of `bug`, `perf`, `feature`, `question`, `invalid`. `perf` is a bug that keeps its measurements. `invalid` covers works-as-designed, cannot-reproduce, and a wrong premise. Set severity by the repo's own convention, not a scale you invented. Read `gh label list` and how the repo rated comparable issues. When the bug-versus-feature call is a real coin flip, ask one focused question instead of picking.
6. Draft the verdict, then stop. Posting is a remote write, so it lands in your reply first and reaches the issue only on the user's word. On that word, `gh issue comment <n> --body-file <file>` and `gh issue edit <n> --add-label <label>`, after confirming the account with `gh api user --jq .login`. Closing needs its own word, `gh issue close <n> --reason duplicate` for a confirmed duplicate.
7. Hand off. Name the playbook that owns what comes next. Bug fix, Perf issue, Feature, or nothing.

**Reply:** the verdict. Classification, severity with its reachability line, dedupe findings, the cause-tracing summary with hypotheses marked as hypotheses, and the disposition. Quote the `gh` output you relied on. No fix unless asked.
