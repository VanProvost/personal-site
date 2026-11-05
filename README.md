# vprovost

The source for [vprovost.ca](https://www.vprovost.ca), a personal site built with Hugo and the Hermit-V2 theme. The site is built from Markdown and published as static assets to Cloudflare Workers.

## Requirements

- Hugo Extended 0.169.0, matching the deployment build. The theme requires Hugo Extended 0.146.0 or later.
- Dart Sass 1.107.1 for the Sass pipeline.
- Git, with submodule support.

Clone the repository with its theme submodule:

```sh
git clone --recurse-submodules https://github.com/VanProvost/main-hugo-site.git
cd main-hugo-site
```

If you already cloned without submodules, initialize them with:

```sh
git submodule update --init --recursive
```

Make sure `hugo` and `sass` are available on your `PATH`. The repository's `build.sh` installs pinned versions in the Linux deployment environment; it is Bash-specific and should not be run on Windows.

## Local development

Start a local preview, including draft content:

```sh
hugo server -D
```

Run the production-style local build:

```sh
hugo --gc --minify
```

Hugo writes generated files to `public/`. Do not edit or commit generated output.

## Content and configuration

- `content/posts/`: blog posts.
- `content/about.md`: About page.
- `assets/identity/`: favicon and site identity assets.
- `layouts/`: project-level Hugo templates and shortcodes. Prefer these overrides over editing the theme.
- `hugo.toml`: site, navigation, and social-link configuration.
- `themes/hermit-v2/`: pinned theme submodule; do not edit its files as part of normal site work.

Create a post with:

```sh
hugo new posts/post-name.md
```

Review the generated front matter and set `draft: false` when the post is ready to publish. The About page and homepage share the configured social links: LinkedIn, GitHub, and email. Update these in `hugo.toml` if they change.

## Build and deployment

Cloudflare Workers Builds runs the build command configured in `wrangler.jsonc`, which invokes `build.sh`. The script downloads the pinned build tools, initializes the theme submodule, runs `npm ci` when a package lockfile is present, and builds the site with:

```sh
hugo build --gc --minify
```

Wrangler publishes the generated `public/` directory as static assets. Do not run `build.sh` as a Windows command, deploy manually, or change Cloudflare settings as part of local development.

## License

This project is licensed under the MIT License; see [LICENSE](LICENSE).
