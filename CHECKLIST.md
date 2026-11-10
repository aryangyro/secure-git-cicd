# Pre-merge and pre-release security checklist

A short checklist to run through before a pull request is merged and
before a release goes out. It is meant to be quick — if it takes more
than a few minutes, the pipeline is doing too little automatically.

## Before merging a pull request

- [ ] The branch is short-lived and based on a recent `main` — not a
      month-old branch with 40 commits nobody reviewed.
- [ ] At least one reviewer who did not write the code has read the
      full diff, not just the description.
- [ ] No secrets in the diff — no passwords, tokens, or private keys.
      (A scanner should catch this too, but read it yourself as well.)
- [ ] New dependencies are named in the PR. A new library is a new
      trust decision, not a detail.
- [ ] Tests pass, including the security tests. A green tick from CI
      is not a review, but a red one is a stop sign.

## Before a release

- [ ] The release is tagged, and the tag points at the exact commit
      that was reviewed and tested — not a later "quick fix".
- [ ] The pipeline that built it ran the full checks: tests, secret
      scan, and dependency scan.
- [ ] You can roll it back. If the answer is "we would have to work
      it out", do not ship yet.
- [ ] The deployment used a service account with only the permissions
      it needs — not a personal account, and not an admin token.
- [ ] Someone other than the person who wrote the release has looked
      at what is going out.

## If something is already wrong

- [ ] Revoke the secret or token first, then work out how it leaked.
      Rotating later is rotating too late.
- [ ] Roll back to the last known-good release instead of patching
      forward under pressure.
- [ ] Write down what happened while it is fresh. A short note now
      saves a long argument later about what actually went wrong.
