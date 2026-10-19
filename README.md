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

## Adding new repos

1. Create the repo on GitHub as usual.
2. Create an empty repo with the same name on Gitee (web UI or API).
3. The next scheduled run picks it up automatically. Until step 2 is done,
   the run logs a `push failed` line for it (harmless).

## Changing the schedule

Edit the `cron` line in `.github/workflows/mirror-to-gitee.yml` and commit.
