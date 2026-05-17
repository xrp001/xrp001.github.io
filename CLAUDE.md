# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository is a GitHub Pages-compatible Jekyll static site for `xrp001.github.io`. It combines personal blog content with a vendored/customized Jekyll port of the NexT theme.

The site is driven by Ruby/Jekyll, not a Node-based build system. There is no `package.json`, Vite/Astro/Hugo config, or project-level JavaScript package workflow in the current tree.

## Common Commands

Run these from the repository root.

```sh
ruby --version
```

Use this to confirm Ruby is available. `README.en.md` documents Ruby 2.1.0+ for the original theme.

```sh
gem install bundler
bundle install
```

Install Ruby dependencies from `Gemfile`. The dependency stack is pinned through `github-pages` in `Gemfile.lock` and currently includes GitHub Pages 232 / Jekyll 3.10.0.

```sh
bundle exec jekyll serve --host 127.0.0.1 --port 4000
```

Serve the site locally at `http://127.0.0.1:4000`.

```sh
bundle exec jekyll build
```

Build the static site into `_site/`. Use this as the main verification command after content, template, or Sass changes.

No repository-specific test runner or linter is currently configured. There is no documented single-test command.

## Architecture

- `_config.yml` is the main site and theme configuration. It controls site metadata, language, permalink style, pagination, menu entries, NexT scheme selection, comments, analytics, search, MathJax, and other third-party integrations.
- `Gemfile` uses the `github-pages` gem group, so dependency changes should stay compatible with the GitHub Pages/Jekyll 3.x toolchain unless the deployment model is intentionally changed.
- `_posts/` contains the blog posts. Post filenames follow Jekyll date-prefix naming such as `YYYY-MM-DD-title.md`; post pages use the default `post` layout from `_config.yml` unless overridden in front matter.
- Top-level page directories such as `about/`, `archives/`, `categories/`, `tags/`, `category/`, and `tag/` provide site sections and route entry points.
- `_layouts/` contains thin layout wrappers. Most layouts assign language data and include templates from `_includes/`.
- `_includes/_layout.html` is the main HTML shell. It composes page title/class/sidebar blocks, head/header/footer partials, content, comments, third-party integrations, and scripts.
- `_includes/` is organized by role: `_partials` for common page regions, `_macro` for reusable post/sidebar fragments, `_helper` for Liquid helpers, `_blocks` for per-page computed variables, `_scripts` for script includes, and `_third-party` for integrations.
- `assets/css/main.scss` is the Sass entry point processed by Jekyll. It imports base variables/mixins, the configured NexT scheme from `site.scheme`, common styles, scheme styles, and `_sass/_custom/custom.scss` last.
- `_sass/` contains the NexT style layers: variables, mixins, common components, schemes, and custom overrides.
- `assets/js/src/` contains browser JavaScript source used by the theme, including bootstrap, motion, navigation, search, post details, and utility behavior. There is no bundler config in this repository, so avoid assuming an npm build step.
- `_data/languages/` stores translation strings keyed by `site.language`; this site currently uses `zh-Hans`.

## Editing Notes

- Prefer changing `_config.yml` for site/theme behavior before editing templates.
- For visual/theme changes, prefer `_sass/_custom/custom.scss` when a small override is enough; edit scheme/common Sass only when changing the theme implementation itself. Custom Sass is loaded after scheme styles and can override theme defaults such as Mist's hidden `.site-subtitle`.
- Keep content changes in `_posts/` or the relevant top-level page directory, and preserve Jekyll front matter delimiters.
- Generated output belongs in `_site/` and should not be committed.
- After changing content, Liquid templates, Sass, or configuration, run `bundle exec jekyll build` to catch build and Liquid errors.
