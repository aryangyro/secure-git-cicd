# Exercises: Secure Git and CI/CD

These exercises are meant to be done, not just read. Each one takes 15–30
minutes and uses a throwaway practice repository, so a mistake costs
nothing. Create one with `git init practice-repo` or on GitHub as a
private repo before you start.

## 1. Protect the main branch

Turn on branch protection for `main` in your practice repo (on GitHub:
Settings → Branches → Add branch protection rule). Require a pull request
before merging and require at least one approval.

Then try to push a commit directly to `main`. Watch it fail. Then open a
pull request for the same change, approve it from a second account (or ask
a friend), and merge it.

**What to notice:** how it feels to be blocked by your own rule, and how
much slower — and safer — the pull request path is. A rule you can bypass
when you are in a hurry is not a rule.

## 2. Review a planted bad diff

Ask a friend, or do it yourself in two sittings: create a branch that
changes three things at once — a real bug fix, a dependency version bump
in the lockfile, and a small change to a config file that disables a
security check. Open a pull request with a vague description like "fixes
and updates".

Now review it as if you were the approver. Find all three changes and
write down what each one does before you approve or reject it.

**What to notice:** which change was hardest to spot, and whether you
would have caught the config change if you had only skimmed the code
files. This is why the README says to review the full diff, including
lockfiles and config.

## 3. Build a tiny pipeline that actually gates a merge

In your practice repo, add a workflow file (`.github/workflows/checks.yml`
on GitHub) that runs on every pull request and does three things: run the
tests, search the diff for the word `password=` followed by a value, and
fail the build if either step fails.

Then open a pull request that breaks a test, and another that adds a
hard-coded password. Watch both get blocked. Fix them, and watch the
pipeline pass.

**What to notice:** the pipeline does not get tired, and it does not skip
checks when you are in a rush. That is the whole point of automating the
gate instead of relying on review alone.

## 4. The leaked token drill

Pretend an API token was committed to the practice repo. Work through the
response, in order:

1. Revoke the token at the provider, so it stops working.
2. Issue a replacement token and store it in the platform's secret store
   (GitHub Secrets, for example) — never in the code.
3. Remove the token from the code and commit the fix.
4. Check the Git history: the old token is still in an old commit. Decide
   what to do about that — for a practice repo, rewriting history is a
   lesson in itself; for a real repo, assume the token is compromised
   forever and rely on step 1.

**What to notice:** deleting the file does not delete the secret. Step 1
is the only step that actually stops the damage, which is why revoking
first matters more than cleaning up.

## 5. Rollback drill

Tag a release in your practice repo (`git tag v1.0.0`), then make a
deliberately bad change, tag it `v1.1.0`, and "deploy" it (for this
exercise, deploying just means noting which tag is live). Now roll back
to `v1.0.0` and time how long it takes you.

Then do the same thing without tags — by guessing which commit was the
last good one from `git log`. Compare the two times.

**What to notice:** knowing exactly which version is live, and being able
to return to it in minutes, is a security control, not just an operations
convenience. It is how you shrink the window of exposure when a bad
release ships.

## 6. Access audit

List everyone who has access to one of your real repositories (on GitHub:
Settings → Collaborators and teams). For each person, write down what
access level they have and when they last contributed.

Then answer three questions: is there anyone with write access who has
not contributed in months? Is there anyone with admin access who does
not need it? Is there a service account or token you no longer recognise?

Remove or downgrade at least one access that is no longer needed. Write
down what you changed and why.

**What to notice:** access accumulates quietly — people change teams,
contractors finish, tokens outlive their purpose. A quarterly audit is
the only thing that catches it, and this exercise is a small version of
one.
