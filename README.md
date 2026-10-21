# setup-php sync notice

Posts to a Discord channel when the [setup-php PPA mirror](https://ppa.setup-php.com/ondrej/php/ubuntu/) publishes new package versions.

Every hour a GitHub Actions workflow downloads `dists/jammy/main/binary-amd64/Packages`, compares each package's version against the previous run, and sends a message listing what changed.

The `Packages` file is watched rather than `InRelease`, because the mirror updates `Packages` without always updating `InRelease`.

## Setup

1. In Discord, open **Channel settings → Integrations → Webhooks**, create a webhook and copy its URL.
2. In this GitHub repository, open **Settings → Secrets and variables → Actions** and add a repository secret named `DISCORD_WEBHOOK_URL` with that URL.
3. Push the repository to GitHub.
4. Open **Actions → PPA sync notice → Run workflow** to run it once. This first run saves the current package list and does not post anything; later runs post when versions change.

## Configuration

Settings live in `.github/workflows/ppa-sync-notice.yml`:

- `DISTS`: Ubuntu releases to watch, separated by spaces (default `jammy`). For example, `jammy noble`. The releases update at the same time, so adding more mostly repeats the same message.
- `cron`: how often to check (default hourly). See Cost before increasing it.

## Notes

- GitHub disables scheduled workflows after 60 days with no repository activity. If notices stop, re-enable the workflow in the Actions tab.
- The previous package list is stored in the GitHub Actions cache. If that cache is lost, the next run saves a new list without posting.

## Cost

This stays within GitHub's free plan:

- **Public repository:** Actions minutes on standard runners are free and unlimited.
- **Private repository:** the free plan includes 2,000 minutes per month, shared across all your private repositories. Each run takes a few seconds but is billed as a full minute, so hourly checks use about 720 minutes per month. Checking every 30 minutes would use about 1,440, leaving little room for other workflows.
- Each run is capped at 2 minutes (`timeout-minutes`), so a stuck download can't use up minutes.
- The cached package list is about 50 KB, far below the 10 GB cache limit per repository.
