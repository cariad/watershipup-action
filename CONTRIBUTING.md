# Contributing

## Required tools

- [mise](https://mise.jdx.dev/)

## Setup

1. Clone this repository.

   ```bash
   git clone git@github.com:cariad/watershipup-action.git
   ```

1. Change into the repository's directory.

   ```bash
   cd watershipup-action
   ```

1. Install the project's tools.

   ```bash
   mise install
   ```

1. To verify the setup, run the linters and check they all pass.

   ```bash
   mise run lint
   ```

## Testing

CI's `test` job runs the action on every runner in its matrix. It publishes a page that only that job could have made to a real storage zone, and checks that the zone holds it. Then it changes the page, dry runs it with the zone's read-only password, and checks that the zone still holds the first page. That shows that the action installed WatershipUp, passed it every input, and passed `--dry-run` only when asked to, and that a dry run works with the read-only password.

The dry run also has a new page that only that job could have made. On a pull request, the job checks that the action's last two comments are the publish's report and then the dry run's, with that new page.

### Storage zone

The `test` job needs a bunny.net storage zone, which you must create and pay for yourself:

1. In the bunny.net dashboard, add a storage zone in Germany. The zone doesn't need replication or a pull zone.

   Each publish deletes the zone's pages that aren't in the test's directory, so don't use the zone for anything else. That includes WatershipUp's own tests, because jobs in different repositories can't take turns.

1. In this repository's *Settings* › *Environments*, create an environment named `integration-testing`.

1. Add a variable to the environment:
   - `WATERSHIPUP_TEST_ZONE`: the zone's name

   The zone's name is a variable, not a secret, because the action's comments on pull requests show it.

1. Add two secrets to the environment:
   - `WATERSHIPUP_TEST_PASSWORD`: the zone's read-write password
   - `WATERSHIPUP_TEST_READ_ONLY_PASSWORD`: the zone's read-only password

1. If Dependabot opens pull requests in the repository, add both secrets under *Settings* › *Secrets and variables* › *Dependabot* too, because Dependabot's pull requests can only read Dependabot secrets.

### Forks

GitHub doesn't run workflows in a fork until you turn them on in the *Actions* tab. CI then runs on pushes to `main` and on pull requests, and its `test` jobs fail until you set up a [storage zone](#storage-zone) for the fork.

GitHub doesn't give secrets to a pull request from a fork, so its `test` jobs fail, even when the fork has its own secrets. Before you open a pull request, push to your fork's `main` to run the tests against your own zone.

## Bumping versions

Dependabot bumps the actions in `action.yml` and `.github/workflows/`. Bump everything else by hand:

- **WatershipUp:** nothing to bump, because the action runs the latest release, and so does CI.
- **Python:** when WatershipUp needs a newer Python, `python-version` in `action.yml`.
- **mise:** `min_version` in `.config/mise.toml`, and `version` in the `jdx/mise-action` step in `.github/workflows/ci.yml`.
- **Tools:** `.config/mise.toml`, then run `mise lock`.
- **Runners:** the `test` matrix and the `lint` job's `runs-on` in `.github/workflows/ci.yml`.
