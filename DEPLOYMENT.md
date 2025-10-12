# Deployment Guide

This blog uses a simplified single-branch deployment to GitHub Pages.

## How It Works

- **Single Branch**: All work happens on the `master` branch
- **Build Output**: The site builds to the `docs/` folder
- **GitHub Pages**: Serves directly from `master` branch's `/docs` folder
- **Automation**: GitHub Actions automatically rebuilds the site on every push

## One-Time GitHub Pages Setup

You need to configure GitHub Pages settings once:

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Pages**
3. Under "Build and deployment":
   - **Source**: Deploy from a branch
   - **Branch**: `master`
   - **Folder**: `/docs`
4. Click **Save**

## Daily Workflow

That's it! From now on, your workflow is simply:

```bash
# 1. Make changes (edit posts, content, etc.)
git add .
git commit -m "Your changes"

# 2. Push to master
git push origin master

# 3. Done! GitHub Actions will:
#    - Build the Hakyll site
#    - Update the docs/ folder
#    - Automatically deploy to GitHub Pages
```

## What Happens Automatically

When you push to `master`, the GitHub Actions workflow (`.github/workflows/build.yml`) automatically:

1. Sets up a Haskell environment
2. Builds the site executable from `site.hs`
3. Generates the static site into `docs/`
4. Commits the updated `docs/` folder back to `master`
5. GitHub Pages serves the updated site

## Manual Local Build (Optional)

If you want to preview locally:

```bash
# Build the site
stack build
stack exec site build

# Preview (if you have the preview command configured)
stack exec site watch
```

The generated site will be in the `docs/` directory.

## Notes

- The `site.hs` file is configured to output to `docs/` instead of the default `_site/`
- The old `hakyll` branch and `_site/` submodule setup is no longer needed
- The `circle.yml` CircleCI configuration is obsolete and can be removed
