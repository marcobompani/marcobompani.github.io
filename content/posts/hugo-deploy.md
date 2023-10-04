---
title: "Deploy a Hugo Website with GitHub Actions"
date: 2023-10-03T23:42:03+01:00
draft: false
categories: ["Docs"]
comments:  false
tags:  ["Hugo", "GitHub", "GitHub Actions", "GitHub Pages"]
lightgallery: true
---
## Introduction
In this post we will create a website using [Hugo](https://gohugo.io/) and automatically deploy it on [GitHub Pages](https://pages.github.com/) using [Github Actions](https://github.com/features/actions).
## Hugo Installation
On Linux, you can simply install Hugo with:
```bash
sudo snap install hugo
```
If you are not a Linux user, follow the appropriate [documentation](https://gohugo.io/installation/) for your OS.
## GitHub Repository
On your GitHub account, create a new repository and call it **\<username\>.github.io**, e.g. **marcobompani.github.io**.
Make sure the repository is **Public**.
## Local Repository
Now we need to create a local git repository to store our website and push it to GitHub. In a terminal create a new folder (e.g. _website_), cd into it and run the following commands:
```bash
git init
git remote add origin git@github.com:<username>/<username>.github.io.git
hugo new site . --force
```
### Themes
You can also use one of the available [Hugo Themes](https://themes.gohugo.io/). If you do it, just add it as a git submodule with:
```bash
git submodule add https://github.com/panr/hugo-theme-hello-friend.git themes/hello-friend
```
Make sure to replace the URL and _themes/hello-friend_ accordingly.
## Push the Changes
After you have added some content to your website, you are ready to push the changes to GitHub with:
```bash
git add -A
git commit -m "some cool commit"
git push
```
How to create a website with Hugo is out of the scope of this post. If you do not know how to do it, a good place where to start is their [quick start guide](https://gohugo.io/getting-started/quick-start/).
## Build and Deployment
On your GitHub repository webpage:
1. click on **Settings**
2. click on **Pages**
3. select **GitHub Actions** as **Source**

![build and deploy](/img/build_deploy.png)

After selecting **GitHub Actions** as **Source**, click on **create your own**.

![workflow](/img/actions.png)

Finally, in the GitHub editor give the new file a name, e.g. _hugo.yml_ and paste the following content into it:
```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Install Hugo CLI
        run: sudo snap install hugo
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: hugo --minify
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```
