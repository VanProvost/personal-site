# Cloudflare branch previews

This repository uses GitHub Actions to check the Hugo build and Cloudflare Workers Builds to publish it. Do not configure both systems to deploy the same commit.

## Repository configuration

- `main` is the production branch and serves `www.vprovost.ca`.
- `dev` is the shared preview branch.
- The preview-only custom domain is `preview.vprovost.ca`. Cloudflare prefixes the Preview name, so branch `dev` is served at `https://dev.preview.vprovost.ca`.
- `wrangler.jsonc` keeps production routing on `www.vprovost.ca`, marks `preview.vprovost.ca` as preview-only, enables `workers.dev` Preview URLs, and enables Preview observability.
- `package.json` and `package-lock.json` include the Wrangler version required for Worker Previews. Workers Builds uses the project-local Wrangler version.
- `build.sh` checks Cloudflare Workers Builds' `WORKERS_CI_BRANCH`. For branch `dev`, Hugo builds with `https://dev.preview.vprovost.ca/` as its base URL. Other branches use the production base URL in `hugo.toml`.
- The GitHub Actions workflow runs the build script as a check. It has no Cloudflare credentials and does not deploy.

## One-time setup

1. Push the repository configuration and workflow to GitHub. Confirm the Hugo CI check passes in the repository's **Actions** tab.
2. Create the `dev` branch from `main` if it does not exist.
3. In Cloudflare, open **Workers & Pages**, select the `main-hugo-site` Worker, then open **Settings > Build > Branch control**:
   - Set `main` as the production branch.
   - Enable **Preview Builds**.
   - Keep the Preview command as `npx wrangler preview`.
   - Keep the production deploy command as `npx wrangler deploy`.
4. If the existing Worker shows a **Set up Worker Previews** prompt, read the migration notice and confirm the switch only if you intend to move to the new Preview model. Cloudflare documents this one-time switch as irreversible.
5. Apply the Wrangler-managed custom-domain configuration through the normal Cloudflare Workers Build from `main`. If Cloudflare instead requires dashboard setup, add `preview.vprovost.ca` for **Preview** traffic only; do not enable Preview traffic on `www.vprovost.ca`.
6. Push a commit to `dev`. In Cloudflare, open the Worker's **Previews** area and use the latest `dev` Preview URL. Initial DNS and certificate provisioning can take time.
7. Optionally protect `dev` and `main` in GitHub and require the **Build Hugo site** check before merging pull requests.

The `dev` Preview is the shared preview. A pull request from another branch can create another branch-specific Preview, but this repository only sets the Hugo base URL override for `dev`; other branch Previews will generate absolute links using the production base URL.

## Preview safety and access

- Preview configuration does not inherit production runtime variables, secrets, or bindings. Add only settings needed by the site and use test resources for any future data bindings.
- Two Previews that use the same KV, D1, R2, Queue, or other account-level resource identifier share that resource. Use separate test resources when isolation matters. Durable Object namespaces are provisioned per Preview when configured as documented by Cloudflare.
- Preview URLs are public unless protected with Cloudflare Access. Cloudflare adds `X-Robots-Tag: noindex` to `workers.dev` Preview URLs, but not custom-domain Preview URLs. Protect a custom Preview hostname with Access if it should not be public.
- Use the Preview's **Observability** tab for logs and traces. Preview observability is configured separately from production.
- Do not use Version URLs for branch testing: they use the uploaded version's configured resources and do not provide branch-isolated Preview settings.

## Troubleshooting

- **A `dev` push updates production:** verify `main` is the configured production branch, Preview Builds are enabled, and Cloudflare's Preview command is `npx wrangler preview`.
- **Preview is missing or warns about bindings:** Preview settings do not inherit production. Add required, non-production settings to the `previews` block or Previews Base; do not reuse production data by default.
- **Preview domain does not resolve:** check that `preview.vprovost.ca` is attached for Preview-only traffic and allow time for wildcard DNS and certificate setup. Cloudflare Preview URLs add a label before the configured preview domain.
- **Preview links lead to production:** check the Workers Builds log for `WORKERS_CI_BRANCH=dev`, and verify generated HTML uses `https://dev.preview.vprovost.ca/`. Keep `public/` generated; do not edit it manually.
- **Wrangler dry run fails on Windows with `chmod` not found:** this repository's custom build hook is Linux/Bash-specific. Validate locally with `hugo --gc --minify`. If checking Wrangler configuration, use a temporary config with only the custom `build` hook removed; never commit that temporary config or use it to deploy.
- **`npm ci` fails:** verify `package.json` and `package-lock.json` are synchronized. Do not silently change unrelated dependency versions to repair lockfile drift.

## Official references

- [Workers Builds](https://developers.cloudflare.com/workers/ci-cd/builds/)
- [Build branches and Preview Builds](https://developers.cloudflare.com/workers/ci-cd/builds/build-branches/)
- [Worker Previews](https://developers.cloudflare.com/workers/previews/)
- [Preview configuration and Previews Base](https://developers.cloudflare.com/workers/previews/configuration/)
- [Preview resources and isolation](https://developers.cloudflare.com/workers/previews/resources/)
- [Preview custom domains](https://developers.cloudflare.com/workers/previews/custom-domains/)
- [Preview testing and observability](https://developers.cloudflare.com/workers/previews/test-and-debug/)
- [Previews compared with Version URLs and Wrangler environments](https://developers.cloudflare.com/workers/previews/compare-workflows/)
- [Preview examples for Workers Builds and GitHub Actions](https://developers.cloudflare.com/workers/previews/examples/)
- [Version URLs](https://developers.cloudflare.com/workers/versions-and-deployments/version-urls/)
