# Work Log

## Nov 8, 2026 — Secure Git and CI/CD write-up

**What was done today**

- Wrote and posted the Discussion 4 post: managing source code securely using Git (branching, merging, access controls) and how CI/CD pipelines enhance software security.
- The post's angle is Git as a security boundary, not just a versioning tool: protected branches and pull requests as checkpoints, full-diff merge reviews including lockfiles and config, least privilege for people and service accounts, and the pipeline as an automated security gate that runs secret scanning, dependency checks, and static analysis on every pull request.
- The post was the first one in the discussion — no classmates had posted yet when it went up, so there was nothing to reply to.
- Built this repo around the post: a beginner-friendly README expanding each idea (branching, merging, access controls, CI/CD, repeatable deployments and rollback), a summary table, a glossary, limitations, and further reading. The original post is included verbatim in a clearly labelled section.

**What is still open**

- The reply is still pending. It is due Nov 28, and it cannot be written until classmates post — there is nobody to reply to yet. It will be added here once it is posted.

**Challenges**

- Keeping the post distinct from the earlier write-ups. The earlier ones covered DevSecOps and the CIA triad in general terms; this one had to stay focused on Git itself — branches, merges, access, and pipelines — without repeating the same examples.
- The closing question needed care: asking whether secret scanning matters more than code review invites a real discussion, rather than just restating the post. The README takes the position that they are layers and both are needed, which is the honest answer.
