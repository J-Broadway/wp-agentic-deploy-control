# WordPress production deploy control

Public, single-operator control plane for manual production deployments of the
private `J-Broadway/wp-agentic-test` website.

This repository contains no website source, artifacts, database data, uploads,
or production credentials. Direct pushes to `main` are intentionally allowed.
The production workflow accepts only a full source commit SHA and immutable OCI
digest; tags and branches are never deployment identities.

## Production Environment

Create a GitHub Environment named `production`, restrict deployment branches to
`main`, and configure:

Secrets:

- `PRODUCTION_SSH_PRIVATE_KEY` (dedicated app-user deploy key for `ggpjvxscfe`)
- `CLOUDWAYS_API_TOKEN`
- A narrowly scoped package/source read credential only if explicit GHCR Actions
  access for this repository's `GITHUB_TOKEN` cannot be granted

Variables:

- `PRODUCTION_SSH_HOST`
- `PRODUCTION_SSH_PORT`
- `PRODUCTION_SSH_KNOWN_HOST_LINE`
- `PRODUCTION_SSH_HOST_KEY_FINGERPRINT`
- `APPLICATION_URL`
- `MEDIA_SMOKE_PATH`
- `CLOUDWAYS_SERVER_ID`

Fixed in reviewed workflow code (not Environment inputs):

- OCI package `ghcr.io/j-broadway/wp-agentic-test-production`
- SSH user `ggpjvxscfe`
- Cloudways app ID `6588863`
- Canonical deploy root under the application `public_html`

The known-host line and SHA-256 fingerprint must be obtained through a trusted
channel. The workflow never runs `ssh-keyscan`.

## Private package access

GitHub’s package-repository link APIs return 404 for this user-owned private
container package, so Actions access must be granted in the UI:

1. Open
   `https://github.com/users/J-Broadway/packages/container/wp-agentic-test-production/settings`
2. Under **Manage Actions access**, click **Add repository**
3. Select `J-Broadway/wp-agentic-deploy-control`
4. Set role to **Read**

Until that grant exists, `oras pull` with the job `GITHUB_TOKEN` fails closed
(`denied` / `not found`). Do not substitute a broad personal access token.

## Operating rule

Review the private source diff from the last deployed commit, then manually run
the workflow from `main` with the exact candidate SHA and OCI digest recorded by
the private artifact publication workflow. Start only one candidate at a time;
GitHub concurrency prevents overlapping running deployments but is not a FIFO
queue for pending runs.

First-release or first-cutover failures with no recorded previous release require
maintenance-mode recovery and explicit webroot restoration; the workflow fails
closed instead of inventing a rollback target.
