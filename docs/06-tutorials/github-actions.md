# GitHub Actions: Scheduled Snapshot Restoration

Using Snaplet with GitHub Actions is a powerful combination to automate your development infrastructure requirements, like keeping a staging database automatically updated.

In this tutorial we'll show you how to create a GitHub Actions Workflow to capture a snapshot from your source database, and restore it to your target database at 4am every morning.

**In order to restore snapshots you'll need:**

1. Setup a source database in Snaplet
2. You've run the `snaplet setup` command in your GitHub repository, which created a `.snaplet/config.json` file. This file associates your repository with your snapshots.
3. A [Snaplet CLI access token](https://app.snaplet.dev/access-token/cli).
4. A target database with superuser priviledges and a connection string to that database.

## Create a GitHub Actions Workflow

This GitHub Actions Workflow that will capture and restore a snapshot at 4am every day. It installs the Snaplet CLI and runs the `snaplet restore --new` command.

1. Create a `.github/workflows` directory in your repository on GitHub if this directory does not already exist.
2. In the `.github/workflows` directory, create a file named `snaplet-restore.yml`.
3. Copy the following yaml into `snaplet-restore.yml`:

```yaml
name: Snaplet Restore
on:
  workflow_dispatch:
    schedule:
      - cron: '0 4 * * 1-5' # At 04:00 on every day-of-week from Monday through Friday.
jobs:
  snaplet-restore-snapshot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install Snaplet CLI
        run: curl -sL https://app.snaplet.dev/get-cli/ | bash
      - name: Restore Snapshot
        run: snaplet restore --new
        env:
          SNAPLET_DATA_TARGET_DB_URL: ${{ secrets.SNAPLET_DATA_TARGET_DB_URL }}
          SNAPLET_ACCESS_TOKEN: ${{ secrets.SNAPLET_ACCESS_TOKEN }}
          # SNAPLET_DATABASE_ID: "Use if `.snaplet/config.json` isn't setup."
```

This workflow runs every morning at 4am, and can also be [manually triggered](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow#running-a-workflow)

It checks out the repository, in order to access the `SNAPLET_DATABASE_ID` from `.snaplet/config.json`, then it installs the Snaplet CLI and runs the `snaplet restore --new` command.

:::tip
Remove the `--new` flag from the `snaplet restore` command to restore the last captured snapshot rather than creating a new one.
:::

## Adding Secrets to GitHub

We're almost done. The last step is to securely add our `SNAPLET_ACCESS_TOKEN` and `SNAPLET_DATA_TARGET_DB_URL` secrets to our GitHub repository.

1. On GitHub.com, navigate to the main page of your GitHub repository.
2. Under your repository name, click on **Settings**.
3. In the left sidebar, click **Secrets**.
4. Click New repository secret.
5. Type a name for your secret in the Name input box and enter the value for your secret.
7. Click **Add Secret**.
