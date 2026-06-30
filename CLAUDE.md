# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This repo contains a single static HTML landing page (`index.html`) for "AI Workspace" — a Chinese-language (zh-CN) marketing site. There is no build system, package manager, bundler, test suite, or linter configured. The page is plain HTML with all CSS inline in a `<style>` block and no JavaScript.

## Development workflow

There are no build, lint, or test commands — none are configured in this repo. To preview changes, open `index.html` directly in a browser or serve the directory with any static file server (e.g. `python3 -m http.server`).

## Structure

- `index.html` — the entire site: navbar, hero section, features grid, CTA section, and footer, in that order, with matching CSS rules grouped by section under comments like `/* 导航栏 */` (navbar), `/* Hero 区域 */`, `/* 功能区域 */` (features), `/* CTA 区域 */`, `/* 页脚 */` (footer).
- `README.md` — single line, just the project title.

## Conventions

- Content and UI copy are in Simplified Chinese (`lang="zh-CN"`); keep new copy consistent with this unless told otherwise.
- Section CSS is grouped directly under a Chinese comment header naming that section — follow this pattern when adding new sections rather than splitting styles into separate files.
- A `@media (max-width: 968px)` block at the end of the `<style>` tag handles all responsive overrides; add new responsive rules there rather than inline per-component.
- Color palette: dark background (`#0f1419`, `#1a1f2e`, `#0a0d12`), white text at varying opacity, and a purple gradient accent (`#667eea` → `#764ba2`) used for primary buttons, icons, and highlighted text.
