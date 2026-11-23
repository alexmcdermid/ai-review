# ai-review

On-demand pull request reviews from Claude and GPT. Comment on a pull request:

```
/ai-review [light|standard|deep|security] [anthropic|openai|both]
```

Each tier trades API spend for depth. Models, prices and prompts live in `review-pull-request.mjs`. Each review is posted as a summary plus inline comments, with its token cost.

## Use it in a repo

Add the repository secrets `ANTHROPIC_REVIEWER_API_KEY` and `OPENAI_API_REVIEWER_KEY`. Then add `.github/workflows/ai-review.yml`:

```yaml
name: AI PR Review

on:
  issue_comment:
    types: [created]

permissions: {}

jobs:
  review:
    if: >-
      github.event.issue.pull_request
      && startsWith(github.event.comment.body, '/ai-review')
      && github.event.comment.author_association == 'OWNER'
    runs-on: ubuntu-24.04
    timeout-minutes: 60
    permissions:
      contents: read
      pull-requests: write
    concurrency:
      group: ai-review-${{ github.event.issue.number }}
      cancel-in-progress: false
    steps:
      - uses: alexmcdermid/ai-review@main
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_REVIEWER_API_KEY }}
          openai-api-key: ${{ secrets.OPENAI_API_REVIEWER_KEY }}
```

Reviews spend API credits, so the `if` condition limits who can trigger one. In an organization repo, where no commenter is `OWNER`, check `github.event.comment.user.id` instead, and pin the action to a commit SHA. Optional inputs `default-tier` (`standard`) and `default-providers` (`both`) apply when the comment names none.

Comment-triggered workflows run from the default branch, so a change to the caller workflow takes effect once it is merged.

## Changing models

Edit the tier's entry in `REVIEW_TIERS` in `review-pull-request.mjs` and commit to `main`. Callers on `@main` pick up the change on their next run. Callers pinned to a commit SHA need their pin bumped.

See [docs/architecture.md](docs/architecture.md) for how a run works, the tiers, and the security model.
