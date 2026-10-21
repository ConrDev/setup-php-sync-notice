# setup-php sync notice
[![PPA sync](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fraw.githubusercontent.com%2FConrDev%2Fsetup-php-sync-notice%2Fmain%2Flast_seen_jammy.json&query=%24.status&label=PPA%20sync)](https://github.com/ConrDev/setup-php-sync-notice/actions/workflows/ppa-sync-notice.yml)
[![Last PPA update](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fraw.githubusercontent.com%2FConrDev%2Fsetup-php-sync-notice%2Fmain%2Flast_seen_jammy.json&query=%24.updated&label=last%20PPA%20update)](last_seen_jammy.json)

Posts to a Discord channel when the [setup-php PPA mirror](https://ppa.setup-php.com/ondrej/php/ubuntu/) publishes new package versions.

Every hour a GitHub Actions workflow downloads `dists/jammy/main/binary-amd64/Packages`, compares each package's version against `last_seen_jammy.json`, and sends a message listing what changed.

The `Packages` file is watched rather than `InRelease`, because the mirror updates `Packages` without always updating `InRelease`.

## Setup

1. In Discord, open **Channel settings → Integrations → Webhooks**, create a webhook and copy its URL.
2. In this GitHub repository, open **Settings → Secrets and variables → Actions** and add a repository secret named `DISCORD_WEBHOOK_URL` with that URL.
3. Push the repository to GitHub.
4. Open **Actions → PPA sync notice → Run workflow** to run it once. This first run commits `last_seen_jammy.json` with the current package list and does not post anything; later runs post when versions change.

## State

`last_seen_<dist>.json` holds:

- `status`: `updated` if the latest check found new versions, otherwise `no update`.
- `updated`: the mirror's `Last-Modified` time for the `Packages` file.
- `packages`: every package and version on the mirror.

The workflow commits the file whenever any of these change, so its git history is a log of every PPA update. It only posts to Discord when versions change.

The **PPA sync** and **last PPA update** badges above read `status` and `updated` from that file. They show an error until the first run has committed the file, and only work while the repository is public.

If posting to Discord fails, the workflow fails before committing, so the next run detects the same change and tries again.

## Configuration

Settings live in `.github/workflows/ppa-sync-notice.yml`:

- `DISTS`: Ubuntu releases to watch, separated by spaces (default `jammy`). For example, `jammy noble`. The releases update at the same time, so adding more mostly repeats the same message.
- `cron`: how often to check (default hourly). See Cost before increasing it.

## Cost

This stays within GitHub's free plan:

- **Public repository:** Actions minutes on standard runners are free and unlimited.
- **Private repository:** the free plan includes 2,000 minutes per month, shared across all your private repositories. Each run takes a few seconds but is billed as a full minute, so hourly checks use about 720 minutes per month. Checking every 30 minutes would use about 1,440, leaving little room for other workflows.
- Each run is capped at 2 minutes (`timeout-minutes`), so a stuck download can't use up minutes.

## Scheduling notes

- GitHub's cron scheduling is best-effort; runs can be delayed by several minutes.
- GitHub disables scheduled workflows after 60 days with no repository activity. The bot's state commits count as activity, so this only happens if the PPA goes 60 days without an update. If notices stop, re-enable the workflow in the Actions tab.
