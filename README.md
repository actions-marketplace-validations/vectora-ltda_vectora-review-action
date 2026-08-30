# Vectora Bot — PR Review Action

Automatic pull request review powered by [Vectora](https://vectora.company). Requires a **Vectora Pro** account.

## Setup

1. Sign in to [bot.vectora.company](https://bot.vectora.company) (same account as your Vectora Pro subscription).
2. Configure your LLM provider, model, and API key.
3. Generate a `VECTORA_BOT_TOKEN` — it's shown once, copy it now.
4. In your repository, create a **GitHub Environment** named `vectora-bot` (Settings → Environments) and register the token as that environment's `VECTORA_BOT_TOKEN` secret.
5. Add the workflow below as `.github/workflows/vectora.yml`.

```yaml
name: Vectora

on:
  pull_request:
    types: [opened, synchronize, ready_for_review]

permissions:
  pull-requests: write
  contents: read

concurrency:
  group: vectora-review-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  review:
    runs-on: ubuntu-latest
    environment: vectora-bot
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0

      - uses: brunosrz/vectora-review-action@v1
        with:
          token: ${{ secrets.VECTORA_BOT_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

That's it — every new PR (and every push to an open PR) gets a review comment. The `concurrency` group above cancels an in-progress review when a new push arrives, so a rapid sequence of commits doesn't queue up multiple runs.

## How it works

1. The action authenticates to the Vectora Bot panel with your `VECTORA_BOT_TOKEN` and fetches your configured provider, model, API key and review style — nothing is hardcoded in the action itself, so changing providers in the panel never requires touching any repository that has the bot installed.
2. It downloads the `vectora-cli` headless binary (no Electron/GUI, Linux x64) from the same channel Vectora's desktop app uses to update.
3. It runs `vectora-cli run` with the PR diff as context, using your own LLM API key.
4. It posts the result as a PR comment via the workflow's own `GITHUB_TOKEN` — no extra personal access token needed. On a later push, it **edits its previous comment** in place instead of posting a new one, so the PR doesn't get spammed by one comment per commit.

By default, review is skipped on draft PRs, and on any PR whose title contains `[skip vectora]`.

## Inputs

| Name | Required | Description |
|---|---|---|
| `token` | yes | `VECTORA_BOT_TOKEN` generated at bot.vectora.company |
| `github-token` | yes | Token used to post the review comment (usually `secrets.GITHUB_TOKEN`) |
| `panel-url` | no | Base URL of the Vectora Bot panel/API (default `https://bot.vectora.company`) |
| `skip-draft` | no | Skip review while the PR is a draft (default `true`) |
| `skip-marker` | no | Skip review when the PR title contains this text (default `[skip vectora]`) |

## License

MIT
