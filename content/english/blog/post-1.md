---
title: "How to Deploy a Static Website with Hugo and GitHub Pages"
meta_title: ""
description: "A comprehensive guide on how to create and deploy a static website using Hugo and GitHub Pages"
date: 2024-12-22T05:00:00Z
image: "/images/blog.png"
categories: ["Technology", "Data"]
author: "yaruyng"
tags: ["technology", "tailwind"]
draft: false
---
How to Deploy a Static Website with Hugo and GitHub Pages

Introduction

In this guide, I'll walk you through the process of creating and deploying a static website using Hugo and GitHub Pages. Hugo is a fast and modern static site generator written in Go, and GitHub Pages provides free hosting for static websites.

## Prerequisites

Before we begin, make sure you have:

- Git installed on your computer
- A GitHub account
- Basic knowledge of command line operations
- Go built Hugo from source

## Step 1: Install Hugo

### For ubuntu users :
Before we begin,we should check the architecture og Ubuntu machine to ensure we download the correct package.
```bash
uname -i
```
if you see the following, it indicates that you are running a 64-bit Ubuntu installation.
```bash
x86_64
```
Vist the Hugo release page to find the last version of Hugo and copy the url.

Hugo is available in three editions:standard,extended,and extended/deploy.While the standard edition provides core functionality,the extended and extended/deploy editions offer advanced features.Many modern Hugo themes require the Extended edition,so I recommend using this edition.
```bash
cd ~
wget https://github.com/spf13/hugo/releases/download/v0.140.1/hugo-extended_0.140.1_amd64.deb
```
Now,you can install the dpkg by entering the following content.
```bash
sudo dpkg -i hugo*.deb
```
### Verify the installation:
```bash
hugo version
```
## Step 2: Fork a Theme
### For this example, we'll use the popular PaperMod theme:
You can directly fork the theme repository and modify the repository name to **yourgithubusername.github.io**
## Step 2: Customize a Theme

```bash
git clone https://github.com/yourgithubusername/yourgithubusername.github.io.git
```
### Update your config.toml file:
```toml
baseURL = 'https://yourusername.github.io/'
languageCode = 'en-us'
title = 'My Blog'
theme = 'PaperMod'
```
## Step 4: Create Content
### Create your first post:
You can modify the template in the theme or create your own md file.
```bash
vim content/english/blog/post-1.md
```
###Edit the created file at content/english/blog/post-1.md:

```markdown
---
title: "My First Post"
date: 2024-12-26
draft: false
---

This is my first blog post using Hugo!
```
## Step 5: Test Locally
### Run the Hugo server to preview your site:
```bash
hugo server -D
```
Visit http://localhost:1313 in your browser to see your site.

## Step 6: Configure GitHub Pages

Create a .github/workflows/hugo.yml file in your project:
```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

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
    env:
      HUGO_VERSION: 0.137.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"          
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
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
### Setup Github
Visit your GitHub repository. From the main menu choose Settings > Pages. In the center of your screen you will see this:

![img.png](/images/post1_pages.png)

From the main menu choose Settings > Action > General.In the bottom of your screen you will see this:

![img.png](/images/post1_workflowpermissions.png)
## Step 7: Deploy
### Push your code to GitHub:

```bash
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/yourusername.github.io.git
git push -u origin main
```

## Final Steps and Verification
1. Wait a few minutes for GitHub Actions to complete the deployment
2. Visit https://yourusername.github.io to see your live site
3. Any future pushes to the main branch will automatically trigger a new deployment
## Common Issues and Solutions
1. 404 Error

  -Check if your baseURL in config.toml matches your GitHub Pages URL
  -Ensure your repository name follows the format: username.github.io
2. Missing Theme

  -Verify that the theme is properly initialized as a submodule
  -Check if the theme name in config.toml matches the theme directory name
3. Build Failures

  -Check GitHub Actions logs for specific error messages
  -Ensure all dependencies are properly specified in the workflow file
### Conclusion
You now have a fully functional static website powered by Hugo and hosted on GitHub Pages! This setup provides a robust foundation for blogging, documentation, or any other static website needs.

### Next Steps
-Customize your theme

-Add more content

-Configure custom domain

-Enable comments using Disqus or other platforms

-Add analytics tracking

-Remember to regularly commit and push your changes to keep your site updated.

Happy blogging! 🚀
