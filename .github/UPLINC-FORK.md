# CIPP-API: Uplinc fork notes

**This fork is NOT in Uplinc's deploy path.**

Production CIPP (`cipp.uplinc.com`) runs on the Azure web app **cippd4ukg**
(Uplinc tenant, resource group `CIPP`) as a Linux container using upstream's
combined frontend+API image, `ghcr.io/cyberdrain/cipp:latest`. The app's own
managed identity refreshes its site config daily (around 03:57Z), so CIPP
updates itself from upstream's image. Nothing in this repo builds or deploys it.

## Do not

- Re-enable or restore the old deploy workflows:
  - UplincIT/CIPP-API `.github/workflows/master_cippd4ukg.yml` (function zip deploy, now **disabled**).
    Pushing a function zip onto the container app would break production.
  - UplincIT/CIPP `.github/workflows/azure-static-web-apps-brave-tree-02a14ea0f.yml`
    (the brave-tree Static Web App no longer exists).
  - UplincIT/CIPP `.github/workflows/azure-static-web-apps-red-stone-0d8c5ed03.yml`
    (upstream's own SWA, not Uplinc's; **disabled** on the fork).
- Re-enable SCM basic-auth publishing on cippd4ukg, or refresh publish-profile /
  SWA token secrets for these workflows.

## Syncing upstream (optional)

Keeping this fork current is housekeeping only; it does not change production.
If you sync, merge, never reset:

```sh
gh api -X POST repos/UplincIT/CIPP-API/merge-upstream -f branch=<main|master>
```

Never use "Sync fork → Discard commits", `gh repo sync --force` or a force push.
A discard-commits sync on 2026-11-07 is what silently deleted the fork's old deploy workflows.

## Checking the workflows stay off

```sh
gh workflow list -R UplincIT/CIPP --all | grep -i 'static web apps'
gh workflow list -R UplincIT/CIPP-API --all | grep -i cippd4ukg
```

Each must show `disabled_manually` or be absent (`deleted` is also fine).
A sync can add new files but does not re-enable a disabled workflow.
