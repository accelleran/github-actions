# github-actions

Shared GitHub Actions workflows for Accelleran repositories.

## Temporary core-network images

`publish-core-image.yml` publishes an immutable image to:

```text
ghcr.io/accelleran/<package>:YYYYMMDD-HHMMSS-<12-character-commit-sha>
```

Callers must trigger it only for tag pushes or pushes to their relevant
`master` or `main` branch. Pull requests and feature-branch pushes must
remain build-only.

`cleanup-core-images.yml` deletes versions whose only tags match that format
once their GitHub package creation time is more than 30 days old. Call it
after a successful publication and from a daily scheduled workflow.

Caller repositories require:

- `contents: read` and `packages: write` workflow permissions.
- A `CN5G_READ_TOKEN` secret when the Docker build needs private sibling
  repositories such as `cn5g-common-src`, `cn5g-common-build`,
  `cn5g-common-ci`, `fcaps-common-src`, or `ogscrypt`.
- Access to reusable workflows from this repository in the organization
  Actions settings.
- The generated GHCR package linked to its source repository with Actions
  write/admin access so its `GITHUB_TOKEN` can delete expired versions.
  Cleanup fails on Packages API `404` rather than treating it as an empty
  package; a missing, misspelled, or unlinked package must not look like a
  successful retention run.

The cleanup workflow deliberately ignores untagged versions and versions with
non-managed tags. Publishing disables BuildKit provenance and SBOM manifests
so temporary packages do not accumulate untagged child manifests.
