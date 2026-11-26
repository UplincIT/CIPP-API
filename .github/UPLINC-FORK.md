# UplincIT/CIPP-API: fork maintenance

This repo is Uplinc's fork of `KelvinTegelaar/CIPP-API` (branch `master`). It
carries fork-only changes that **an upstream sync must never remove**.

## Fork-only state

| What | Why |
| --- | --- |
| `.github/workflows/master_cippd4ukg.yml` | Deploys `master` to Uplinc's Azure Function App **cippd4ukg** (Production slot). Upstream has no deploy for forks. Uses secret `AZUREAPPSERVICE_PUBLISHPROFILE_96523667864748B6A37F8FEE948F0121`. |
| `.github/UPLINC-FORK.md` | This file. |
| GitHub setting: workflow **PR Branch Check** (`PR_Branch_Check.yml`) disabled | Upstream's check auto-closes every PR from a fork into `master`, which includes our own PRs. Leave it disabled (`gh workflow list -R UplincIT/CIPP-API --all`). |

History: on 2026-11-07 `master` was force-updated to the upstream head (GitHub
"Sync fork → Discard commits"), which deleted the deploy workflow and stopped
deploys until it was restored on 2026-11-25.

## Syncing upstream safely

Never use **Sync fork → Discard commits**, `gh repo sync --force`, or any
`git push --force` to `master`. Those reset `master` to upstream and drop the files above.

1. See how far behind/ahead the fork is:
   ```sh
   gh api repos/UplincIT/CIPP-API/compare/KelvinTegelaar:CIPP-API:master...UplincIT:CIPP-API:master \
     --jq '{ahead:.ahead_by, behind:.behind_by}'
   ```
   `behind: 0` means nothing to do. `ahead` is normally ≥ 1 (our fork-only commits); that is expected.
2. Merge upstream with GitHub's merge API (a normal merge commit, never a reset):
   ```sh
   gh api -X POST repos/UplincIT/CIPP-API/merge-upstream -f branch=master
   ```
   - `200` = merged (or fast-forwarded). Continue at step 3.
   - `409` = conflict. Do **not** discard. Merge locally on a branch and open a PR:
     ```sh
     git fetch upstream && git switch -c sync/upstream-$(date +%F) origin/master
     git merge upstream/master   # resolve; keep the fork-only state above
     git push -u origin HEAD && gh pr create -R UplincIT/CIPP-API -B master
     ```
3. Verify the deploy workflow survived:
   ```sh
   gh api repos/UplincIT/CIPP-API/contents/.github/workflows/master_cippd4ukg.yml --jq .path
   ```
   If it is missing, restore it from history
   (`git log --all -- .github/workflows/master_cippd4ukg.yml`) through a PR,
   and report what removed it.

## Checking a deploy ran

Every push to `master` triggers **Build and deploy Powershell project to Azure
Function App - cippd4ukg**.

```sh
HEAD=$(gh api repos/UplincIT/CIPP-API/commits/master --jq .sha)
gh run list -R UplincIT/CIPP-API --workflow master_cippd4ukg.yml \
  --branch master --event push -L 5 --json headSha,status,conclusion,url
```

The newest run's `headSha` must equal `$HEAD` with `conclusion: success`. If it
failed, read `gh run view <id> --log-failed`; retry with `gh run rerun <id>` or
`gh workflow run master_cippd4ukg.yml -R UplincIT/CIPP-API --ref master`.
Upstream's own deploy workflows (`dev_cippahmcc.yml`, `dev_cippjta72.yml`,
`dev_api.yml`, etc.) target CyberDrain's apps; their runs here are not ours.

Never print or change the publish profile secret; only check it exists by name:
`gh secret list -R UplincIT/CIPP-API`.
