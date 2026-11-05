---
name: Cloudflare Workers Deployment
description: "Use when editing build.sh, wrangler.jsonc, package metadata, Git submodules, or the GitHub-to-Cloudflare Workers build and deployment configuration."
applyTo: "**/{build.sh,wrangler.jsonc,package.json,package-lock.json,.gitmodules}"
---
# Cloudflare Workers Deployment

- `wrangler.jsonc` is the project deployment configuration. Keep its custom build command, asset directory, `404-page` behavior, and custom domain aligned with the task; do not change production routing as incidental cleanup.
- Workers Builds invokes `chmod a+x build.sh && ./build.sh` in a Linux environment. `build.sh` is Bash-specific and is not the Windows local build command.
- The deployment script pins Dart Sass 1.107.1, Hugo Extended 0.169.0, Node.js 26.13.0, and Go 1.27.3 when `go.mod` exists. It initializes Git submodules, runs `npm ci` when `package-lock.json` exists, then runs `hugo build --gc --minify`. On Workers Builds branch `dev`, it uses `WORKERS_CI_BRANCH` to set the preview base URL to `https://dev.preview.vprovost.ca/`; production builds use the base URL in `hugo.toml`.
- Hugo writes deployable files to `public/`, which Wrangler publishes as static Worker assets. Never hand-edit generated files there.
- Keep the `themes/hermit-v2` submodule path casing consistent. A casing mismatch can pass on Windows and fail in Linux Builds.
- Worker Previews require Wrangler 4.135.0 or later. Keep the Wrangler dependency and lockfile committed; Workers Builds uses the project-local version. Verify dependency changes with a clean `npm ci`.
- Keep production `www.vprovost.ca` separate from the preview-only `preview.vprovost.ca`. Cloudflare prefixes the Preview name, so branch `dev` is served at `dev.preview.vprovost.ca`. Do not enable Preview traffic on the production hostname as a shortcut.
- Workers Builds uses `main` for production and runs its Preview command (normally `npx wrangler preview`) for non-production branches when Preview Builds are enabled. GitHub Actions is a build check only; do not add a second deploy workflow unless requested.
- Worker Previews do not inherit production runtime variables, secrets, or bindings. Configure only Preview-safe values in the Wrangler `previews` block or Previews Base; do not copy production credentials or point Preview bindings at production data without an explicit decision.
- `package.json` and `package-lock.json` must agree for `npm ci`. If they disagree, update the lockfile as part of an intentional dependency change and verify with a clean `npm ci`.
- Do not run `wrangler deploy`, alter Cloudflare dashboard settings, or change routes/secrets without explicit user authorization. For local site checks, use Hugo rather than the production deployment script.

## Worker Preview References

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

See [Cloudflare branch preview setup](../../docs/cloudflare-previews.md) for the repository-specific setup and troubleshooting steps.
