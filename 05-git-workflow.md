# 🔀 Git & Workflow

## Q1. Describe a standard Git and GitHub workflow for collaborating within a team.

**Answer:**

I use **GitHub Flow**: `main` is always deployable. Work happens on short-lived branches. Nothing reaches `main` without a pull request and review.

```
main ───●────────────●────────●──
         \          /        /
          ●──●──●──●  feature/cart-merge
```

**Day-to-day:**

1. **Update `main`** — `git checkout main && git pull`.
2. **Branch** — `git checkout -b feature/cart-merge` (or `fix/`, `chore/`). Named after the ticket.
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

> [!TIP]
> **One-liner:** Branch from `main`, push a small PR, wait for CI + review, squash-merge, delete the branch. `main` stays green and shippable.
