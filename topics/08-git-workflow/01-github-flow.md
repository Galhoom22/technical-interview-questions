# Describe a standard Git and GitHub workflow for collaborating within a team.

**Answer:**

I use **GitHub Flow**: `main` is always deployable. Work happens on short-lived branches. Nothing reaches `main` without a pull request and review.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| GitHub Flow | Branch from `main` → PR → review → merge → delete the branch. |
| `main` | Always deployable. No direct commits in a team. |
| Pull request | Proposed change for review before it hits `main`. |
| Squash merge | One commit on `main` per PR — keeps history readable. |
| `git switch -c` | Create and switch to a branch (Git 2.23+; `checkout -b` still works). |

### 🧠 Analogy

GitHub Flow is a **short side road**: leave `main` (always shippable), do the work, get a review at the junction, merge, delete the road. Nobody drives the wrong way on `main`.

---

### 📘 Official ([GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow))

```text
Create a branch → make commits → open a pull request →
address review → merge → delete the branch
```

### 💼 In production

```bash
git switch main && git pull
git switch -c feature/cart-merge
git commit -m "Merge guest cart into user cart on login"
git push -u origin HEAD
# PR, CI (tests + Pint), one approval, squash-merge, delete branch
```

**Day-to-day:**

1. **Update `main`** — `git switch main && git pull`.
2. **Branch** — `git switch -c feature/cart-merge` (or `fix/`, `chore/`). Named after the ticket. (`git checkout -b` still works; `switch` is the Git 2.23+ command meant for this.)
3. **Small commits** — present tense, one idea each: `Add cart merge on login`.
4. **Push** — `git push -u origin HEAD`.
5. **Open a PR** — description, screenshots, test plan, link the issue.
6. **CI must pass** — tests, Pint, static analysis. I do not merge red builds.
7. **Review** — at least one approval. Address comments with new commits (or amend only if the PR is still mine and not shared).
8. **Merge** — squash merge to keep `main` readable, or merge commit if the team wants history.
9. **Delete the branch** — local and remote. Deploy from `main`.

**Rules I call out in interviews:**

| Do | Don’t |
|----|--------|
| Pull/rebase on latest `main` before the PR | Commit straight to `main` |
| Keep PRs small and reviewable | One 2,000-line “fix stuff” PR |
| `.env` and secrets stay out of Git | Force-push `main` |
| Resolve conflicts on the branch | Rewrite published history others pulled |

**Conflicts:** update `main`, `git merge main` (or rebase) on the feature branch, fix files, test, push. For a mistake that never left my machine: `git restore` / a new commit. I do not `reset --hard` on shared branches.

Hotfixes follow the same path, just faster: branch from `main`, PR, merge, deploy.

---

> [!WARNING]
> No commits on `main`. No secrets in Git. No force-push on `main`. Do not `reset --hard` on a branch others pulled.

> [!NOTE]
> - Conflicts? Merge/rebase `main` onto the feature branch, fix, push.
> - Squash vs merge commit? Squash for a readable `main`; team policy wins.

---

> [!TIP]
> **One-liner:** Branch from `main`, push a small PR, wait for CI + review, squash-merge, delete the branch. `main` stays green and shippable.

**Source:** [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow) — official branch → commit → pull request → review → merge → delete branch.

**Learn more:**
- [About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) — what a PR is for
- [`git switch`](https://git-scm.com/docs/git-switch) — current Git command to switch/create branches (`git switch -c`)
- [Git: Branching Workflows](https://git-scm.com/book/en/v2/Git-Branching-Branching-Workflows) — topic branches (the Git book, not GitHub-specific)
- [Laravel Pint](https://laravel.com/docs/13.x/pint) — first-party style fixer most Laravel CI jobs run
- [Resolving a merge conflict using the command line](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-using-the-command-line) — what to do when `main` moved

---

[Topic](./README.md)
