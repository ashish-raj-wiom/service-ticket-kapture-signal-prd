# Kapture Assignment Comment — PRD

When a CSP assigns someone to a service ticket, add a comment to the Kapture ticket naming them,
so a call-centre agent taking a callback can say who is working on it.

**Read it:** https://ashish-raj-wiom.github.io/service-ticket-kapture-signal-prd/

| File | What it is |
|---|---|
| `Kapture_Assignment_Comment_PRD.md` | The PRD. Wiom Template v4. **The single source of truth.** |
| `Kapture_Assignment_Comment_Tradeoffs.md` | The seven decisions behind it, the measurements, and the code facts the spec rests on. |
| `index.html` | Renders the markdown live from this repo. Holds no content of its own. |

## How to change the document

Edit the markdown, commit, push. The page re-renders itself — there is no build step and no
generated copy to keep in sync.

```bash
git add -A && git commit -m "PRD: <what changed>" && git push
```

GitHub caches raw files for a few minutes, so a fresh push can take a moment to appear.

## Status

v0.1 Draft. Lint clean against the Template v4 checklist, with every acceptance criterion scored
against the AC rubric. One override recorded: the push is best effort, so a dropped comment is
invisible.

Sibling of the [Service Ticket Chat PRD](https://ashish-raj-wiom.github.io/service-ticket-handler-notice-prd/)
— same trigger, different audience. Decisions that apply to both are deliberately kept identical:
one comment per action, and a genuine re-assign of the same person still sends.

Blocking sign-off:

1. No engineering reviewer named.
2. No consulted domains named.
3. The comment's exact wording is not fixed — the PRD says what it must carry, not the sentence.

## The thing to know before reading

Nobody is assigned during the window when agents most need this. A restore ticket waits a median
131 minutes from reaching the CSP to the first assign action, and 73% are resolved within five
minutes of it. This feature fires at the end of that wait, not inside it — useful on the ~22% of
tickets with a real gap, and on half of technician-assigned ones. Section 1 says so explicitly.
