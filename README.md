# gitee-mirror

Scheduled CI that mirrors every owned, non-fork repository from GitHub
(`123123213weqw`) to Gitee (`wangyue2006`), branches and tags included.

## How it works

- Runs on GitHub Actions every 12 hours (`0 */12 * * *` UTC) and can be
  triggered manually from the **Actions** tab → **mirror-to-gitee** →
  **Run workflow**.
- For each repo: `git clone --mirror` from GitHub, strips GitHub-internal
  pull-request refs, then `git push --mirror` to Gitee. Force-syncs, so
  branches deleted on GitHub are also deleted on Gitee.
- The `gitee-mirror` repo itself is excluded.
- This repo is intentionally **public** (public repos get free, unlimited
  Actions minutes). Because public workflow logs are world-readable, the
  workflow never prints private repo names — progress is reported by index.
- Each run commits a `.last-run` timestamp, because GitHub auto-disables
  scheduled workflows on public repos after 60 days without a commit.

## Secrets (Actions → Settings → Secrets → Actions)

| Secret           | Content                                            |
| ---------------- | -------------------------------------------------- |
| `GITEE_SSH_KEY`  | Private SSH key authorized for the Gitee account   |
| `GH_MIRROR_PAT`  | GitHub token with `repo` scope (reads private repos) |
| `GITEE_TOKEN`    | Gitee personal access token with `projects` scope. Optional: when set, the workflow creates missing Gitee repos itself |

## Adding new repos

1. Create the repo on GitHub as usual.
2. The next scheduled run creates the empty Gitee repo and pushes to it, as
   long as `GITEE_TOKEN` is set. Without that secret, create the empty repo on
   Gitee by hand first — until then the run logs a `push failed` line for it
   (harmless).

Gitee has no "create on push", so `GITEE_TOKEN` is what turns a brand-new
GitHub repo into a mirrored one without manual work. Two Gitee quirks are
handled explicitly, both verified against the live API:

- `POST /user/repos` accepts and ignores `private`, so every API-created repo
  starts private. The workflow therefore issues a follow-up
  `PATCH /repos/{owner}/{repo}` to match the GitHub visibility.
- That `PATCH` is rejected on an empty repo (`空仓库不支持设置为公开仓库`), so it
  runs only after the first successful push — and only for repos the run
  created itself, never for repos that were already there. Flipping an
  existing Gitee repo between public and private stays a manual decision.

## Changing the schedule

Edit the `cron` line in `.github/workflows/mirror-to-gitee.yml` and commit.
