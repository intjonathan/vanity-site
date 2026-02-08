# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal Jekyll blog hosted on GitHub Pages at intjonathan.com. It uses the `no-style-please` theme via the `github-pages` gem.

## Development Commands

Ruby 3.4.7 is managed via mise. All commands should be run through mise:

```bash
# Install dependencies
mise exec -- bundle install

# Run local development server
mise exec -- bundle exec jekyll serve

# Build the site
mise exec -- bundle exec jekyll build
```

## Structure

- `_config.yml` - Jekyll configuration (theme, title, description)
- `_posts/` - Blog posts in `YYYY-MM-DD-title.md` format
- `index.md` - Homepage content
- `Gemfile` - Ruby dependencies (github-pages gem + plugins)

## Adding Blog Posts

Create new posts in `_posts/` with filename format `YYYY-MM-DD-title.md` and front matter:

```yaml
---
title: "Post Title"
date: YYYY-MM-DD
---
```
