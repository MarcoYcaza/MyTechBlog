# Field Notes

A small technical knowledge base built with Hugo and PaperMod. Three posts marked **Sample** demonstrate the format; replace or delete them when ready. No database, CMS, analytics, or paid service is required.

## What's in the repository

- `hugo.yaml` — site title, description, navigation, search settings, and local base URL.
- `content/posts/<article>/index.md` — Markdown page bundles; keep images beside the article.
- `content/search.md` and `content/archives.md` — search and archive pages.
- `assets/` and `layouts/` — small customizations; upstream theme files are untouched.
- `themes/PaperMod/` — vendored PaperMod (commit `118119693d8b5ac211b939775678310973bcb232`, MIT license). It is committed with the site, so neither a submodule nor a separate theme download is needed. When updating it, review the overrides in `layouts/list.html`, `layouts/search.html`, and `assets/js/fastsearch.js` against upstream changes.
- `.github/workflows/deploy.yml` — the Pages build and deployment.

## Set up GitHub Pages

1. Create a **public** GitHub repository and push this project to its `main` branch. Use `USERNAME.github.io` for a user site, or any repository name for a project site. Replace `USERNAME` with your account name.
2. In the repository, open **Settings → Pages → Build and deployment → Source** and select **GitHub Actions**. Enable Actions if your repository or account settings disable them.
3. Push a change to `main` or run **Actions → Deploy Hugo to GitHub Pages → Run workflow**. The workflow installs Hugo Extended 0.158.0, builds the site, and publishes the artifact using the official Pages actions. Check the run's logs if a step fails.
4. Open the URL shown under Settings → Pages or in the deployment. A user site is normally `https://USERNAME.github.io/`; a project site is normally `https://USERNAME.github.io/REPOSITORY/`. Publishing can take a few minutes after the run starts.

The workflow obtains the actual base URL from GitHub Pages, including the project path or configured custom domain. No username or repository name is hardcoded. `baseURL` in `hugo.yaml` is for local use; for a manual production build, supply your actual URL, including its trailing slash:

```sh
hugo --minify --baseURL "https://USERNAME.github.io/REPOSITORY/"
```

Change `title` and `params.description` in `hugo.yaml` to personalize the site. Check Settings → Pages, the Actions run, and repository visibility if no site appears. A protected `github-pages` environment may require a reviewer before deployment. If the default branch is not `main`, update the workflow's branch filter.

## Write a post

Create `content/posts/my-topic/index.md`. Use the same YAML front matter as the sample posts:

````markdown
---
title: "My first technical note"
date: 2026-10-28
description: "A short description shown in the article list and search."
tags: [SQL, Performance]
categories: [Data Engineering]
draft: false
# lastmod: 2026-10-29
# cover:
#   image: "cover.png"
#   alt: "Description of the cover"
#   caption: "Optional caption"
---

## The problem

Write your explanation here. Use meaningful link text and descriptive image alt text.

![Query plan with two stages](query-plan.png "Optional caption")

```sql
SELECT * FROM orders;
```

```python
print("Hello, data")
```
````

Put `query-plan.png` beside `index.md`. Ordinary relative Markdown image paths work in page bundles and under project-site URL prefixes. The optional image title above is not rendered as a visible caption by default; use PaperMod's cover caption when a visible caption is needed. Set `draft: true` to keep a post out of production and search. Dates in the future also remain unpublished unless explicitly enabled. The table of contents comes from headings; set `ShowToc: false` in a short post's front matter if it is unnecessary. Code blocks support Hugo's Chroma languages, including SQL, Python, Bash, JSON, YAML, JavaScript, and Scala.

## Local development

Install [Hugo Extended](https://gohugo.io/installation/) 0.158.0, clone the repository, and open it in VS Code. Its Markdown preview works without starting the website. From the repository root:

```sh
hugo server -D
hugo --minify
```

Open the local URL printed by Hugo. `-D` includes drafts for preview; omit it to match production. The build writes `public/`, which is ignored by Git. Commit your source files and push to `main` to publish.

## Publish from a browser

1. Open the repository on GitHub and press `.` to open github.dev. It is a browser editor; it does not need local Hugo, Git, Node.js, or a terminal.
2. Create a folder under `content/posts/` and an `index.md` inside it. Copy the front matter above, set your title, description, date, tags and categories, and write the article. Use the editor's Markdown preview to check formatting; it is not a full Hugo website preview.
3. In github.dev, use Source Control to commit your Markdown changes to `main` (or commit on a branch and merge it). GitHub's built-in file editor is another option.
4. For images, return to GitHub's repository page and use **Add file → Upload files** in the now-committed article folder. Commit the upload. Reference each file by its exact name in the Markdown. Avoid spaces in filenames and include descriptive alt text. The article and image commits each trigger a build if made directly on `main`.
5. Check the **Actions** tab for a successful run. Visit the article, search for a distinctive phrase or tag, and check the relevant tag and category pages. Refresh after the deployment finishes.

## Search and troubleshooting

Hugo generates `index.json` at build time from published posts: title, description, full plain-text body, tags, and categories. PaperMod's bundled Fuse.js searches that local index in the browser, ranks titles more strongly, and tolerates modest spelling differences. Nothing is sent to a search service. JavaScript is required for search, but article navigation remains available without it.

If a result is missing, check `draft`, the post date, the `posts` folder, and `searchHidden` in front matter. Confirm the latest Actions run succeeded and the deployed `/index.json` contains the post (include the repository prefix for a project site). Refresh the page after deployment. If the search index fails to load, check the browser console and that Pages supplied the correct base URL. On a project site, test links, images, and search under the repository path rather than at the domain root.

For deployment errors, open the failed Actions run and expand the failed step. Check the Hugo version, YAML front matter, branch filter, Pages source, and workflow permissions. The workflow reports errors in its logs and does not publish a failed build.
