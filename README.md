# Managing Source Code Securely with Git and CI/CD

Most people first meet Git as a backup tool: a place to save code so it is not lost when a laptop dies. That is true, but it misses the bigger point. In a team, Git is a security boundary — the code in a repository *is* the product. Who can push what, what happens before code reaches the main branch, and what checks run automatically on the way there matter as much as the code itself.

This write-up explains, in plain English, how to manage source code securely using Git — branching, merging, and access controls — and how Continuous Integration / Continuous Deployment (CI/CD) pipelines make software more secure. It is written for beginners, and the examples come from the way real teams actually work.

## Git as a security boundary

A repository holds everything that makes the product run: the source code, the configuration, the database scripts, and the deployment settings. If someone can change the code, they can change what the product does. If they can read it, they can learn how it works and where its weak points might be.

That makes Git a security boundary in the same way a locked door is one. The question is not just "can we get the code back if something goes wrong?" but "who is allowed to change what, and how do we know what changed?"

Three ideas run through everything that follows: nothing reaches the main branch without being checked; every change should be traceable to the exact change, author, and reviewer; and machines should check what humans forget, because automated checks run the same way every time.

## Part 1: Branching — the first control

### The problem with pushing straight to main

If everyone pushes straight to main, there is no checkpoint between a developer's keyboard and the production code. One bad commit can break the build, one compromised account can push malicious code, and one tired developer can overwrite someone else's fix. Worse, there is no moment for anyone else to look at the change — the code goes live before it has been reviewed.

### Short-lived feature branches

The standard answer is short-lived feature branches. Each piece of work gets its own branch, created from main, used for that one change, and deleted after it is merged.

Short-lived matters. A branch that lives for a day or two is easy to review and merge, because it has not drifted far from main. A branch that lives for three weeks becomes a merge nightmare, and big diffs get skimmed instead of reviewed. Small branches are a security practice, not just a tidiness practice.

### Pull requests as checkpoints

A pull request (called a merge request on some platforms) is how a branch gets back into main. It is a proposal — "here is what I changed, please look at it" — and a checkpoint that forces a pause where someone else reviews the change before it lands.

A good pull request has a small, focused diff (one change, one purpose — a pull request that fixes a bug, refactors three files, and updates a dependency is three pull requests pretending to be one), a clear description of what changed, why, and how it was tested, and at least one reviewer who is not the author. Fresh eyes catch the mistakes the author has read past ten times.

### Protected branches

