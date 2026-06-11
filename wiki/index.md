# gh-workflows Wiki

Despite the repo name, this repo ships **composite actions only** (`.github/actions/*/action.yml`) — no reusable workflows. Consumers reference `passportxyz/gh-workflows/.github/actions/<name>@<tag>`.

- [Actions Catalog](actions.md) — the six composite actions: inputs/outputs, what each assumes about the caller (checkout, repo layout)
- [Versioning & Tags](versioning.md) — floating major tags v1–v5, what changed in each major, known tag inconsistencies
- [Consumers](consumers.md) — which passportxyz repos pin which versions, and the repo layout consumers must have
