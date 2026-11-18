# renovate-config

Shared [Renovate presets](https://docs.renovatebot.com/config-presets/) for
Buildkite plugin repositories.

## Use the preset

Add `github>buildkite-plugins/renovate-config:automerge-buildkite` to the
`extends` array in your repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    "github>buildkite-plugins/renovate-config:automerge-buildkite"
  ]
}
```

If Renovate is already configured, keep the existing presets, settings, and
package rules, and add the shared preset reference to `extends`. There is no need
to change an existing `config:base` preset or Dependency Dashboard setting to
adopt this policy.

The reference loads [automerge-buildkite.json](automerge-buildkite.json) from this
repository's default branch. Changes to the preset take effect on subsequent
Renovate runs in repositories that extend it. Adoption is opt-in; this repository
does not automatically configure other repositories.

## Automerge policy

The preset contains two package rules:

| Dependencies | Matching | Updates eligible for automerge |
| --- | --- | --- |
| Buildkite plugins hosted in the `buildkite-plugins` GitHub organization | The `buildkite` manager and package names matching `buildkite-plugins/*-buildkite-plugin` | Minor and patch |
| The `buildkite/plugin-linter` and `buildkite/plugin-tester` Docker images | The `docker` datasource, with or without the `docker.io/` prefix | Minor and patch |

The plugin rule includes shorthand references such as `plugin-linter#v4.1.0`.
Renovate resolves these to a package name such as
`buildkite-plugins/plugin-linter-buildkite-plugin` before matching the rule.

The image rule applies across managers, including Dockerfiles and Docker Compose.
It does not match other `buildkite/*` images. The preset does not enable automerge
for major, digest, pin, or replacement updates, or for unrelated dependencies.
Other presets or repository-level package rules can change the effective policy.

The preset only changes automerge behavior for dependencies Renovate already
detects. It does not add dependency discovery for arbitrary image references in
shell commands or scripts. Those references may need a
[custom manager](https://docs.renovatebot.com/modules/manager/regex/).

### Checks and approvals

Both rules use `automergeType: "pr"` and `platformAutomerge: false`. Renovate
creates a pull request, waits for successful status checks, and performs the merge
itself on a later run. GitHub's native **Allow auto-merge** setting is not required
for this mode. Ensure the repository's CI runs on Renovate pull requests; the
preset does not create a CI workflow or bypass checks.

Required reviews and other branch protections still apply. This preset does not
approve pull requests or provide an approval bot. If a review is required,
automerge waits until the review requirement is satisfied. See
[Renovate's automerge guidance](https://docs.renovatebot.com/key-concepts/automerge/)
for details.

## Validate changes

Run Renovate's configuration validator before proposing changes:

```bash
npx --yes --package renovate renovate-config-validator --strict automerge-buildkite.json
```
