# setup-csph

Install the [ComputeSphere](https://computesphere.com) CLI (`csph`) in GitHub
Actions — version-pinned, **checksum-verified**, cached on the runner, and added
to `PATH`.

```yaml
steps:
  - uses: computesphere/setup-csph@v1
    with:
      version: 0.12.1                                  # pin for reproducible CI
      token: ${{ secrets.COMPUTESPHERE_API_TOKEN }}    # optional — auths csph
  - run: csph deploy --file computesphere.yaml
```

Security-conscious? Pin the action to a full commit SHA (Dependabot/Renovate
keep it fresh):

```yaml
  - uses: computesphere/setup-csph@<40-char-sha> # v1
```

## What it does

1. Resolves the version — an explicit `version`, else a `.csph-version` file in
   the repo, else the latest release.
2. Checks the runner tool cache; on a miss, downloads the release archive for
   the runner's OS/arch and **verifies its SHA-256 against the published
   checksums** before installing.
3. Puts `csph` on `PATH`, and (optionally) exports a masked
   `COMPUTESPHERE_API_TOKEN` so subsequent `csph` steps just work.

The same `csph` you run in your terminal is what runs in CI — no wrapper
semantics, no drift.

## Inputs

| Input | Description | Default |
|---|---|---|
| `version` | csph version to install (e.g. `0.12.1`) | `latest` |
| `version-file` | File holding the pinned version, used when `version` is `latest` | `.csph-version` |
| `token` | ComputeSphere API token; masked and exported as `COMPUTESPHERE_API_TOKEN` | — |

## Outputs

| Output | Description |
|---|---|
| `csph-version` | The resolved version that was installed |
| `cache-hit` | Whether the binary came from the runner tool cache |

## Examples

**Deploy a prebuilt image** (promotion by digest):

```yaml
  - uses: docker/build-push-action@v6
    id: build
    with: { push: true, tags: ghcr.io/acme/api:${{ github.sha }} }
  - uses: computesphere/setup-csph@v1
    with:
      token: ${{ secrets.COMPUTESPHERE_API_TOKEN }}
  - run: csph deploy --image ghcr.io/acme/api@${{ steps.build.outputs.digest }}
```

**Matrix over CLI versions:**

```yaml
strategy:
  matrix:
    csph: ["0.12.1", "latest"]
steps:
  - uses: computesphere/setup-csph@v1
    with:
      version: ${{ matrix.csph }}
  - run: csph --version
```

**Track a repo-pinned version** — commit a `.csph-version` file containing e.g.
`0.12.1`, and the default configuration picks it up.

## Platforms

Linux, macOS, and Windows runners; `amd64` and `arm64` (plus `386`/`arm` on
Linux). Binaries come from the public
[csph releases](https://github.com/computesphere/csph/releases).

## Documentation

- [ComputeSphere CLI docs](https://docs.computesphere.com/docs/cli/getting-started)
- [ComputeSphere](https://computesphere.com)
