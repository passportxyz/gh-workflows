# Actions Catalog

Six composite actions under `.github/actions/`. Contracts below are for current HEAD (v5); see [versioning.md](versioning.md) for older majors.

## build_and_push

Checkout + OIDC ECR login + docker build/push.

- Inputs: `refspec`, `docker_tag`, `ecr_repository_name`, `aws_region`, `aws_role_arn` (OIDC role); optional `dockerfile_name`, `build_dir`
- Does its own `actions/checkout` of `refspec` (fetch-depth 0) — caller need not checkout
- **`dockerfile_name` is repo-root-relative** (e.g. `./api/Dockerfile`), NOT joined with `build_dir` — deliberate, matches v3 semantics (fix #15, commit `4dfb123`). `build_dir` is only the docker context.
- Builds with `--provenance=false` — otherwise BuildKit pushes an OCI image index to ECR, which broke consumers (fix in #11)
- v3→v5 auth change: static `aws_access_key_id`/`aws_secret_access_key`(/`aws_session_token`) inputs replaced by `aws_role_arn` OIDC in v5

## deploy_to_aws

Pulumi preview/up against the S3 state backend via OIDC.

- Inputs: `docker_tag`, `stack_name` (S3-backend form: `staging`/`production`, no org prefix), `aws_region`, `aws_role_arn`, `pulumi_command` (default `preview`), `pulumi_diff` (default `"false"`), `OP_SERVICE_ACCOUNT_TOKEN` (required), `CLOUDFLARE_API_TOKEN` (optional)
- `pulumi login s3://passportxyz-pulumi-state`; `pulumi/actions@v5` with `cloud-url` pinned to the same S3 URL (without the pin it tries Pulumi Cloud — fix #13)
- `upsert: false` and `stack select` without `-c` — the stack must already exist in the S3 backend
- Hardcoded `work-dir: infra/aws`
- Does NOT checkout or install deps — caller must checkout and run `prepare_deploy_to_aws` first
- Exports `DOCKER_IMAGE_TAG` env to the Pulumi program
- v3 contract was different: `PULUMI_ACCESS_TOKEN` + static AWS keys, Pulumi Cloud backend, `stack select -c`

## prepare_deploy_to_aws

`setup-node@v4` (yarn cache keyed on `infra/yarn.lock`) + `yarn install` in `infra/`. Note the asymmetry: installs in `infra/`, but deploy_to_aws runs Pulumi in `infra/aws` — consumer repos use a yarn workspace rooted at `infra/`.

## load_commit_ref

Resolves a commit input into deploy metadata. Does its own checkout (fetch-depth 0).

- Input: `commit` (falls back to `github.sha` if empty)
- Outputs: `refspec` (resolved ref), `docker_tag` (short SHA), `version_tag` (from `paulhatch/semantic-version@v5.0.2`: tag prefix `v`, bumped by `(MAJOR)`/`(MINOR)` in commit messages)

## push_to_branch

Deploy-by-branch: **force-pushes** `commit` (default `origin/main`) onto `branch` (default `staging-app`) via `git reset --hard` + `push -f`. Embeds `github-token` in the remote URL. Output: `deployed-commit`. Assumes caller already checked out the repo.

## push_docker_image (legacy)

Push-only: expects an image already built and tagged locally as `<ecr_repository_name>:<docker_tag>`; retags to the ECR registry and pushes. Static AWS keys (`configure-aws-credentials@v1`), no OIDC. Superseded by build_and_push for new work.