A protected branch is a branch with rules enforced by the hosting platform (GitHub, GitLab, Bitbucket, and others). The most common rules for main are: no direct pushes (every change comes through a pull request, not even from the repository owner); required approvals (one or two, from other team members); required status checks (the pull request cannot merge until the pipeline has passed); no force pushes (nobody can rewrite main's history, which would destroy traceability); and branches must be up to date with main before merging, so the change is tested against the current code.

These rules slow things down a little, and that is the point: a single stolen password is no longer enough to ship something bad, because the attacker would also need a pull request approved and the automated checks passed. A protected main branch turns a single point of failure into a chain an attacker has to break in several places at once.

## Part 2: Merging — where the risk hides

Merging is where a lot of the risk hides: a merge that looks clean can still bring in a dependency change, an edited config file, or a secret added to a settings file.

### Review the full diff

A merge review should look at the full diff, not just the code files, including:

- **Lockfiles** (`package-lock.json`, `requirements.txt`, `Gemfile.lock`, and similar). These pin exact versions of every library the project uses. A change here means a different version of a library will run in production, and that version might have a known vulnerability — or it might be a malicious package that looks like a real one.
- **Configuration files.** A change here can turn off authentication, open a port, disable logging, or point the application at a different database. Config changes are small, easy to overlook, and capable of doing enormous damage.
- **Build and deployment scripts.** Changes here affect how the code is built and shipped, and a modified build script could inject extra code into the final product.
- **Database migrations.** A migration that drops a column or changes a constraint can destroy data, so migrations deserve the same careful review as application code.
- **New files.** It is easy to hide a malicious script or a hard-coded credential in a new file that reviewers skip because it is "just a helper."

Read the diff from top to bottom, including the boring parts — that is where the surprises live.

### Squash merging and traceable history

When a feature branch is merged, there are a few ways to do it. The two common ones are a regular merge commit, which keeps every commit on the branch, and squash merging, which compresses the whole branch into a single commit on main.

Squash merging helps keep history readable. A feature branch might contain twenty commits — "work in progress", "fix typo", "try again", "revert that" — none of which matter to someone reading the history later. Squashed into one commit with a clear message ("Add password reset with email token"), the history becomes a clean list of what changed and why.

That readability is a security feature. When something breaks later, a clean history lets `git blame` and `git log` answer "when did this line change?" in seconds; with a messy history, the same question takes an afternoon, and in an incident, an afternoon is a long time. Whatever style a team chooses, consistency matters: one style, applied everywhere.

## Part 3: Access controls — least privilege

Access control in Git is mostly about least privilege: give each person (and each machine) the smallest amount of access they need, and no more.

### The usual roles

Most Git hosting platforms have a small set of access levels:

| Role | What it can do | Who should have it |
| --- | --- | --- |
| Read | Clone and view the code, open pull requests | Most of the team, auditors, new joiners |
| Write | Push branches, open and merge pull requests | Contributors working on the code |
| Maintain / Admin | Change settings, manage access, change branch protection | One or two people, usually the tech lead |

Not everyone needs write access, and almost nobody needs admin. Read access is enough for most of the team — they can read the code, open issues, and propose changes through pull requests. Admin access should be rare, because an admin can turn off branch protection, change who has access, and rewrite history.

### Who gets what, and when

A few practical rules that teams use:

- **Default to read.** New joiners start with read access and get write access when they start contributing — the opposite of the common default, and much safer.
- **Review access regularly.** A quarterly check of who has access catches the accounts that should have been removed when someone changed teams or stopped contributing.
- **Remove access promptly when someone leaves.** This is the most commonly missed step, and an ex-employee with write access is a real risk.
- **Use teams, not individuals.** Assign access to a team (for example, "backend developers") so access follows automatically when someone joins or leaves.

### Service accounts and CI tokens

Service accounts are the accounts machines use: the CI pipeline, the deployment tool, the bot that updates dependencies. They need access too, but only the exact permissions they need. A CI token with full repository access is a huge risk if it leaks — CI logs are often visible to more people than the code is, and tokens end up in log files and error messages. A token that can only read the repository and post status checks is far less dangerous than one that can push code, change settings, or delete branches.

Practical rules for service accounts: scope tokens narrowly (read-only where possible), give them short lifetimes and rotate them regularly, use separate tokens for separate jobs so one leak does limited damage, and never put a token in code. Tokens belong in the platform's secret store (GitHub Secrets, GitLab CI variables, and similar). This is the most common way secrets leak: someone commits a config file with a real token in it, and it stays in the Git history forever, even after the file is deleted — anyone who clones the repository can find it in an old commit.

## Part 4: CI/CD — the automated security gate

A pipeline is really an automated security gate: it runs tests, but it also runs security checks on every pull request, before anything merges.

### What a pipeline checks

A typical secure pipeline runs several kinds of checks on every pull request:

- **Tests.** The code still works — the basic check, catching broken changes before they reach main.
- **Secret scanning.** The diff is searched for things that look like secrets: API keys, passwords, private keys, tokens. Tools like GitHub's secret scanning, Gitleaks, and TruffleHog do this automatically. A leaked API key caught by the machine is a minor incident; the same key reaching production is a major one.
- **Dependency checks.** The project's libraries are checked against databases of known vulnerabilities (Dependabot, Snyk, OWASP Dependency-Check). If a library has a known vulnerability, the pipeline fails. This matters because most modern applications are mostly third-party code — your code might be 10% of what runs, and the other 90% is libraries you did not write.
- **Static analysis (SAST).** The code is analysed without running it, looking for known-unsafe patterns: SQL injection, cross-site scripting, hard-coded credentials, insecure random numbers, weak cryptography. Tools like CodeQL, SonarQube, and Semgrep do this. Static analysis does not catch everything, but it catches common mistakes early, when they are cheap to fix.
- **Build check.** The code actually builds. A change that breaks the build should never reach main.

Because the pipeline runs the same checks every time, it does not skip them when the team is in a rush — which is exactly when mistakes happen. A tired reviewer might skim a diff and miss a hard-coded password. The pipeline does not get tired, and it does not skim.

### Why automation beats review alone

Code review and automated checks catch different things. A reviewer is good at judging intent: does this code do what it was supposed to do, and is there a business-logic flaw no tool would spot? A pipeline is good at judging patterns: is there a secret in this diff, is this library version vulnerable, is this code using a known-unsafe function?

The question at the end of the original post — does secret scanning matter more than code review, or is review still the main defence? — has a practical answer: they are layers, and removing either one weakens the whole thing. Secret scanning catches things reviewers almost never catch, because a 40-character random string looks the same whether it is a real key or a placeholder. Review catches things scanners never catch, because a scanner cannot tell that code grants access to the wrong users. The strongest setup is both, and the pipeline makes sure the automated half happens every single time.

### The pipeline itself needs securing

A pipeline that can deploy to production is a powerful tool, and it needs protecting: treat the pipeline configuration file (for example, `.github/workflows/`, `.gitlab-ci.yml`) like code, with changes going through pull requests and review — an attacker who can change the pipeline can make it skip security checks. Only the main branch or tagged releases should be able to deploy to production, never a feature branch. Many teams add a manual approval step before production deployments. And pipeline logs should never print secrets — treat them as semi-public and keep them clean.

## Part 5: Repeatable deployments and rollback

Pipelines also make deployments boring and repeatable. When a release is just a pipeline run from a tagged commit, you know exactly what code is running in production, and you can roll back to the last good tag in minutes.

### Tags as the source of truth

A Git tag is a label on a specific commit, usually a version number like `v1.4.2`. When a team tags a release and deploys from that tag, the tag becomes the source of truth for what is running: anyone can answer "what version is in production?" and reproduce that exact version by checking out the tag. That makes incident response fast — if a release has a vulnerability, you know which commit introduced it and can roll back to the previous tag while you fix it — and it makes auditing possible, since you can prove which code was running at any given time.

### Manual deployments are where things go wrong

Manual deployments are where people copy files around and lose track of what changed. Someone copies a file to a server by hand, someone else edits a config file directly on the server, and after a few weeks nobody knows what is actually running. That drift is dangerous: the server no longer matches the repository, so a fix that is merged may not actually reach production, or a change that was never reviewed is running live.

The rule is simple: if it is not in Git, it is not deployed. Servers should be treated as replaceable — you should be able to rebuild one from Git and the pipeline alone, with no manual steps.

### Rollback as a security control

Rollback is often thought of as an operations feature, but it is a security control too. When a vulnerability is discovered in a release, the fastest mitigation is usually to roll back to the last known-good version while a fix is prepared. A team that can roll back in minutes has a much smaller window of exposure than one that needs hours to rebuild by hand. Practising rollbacks is worth the time, because the first time you need one should not be during a real incident.

## Putting it together: layers, not one feature

Secure Git is not one feature, it is layers: protected branches (nothing reaches main without a pull request and approvals); reviewed merges (the full diff is read, including lockfiles, config, and new files); tight access (least privilege for people, narrowly scoped tokens for machines); a pipeline that checks everything (tests, secret scanning, dependency checks, static analysis on every pull request); and repeatable deployments (tagged releases, deployed by the pipeline, with fast rollback).

Each layer catches what the others miss. Branch protection stops a direct push; review stops a bad design; access control limits the damage of a compromised account; the pipeline catches the secrets and vulnerable libraries reviewers miss; tagged releases make it possible to know what is running and to roll back fast. Remove any one layer and attackers will find the weakest one.

## The original post

> I think most people treat Git as a backup tool, but in a team it is really a security boundary. The code in a repo is the whole product, so who can push what, and what happens before code reaches the main branch, matters as much as the code itself.
>
> Branching is the first control. If everyone pushes straight to main, one bad commit or one compromised account can break everything or slip in malicious code. With short-lived feature branches and pull requests, nothing lands in main without being looked at. The branch itself is a checkpoint, it forces a pause where someone else has to review the change. I have seen teams where the main branch was protected so nobody, not even the repo owner, could push directly, and every merge needed two approvals. It slows things down a little, but it also means a single stolen password is not enough to ship something bad.
>
> Merging is where a lot of the risk hides. A merge that looks clean can still bring in a dependency change or a config file nobody noticed. That is why I think merge reviews should look at the full diff, including lockfiles and config, not just the code files. Squash merging helps here too, it keeps history readable so when something breaks later you can actually trace which change caused it.
>
> Access control in Git is mostly about least privilege. Not everyone needs write access, and almost nobody needs admin. Read access for most of the team, write for contributors, and admin for one or two people. And service accounts, the ones CI uses, should only have the exact permissions they need. A CI token with full repo access is a huge risk if it leaks.
>
> The part that changed my mind about CI/CD is that a pipeline is really an automated security gate. A good pipeline runs tests, but it also runs secret scanning, dependency checks, and static analysis on every pull request, before anything merges. So a leaked API key or a known vulnerable library gets caught by the machine, not by a tired reviewer at the end of the day. And because the pipeline runs the same checks every time, it does not skip them when the team is in a rush, which is exactly when mistakes happen.
>
> The other thing pipelines do is make deployments boring and repeatable. When a release is just a pipeline run from a tagged commit, you know exactly what code is running in production, and you can roll back to the last good tag in minutes. Manual deployments, by contrast, are where people copy files around and lose track of what changed.
>
> So to me, secure Git is not one feature, it is layers: protected branches, reviewed merges, tight access, and a pipeline that checks everything before it ships. Does anyone else think secret scanning in the pipeline matters more than code review, or is review still the main defence?

No classmates had posted yet when this was written, so no reply is included.

## Summary table

| Practice | What it does | What happens without it |
| --- | --- | --- |
| Short-lived feature branches | Every change gets reviewed before reaching main | Bad or malicious code goes straight to production |
| Protected main branch | No direct pushes, approvals required, checks must pass | One compromised account can ship anything |
| Full-diff review | Lockfiles, config, and new files are checked | A dependency or config change slips through unnoticed |
| Squash merging | Clean, readable history | Nobody can trace which change caused a bug |
| Least privilege | People and tokens get only the access they need | A leaked token or a compromised account does maximum damage |
| Secret scanning in CI | Leaked keys and passwords are caught before merge | Secrets reach production and stay in Git history |
| Dependency checks in CI | Known-vulnerable libraries are blocked | The product ships with known vulnerabilities |
| Static analysis in CI | Common unsafe patterns are flagged automatically | SQL injection and similar flaws reach main |
| Tagged releases | You know exactly what code is running | Nobody knows what version is live, or what changed |
| Pipeline deployments | Deployments are repeatable and auditable | Manual changes drift from the repository |

## Limitations

- This write-up describes practices, not a specific setup. The exact settings differ between GitHub, GitLab, Bitbucket, and self-hosted Git, and teams should adapt the ideas to their own platform.
- No control here stops an attacker who has already compromised an admin account and a reviewer's account at the same time. Defence in depth reduces risk; it does not eliminate it.
- Automated checks have false positives and false negatives. Secret scanners sometimes flag harmless strings, and static analysis sometimes misses real flaws. A clean scan does not mean the code is safe.
- Small teams and solo developers cannot always afford two approvals on every pull request. Protected branches and a pipeline that runs checks are still valuable for one person, but the exact rules should match the size of the team.
- This write-up focuses on the repository and the pipeline. It does not cover the hosting platform, the developers' machines, or the production servers, all of which matter just as much.

## Glossary

| Term | Meaning |
| --- | --- |
| Repository (repo) | A project stored in Git, with its code, history, and settings. |
| Branch | A separate line of development; main is the version that gets deployed. |
| Feature branch | A short-lived branch for one piece of work, merged back into main when done. |
| Pull request / merge request | A proposal to merge a branch into main, with review and discussion. |
| Protected branch | A branch with platform-enforced rules, such as required approvals and no direct pushes. |
| Merge | Combining a branch into another branch, usually into main. |
| Squash merging | Compressing a whole branch into a single commit when merging. |
| Diff | The list of changes between two versions of the code. |
| Lockfile | A file pinning the exact versions of every library a project uses. |
| Least privilege | Giving each person or machine only the access they need, and no more. |
| Service account | An account used by a machine (such as a CI pipeline) rather than a person. |
| CI token | A credential that lets a pipeline act on a repository. |
| CI/CD | Continuous Integration / Continuous Deployment: automatically building, testing, and deploying code on every change. |
| Secret scanning | Automatically searching code for API keys, passwords, or tokens. |
| Dependency check | Checking a project's libraries against databases of known vulnerabilities. |
| Static analysis (SAST) | Analysing code without running it, looking for known-unsafe patterns. |
| Tag | A label on a specific commit, usually a version number, marking a release. |
| Rollback | Returning to a previous known-good version after a bad release. |

## Further reading

GitHub Docs ("About protected branches", "About secret scanning"), GitLab Docs ("Protected branches", "CI/CD security"), Atlassian Git Tutorials (branching workflows and pull requests), OWASP ("Source Code Management Security", OWASP Top 10), NIST SP 800-218 (Secure Software Development Framework).
