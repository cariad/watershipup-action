# watershipup-action

A GitHub Action that publishes a directory to a [bunny.net](https://bunny.net) storage zone with [WatershipUp](https://github.com/cariad/watershipup).

## Usage

To publish your site's `public` directory to the `example-zone` storage zone, with the zone's read-write password in the `STORAGE_ZONE_PASSWORD` secret:

```yaml
jobs:
  publish:
    runs-on: ubuntu-26.04

    concurrency:
      group: publish-example-zone

    permissions:
      contents: read

    steps:
      - uses: actions/checkout@<commit> # <release>
        with:
          persist-credentials: false

      # Build your site into `public` here.

      - uses: cariad/watershipup-action@<commit> # <release>
        with:
          directory: public
          zone: example-zone
          password: ${{ secrets.STORAGE_ZONE_PASSWORD }}
```

The action writes the files that it uploaded and deleted to the step's log and the job's summary. A report over GitHub's 1 MiB limit for a summary is only in the log.

WatershipUp can only publish to a storage zone whose main region is Germany.

## Dry runs

To see what a publish would upload and delete, without changing the zone, set `dry-run` to `true`. A dry run only reads the zone, so it can take the zone's read-only password:

```yaml
      - uses: cariad/watershipup-action@<commit> # <release>
        with:
          directory: public
          zone: example-zone
          password: ${{ secrets.STORAGE_ZONE_READ_ONLY_PASSWORD }}
          dry-run: true
```

## Pull requests

When the workflow runs for a pull request, the action also comments its report on the pull request, with a new comment for every run. A run that fails, times out or is cancelled comments a link to the workflow run instead, and so does a report over GitHub's 65,536-character limit for a comment.

The comments show the zone's name, and GitHub doesn't hide secrets in comments, so don't pass the zone from a secret.

To comment, the action uses the GitHub CLI, which GitHub's hosted runners have, with the job's `GITHUB_TOKEN`, which needs `pull-requests: write`. For example, to dry run every pull request:

```yaml
on:
  pull_request: {}

jobs:
  dry-run:
    runs-on: ubuntu-26.04

    concurrency:
      cancel-in-progress: true
      group: dry-run-${{ github.workflow }}-${{ github.event.pull_request.number }}

    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@<commit> # <release>
        with:
          persist-credentials: false

      # Build your site into `public` here.

      - uses: cariad/watershipup-action@<commit> # <release>
        with:
          directory: public
          zone: example-zone
          password: ${{ secrets.STORAGE_ZONE_READ_ONLY_PASSWORD }}
          dry-run: true
```

GitHub doesn't give a pull request from a fork your secrets, or a token that can comment, so the action can't dry run those. Dependabot's pull requests can only read Dependabot secrets, so to dry run those, add the read-only password under *Settings* › *Secrets and variables* › *Dependabot* too.

## Inputs

| Input                 | Required | Description                                                                                                 |
| --------------------- | -------- | -----------                                                                                                 |
| `directory`           | Yes      | The directory to publish.                                                                                   |
| `zone`                | Yes      | The name of the storage zone to publish to.                                                                 |
| `password`            | Yes      | The storage zone's read-write password, or its read-only password for a dry run. Pass it from a secret.     |
| `dry-run`             | No       | `true` to only show what a publish would upload and delete, without changing the zone. Defaults to `false`. |
| `watershipup-version` | No       | The version of WatershipUp to publish with. Defaults to the latest release.                                 |

## Security

The read-write password can upload and delete any file in the zone. Keep it in a secret, and never pass it to a job that runs code you don't trust, such as a pull request's build. The read-only password can't change the zone, so give dry runs that one instead.

Everything that WatershipUp publishes is public. It publishes every file in the directory except symbolic links, which it skips. If it finds a hidden file or directory, other than `.well-known` at the directory's root, it publishes nothing and the step fails.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) to set up the project.
