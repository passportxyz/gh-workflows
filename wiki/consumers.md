# Consumers

Nothing in this repo declares its consumers; the list below comes from grepping local clones of passportxyz repos. Verify with an org-wide code search before breaking changes.

## Known consumers and pins

- **passport** — load_commit_ref, prepare_deploy_to_aws, build_and_push, push_to_branch mostly `@v3`; `deploy_generic.yml` and `infra.yml` use deploy_to_aws`@v4` + build_and_push`@v5` (OIDC/S3-backend paths)
- **passport-scorer** — same pattern as passport; also the only user of legacy push_docker_image (`build_and_deploy_generic.yml`)
- **Passport-Data-Science** — `release_and_deploy_generic.yml`

The `@v4`/`@v5` pins track the Pulumi Cloud → S3+OIDC state migration: old static-key workflows stay on `@v3`, migrated ones move forward. Expect mixed pins per repo until the migration completes.

## Required consumer repo layout

- `infra/` — yarn workspace root with `infra/yarn.lock` (prepare_deploy_to_aws installs here)
- `infra/aws/` — Pulumi project (deploy_to_aws hardcodes this work-dir)
- Pulumi stacks named `staging`/`production`, pre-created in `s3://passportxyz-pulumi-state` (`upsert: false` — deploy won't create them)
- For `@v4+`: per-repo OIDC IAM roles passed via `aws_role_arn` (e.g. `pulumi-ci-<repo>-<env>`)
- Pulumi program reads the image tag from the `DOCKER_IMAGE_TAG` env var
