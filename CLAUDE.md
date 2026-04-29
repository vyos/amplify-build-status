# CLAUDE.md

## Project purpose

Standalone, third-party-style GitHub Action (Docker-based) that polls the AWS Amplify API for the build status of an app/branch/commit and optionally waits for it to finish before continuing the workflow. Used when a VyOS web property is built remotely on Amplify and downstream CI must gate on the Amplify build outcome.

## Tech stack

- Docker-image action (`runs.using: docker`, `image: Dockerfile`).
- `alpine:3.19` base + `aws-cli` + `jq`.
- Shell `entrypoint.sh`.

## Build / test / run

Used as a step inside a consumer workflow:

```yaml
- uses: vyos/amplify-build-status@v2.2
  with:
    app-id:      ${{ secrets.AMPLIFY_APP_ID }}
    branch-name: ${{ github.ref_name }}
    commit-id:   ${{ github.sha }}
    wait:        true
  env:
    AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    AWS_REGION:            ${{ secrets.AWS_REGION }}
```

To test locally: `docker build -t amplify-status .` then run `docker run amplify-status <app-id> <branch> <commit> <wait> <timeout> <no-fail>` with AWS env vars set.

## Repository layout

- `action.yml` — action metadata (inputs/outputs/runs).
- `Dockerfile` — alpine + aws-cli + jq.
- `entrypoint.sh` — shell logic that calls `aws amplify` and parses with `jq`.
- `LICENSE` (MIT), `README.md`.

## Cross-repo context

Generic GHA action — not part of the VyOS image build pipeline. Likely consumed by `sentrium/*` Next.js sites or similar Amplify-hosted properties. No mirror twin; not referenced by `vyos-build-packages/repos.toml`.

## Conventions

- Commit / PR title format: `component: T12345: description` (Phorge task ID mandatory). Enforced by `vyos/.github` reusable workflows where consumed.
- Branch model: `current` (rolling), `circinus` (1.5 LTS), `sagitta` (1.4 LTS), `equuleus` (1.3 LTS).
- Released by tag (`v1`, `v1.1`, `v2.0`, `v2.1`, `v2.2`); consumers pin `@vX.Y` rather than `@current`.

## Mirror relationship

No mirror twin.

## Notes for future contributors

- Likely a fork or rewrite of `duckbytes/amplify-build-status` (README example references that source).
- License: MIT.
- AWS credentials are passed via env from the calling workflow — never bake them into the image.
