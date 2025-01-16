---
title: Publishing a Hugo Static Site with GitHub Pages
description: ""
date: 2025-01-16T03:12:47.908Z
preview: ""
draft: false
tags:
    - deployment automation
    - GitHub Actions
    - GitHub Pages
    - Hugo
    - personal website
    - static site generation
categories: []
slug: publishing-hugo-static-site-github-pages
fmContentType: blog
---

<!-- # Outline
1. Building Hugo Sites
2. How github pages works
3. Integrating hugo steps with gh actions 
-->

One of my motivations for starting a personal site was developing my technical (and non-technical) writing. With that in mind, what's a better starting point than writing about how the site is built and hosted? In this post I’ll go over how to start a Hugo site, how GitHub Pages works, and how to automate deployment with GitHub Actions.

# Building Hugo Sites
<!-- - need to install hugo (and have git)
- `hugo new site xyz`
    - this bootstraps a hugo project locally (directory structure mainly)
- themes (get up and running)
    - papermod seems like the most starred
- first post, with an image
- building and viewing it locally

--- -->

Hugo provides a convenient command to get your site project started. However, you must install Hugo on your machine first. Hugo is distributed as a [prebuilt binary which can be manually installed](https://gohugo.io/installation/macos/#prebuilt-binaries), or more easily, installed with homebrew (on macOS and Linux).
```shell
brew install hugo
```
Your installation can be confirmed with:
```shell
hugo version
# example output 
# hugo v0.140.0+extended+withdeploy darwin/arm64 BuildDate=2024-12-17T14:20:55Z VendorInfo=brew
```
Note: You may see a different version depending on when you installed hugo

Git is also required, but if you're reading this post I'm hoping you already have that configured on your system.


To bootstrap your site you can run the following, which will create a set of directories. Using your github username in the directory will help later when we need to initialize a git repo for your site.

```shell
hugo new site <your-github-username.github.io>
```

```shell
my-site/
├── archetypes/ # starter templates for content
│   └── default.md
├── assets/ # preprocessed files (SCSS, Javascript)
├── content/ # where you markdown pages are stored 
├── data/ # data files for templates
├── hugo.toml # configuration file for your Hugo Project
├── i18n/ # translation files
├── layouts/ # html templates, used to render pages
├── static/ # stores things like images, logos, favicons, etc.
└── themes/ # themes you install end up here
```

Each of these directories plays a role in your hugo site, but we will only focus on a few of them. Let's try creating some new content.

```shell
hugo new content hello.md
```

This creates a new markdown file `hello.md` under the `content` directory. When you open it you'll notice it's not empty, it starts out with some metadata important for Hugo. 

```markdown
+++
date = '2025-01-04T10:51:50-06:00'
draft = true
title = 'Hello'
+++
```

Let's try adding some text and rendering the page locally. Add something like `# Hello World!` to the bottom of the file and switch `draft = true` to `draft = false`. Now we can run `hugo server` to build your site (check the `public/` directory for the build output!) and start the development server.

With `hugo server` running in the background, navigate to `http://localhost:1313/hello` to view our newly created page!
It can also be useful to add the `-D` flag, i.e. `hugo server -D`, which will include "drafts" in your local build


Unfortunately, instead of our page rendering at this point, we'll see a "Page Not Found" error.. Why is that? It turns out we skipped an important step in the process by not setting up a theme. Themes provide hugo with templates to use when rendering your pages. There are hundreds of [themes](https://themes.gohugo.io/) to choose from, but for now we'll use [Ananke](https://github.com/theNewDynamic/gohugo-theme-ananke). To install it simply run the following:

```shell
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo "theme = 'ananke'" >> hugo.toml
```

The first command sets up the theme as a git submodule, which is a very common way to use hugo themes, and the second configures your site to use the newly added theme. Now try running `hugo server` once more and voila! your pages will be there. At this point, you're all set to add pages, images, links to your socials, etc. but your site will only be available to you on your local machine until we deploy it to the internet.

# A Brief Introduction to GitHub Pages
GitHub Pages is a free service provided by GitHub for all accounts to host and serve static content on the public internet. Pages can serve content from a few different sources:
- A repo named `username.github.io` - what we are doing
- A `/docs` folder in any repository
- A specific branch (`gh-pages`) in any repository

The latter two are useful for project documentation, while the first option typically makes the most sense for a personal website. By default, GitHub promotes using Jekyll as your static site generator. However, you can also use GitHub Actions to build and deploy from many different static site generators, including Hugo!

# Deploy Your Site with GitHub Pages 
Now that you've got a local hugo site and understand the basics of Pages, you are all ready to publish your site! To start, your project needs to be in a git repo and pushed to GitHub. If you haven't initialized git in your project directory, run `git init`. Then [create a new repo on GitHub](https://github.com/new) named `your-username.github.io`. Follow the instructions from GitHub to push your local project up. 


Your site files are now up in GitHub but GitHub Pages is not yet set up to serve content from your site. If you remember from the first step, Hugo needs to build a site based on your content, templates, etc. before it can be served with a web server. We’ll rely on GitHub Actions to build and deploy automatically whenever we push changes to the main branch.


Create a file at the `.github/workflows` path in your project directory titled `hugo.yaml`. This will be the specification for your GitHub Actions workflow. Hugo's docs [provide a complete spec](https://gohugo.io/hosting-and-deployment/hosting-on-github/#procedure) you can use for your site, which I have attached, annotated and slightly modified:

## Setup
```yaml
# A name for your workflow, this can be anything you want
name: Deploy Hugo site to Pages

# Configure when to run this workflow
on:
  push:
    branches:
      - main

  # Lets you manually trigger the workflow from GitHub's API, CLI or web browser
  workflow_dispatch:

# Sets permissions for the github token that gets used in each workflow run
permissions:
  contents: read # read contents of repo
  pages: write # write to github pages (i.e. deploy, build)
  id-token: write # necessary to get token, essentially verifies origin of deployment is authorized to do so

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash shell in workflow steps
defaults:
  run:
    shell: bash

```

The above is all setup and config for the workflow. You'll notice you can configure when the workflow runs with 
```yaml 
on:
  push:
    branches:
      - main
```
If, for example you wanted to run only on Release Tag events you could use 
```yaml
on:
  release:
    types: [released]
```

Hugo already has a concept of unpublished draft posts, so in most cases it makes sense to just deploy on each push to your main branch. 

## Build site
```yaml
jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.137.1
    steps:
    # Install Hugo in the workflow runtime environment
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
    # Install a style compiler (not strictly required but doesn't hurt)
      - name: Install Dart Sass
        run: sudo snap install dart-sass
    # Run a reusable action to checkout your repo for use later in the workflow
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
    # Run a resuable action that configures GitHub Pages
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
    # Javascript can be used together with Hugo for advanced usecases
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
    # Actually build the site and place it on the filesystem of the workflow
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Chicago
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/" 
    # Pushes the site files up to an ephemeral location for deployment
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
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
    # Deploy the compiled site to GitHub Pages
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

Commit and push this complete yaml and your workflow will trigger! Once it's done, your site will be published on the public internet at <your-github-username>.github.io, congratulations! From here you can work on more content, add a home page, customize your theme, and more. 

# Useful Resources
[Hugo Docs](https://gohugo.io/documentation/)
[GitHub Pages Docs](https://docs.github.com/en/pages)
[Guide to Hugo Deployment](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
[Hugo Themes Gallery](https://themes.gohugo.io/)
