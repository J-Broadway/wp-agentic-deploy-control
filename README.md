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

- `PRODUCTION_SSH_PRIVATE_KEY`
- `CLOUDWAYS_API_TOKEN`
- A narrowly scoped package/source read credential only if explicit GHCR Actions
  access for this repository's `GITHUB_TOKEN` cannot be granted

Variables:

- `PRODUCTION_SSH_HOST`
- `PRODUCTION_SSH_USER`
- `PRODUCTION_SSH_PORT`
- `PRODUCTION_SSH_KNOWN_HOST_LINE`
- `PRODUCTION_SSH_HOST_KEY_FINGERPRINT`
- `PRODUCTION_ROOT`
- `APPLICATION_URL`
- `MEDIA_SMOKE_PATH`
- `CLOUDWAYS_SERVER_ID`
- `CLOUDWAYS_APP_ID`

The known-host line and SHA-256 fingerprint must be obtained through a trusted
channel. The workflow never runs `ssh-keyscan`.

## Operating rule

Review the private source diff from the last deployed commit, then manually run
the workflow from `main` with the exact candidate SHA and OCI digest recorded by
the private artifact publication workflow. Start only one candidate at a time;
GitHub concurrency prevents overlapping running deployments but is not a FIFO
queue for pending runs.
