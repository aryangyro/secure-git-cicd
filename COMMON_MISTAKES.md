# Common mistakes — Git and CI/CD security

The `README.md` explains how Git and CI/CD are supposed to work. This
file is about how they go wrong in practice — the mistakes teams make
over and over, usually with good intentions.

---

## 1. Pushing straight to main "just this once"

It starts as a one-off: a tiny fix, no time for a pull request. Then
it becomes a habit, and then nobody reviews anything. The whole point
of a protected branch is that there are no exceptions — the day you
make one is the day the rule stops meaning anything. If the fix is
really that small, the pull request takes two minutes.

## 2. Treating a pull request approval as a rubber stamp

An approval that took ten seconds is not a review. The reviewer
skimmed the diff, saw nothing obviously broken, and clicked approve.
Real review means reading the change, understanding what it does, and
asking at least one question. If nobody on the team has time to review
properly, the problem is the size of the pull requests, not the
reviewers — small changes get real reviews, huge ones get stamps.

## 3. Secrets in the pipeline configuration

CI/CD files live in the repository, so it is tempting to put an API key
or a deployment password right there in the YAML. Anyone who can read
the repo can read the secret, and the secret is in the git history
forever. Use the platform's secret store instead, and pass secrets to
jobs as environment variables. And never print them in build logs —
one `echo` in a debug step can leak a token to everyone who can see
the pipeline output.

## 4. Running the pipeline with more permissions than it needs

A pipeline that can deploy to production, read every secret, and write
to every repository is a very attractive target. If an attacker gets
code into a pull request that runs in that pipeline, they inherit all
of it. Give each job the smallest permissions it needs: read-only
where possible, write access only for the exact job that needs it, and
no access to production secrets for pull requests from forks.

## 5. Trusting the green tick too much

A passing pipeline means the checks you wrote passed. It does not
mean the code is secure. A test suite with no security tests passes
insecure code happily. A scanner that is misconfigured passes
everything. Treat the green tick as "the checks we asked for ran and
passed", not as "this is safe". Every so often, check that the checks
themselves are actually working — a pipeline that always passes is
suspicious, not reassuring.

## 6. Force-pushing shared branches

Force push rewrites history. On your own private branch that is fine.
On a shared branch it destroys other people's work and, worse, it
destroys the record of what changed and when. That record is exactly
what you need when something goes wrong. Protect shared branches
against force pushes, and if history really needs rewriting, do it
with the team agreeing first, not alone at midnight.

## 7. No plan for when a secret leaks

Sooner or later, a real secret ends up in the repository — someone
commits the wrong file, a token gets pasted into a test. Teams that
have never thought about this panic: they delete the file, push a new
commit, and assume the problem is gone. It is not — the secret is
still in the git history, and anyone who cloned the repo still has it.
The plan should be written down before it happens: rotate the secret
immediately, check who used it, and only then clean up the history.
Deleting the file is the last step, not the first.
