# Versioning & Tags

Floating major tags (`v1`–`v5`) moved by hand, sometimes with a semver tag alongside. The whole repo is versioned together — bumping for one action bumps the pin surface for all six.

## What each major means

| Tag | Commit | Contents |
|-----|--------|----------|
| v1 | `7f175f5` | Initial action set; yarn for prepare_deploy_to_aws |
| v2 | `39e24d6` | + optional `aws_session_token` on build_and_push |
| v3 | `e1cb3e2` | `--provenance=false` on docker build (no OCI index in ECR); setup-node@v4 |
| v4 / v4.0.0 | `8f32015` | deploy_to_aws: GitHub OIDC + S3 Pulumi state backend (`s3://passportxyz-pulumi-state`) |
| v4.0.1 | `f089990` | deploy_to_aws fix: pin pulumi/actions `cloud-url` to S3 backend; drop `-c` on stack select |
| v5 | `4dfb123` | build_and_push: OIDC for ECR push; `dockerfile_name` confirmed repo-root-relative (build refactored to single build+push step) |

## Known tag inconsistencies (verified)

- **`v4` was never advanced to `v4.0.1`** — anything pinned `@v4` is missing the S3-backend pin fix and may try to hit Pulumi Cloud. Pin `@v4.0.1` or `@v5`, or move the `v4` tag.
- `v5` has no companion semver tag (`v5.0.0` doesn't exist).
- `v2` (`39e24d6`) and `v3.0.0` (`a60e0b9`) point at pre-squash branch commits not on `main`; the floating tags are the source of truth, not the `vX.Y.Z` ones.
- `load_commit_ref` computes semver from `(MAJOR)`/`(MINOR)` commit-message markers for *consumer* repos — unrelated to this repo's own tags.
