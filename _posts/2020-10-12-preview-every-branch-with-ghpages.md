---
layout: post
title: Deploy previews of branches to gh-pages 
categories: automation
image: /assets/posts/2020-10-12/drew-graham-FK0RhfEeY0w-unsplash.jpg
tags: testing, gh-pages, ci, preview, workflows
author: Hannes Diercks
imageBy: '<span>Photo by <a href="https://unsplash.com/@dizzyd718?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Drew Graham</a> on <a href="https://unsplash.com/s/photos/parallel?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>'
cta: <li>Hire me to build or improve your continuous deployment or recommend me</li>
description: Learn to automatically get previews pages of all your work in progress
---

I love [GitHub Pages](https://pages.github.com/) for it's simplicity.  
I use it for simple websites and blogs (like [this page](https://github.com/Xiphe/xiphe.github.io)). It works great for style-guides and component libraries that I build for my clients.
And it's also great for single page applications (SPAs). 

No additional services needed, just works, awesome!

&nbsp;

## The Problem

Normally only one Source (`branch` or `directory`) is deployed to `production`.  
But when you work with a team you might want to use feature branches for work in progress or spikes.

Wouldn't it be great when these branches would also be deployed as a preview so that I can show my WIP to colleagues and get feedback?

> Spoiler: The answer is "Yes of course!"

&nbsp;

## TL;DR

See [example-ghpage-feature-preview repo](https://github.com/Xiphe/example-ghpage-feature-preview/)
for the complete setup.

&nbsp;

## What we'll build

In this article I'll walk you through the setup of...

 - A GitHub repository 
 - with GitHub Actions
 - running a build-step (jekyll in this case) (could be anything generating a deployable page)
 - deploying it's production branch to `[GITHUB_USER].github.io/[PROJECT_NAME]` (Can be changed with CNAME)
 - creating previews for other branch on `[GITHUB_USER].github.io/[PROJECT_NAME]/preview/[branchname]`
 - cleaning up previews of deleted or merged branches

I assume you're familiar with [git](https://git-scm.com/), [GitHub](https://github.com/) and have a brief understanding of building software in continuous integration environments.

Whenever you see a `[YOUR_...]` notation, that's a placeholder that you should 
replace including the `[]` braces.

This example uses `main` as the production branch and `gh-pages` for GitHub Pages. It's not required to use these branch names.

&nbsp;

## Create the project locally and push it to GitHub

1. Create a new jekyll project:
  ```bash
  gem install bundler jekyll
  jekyll new [YOUR_PROJECT_NAME]
  cd [YOUR_PROJECT_NAME]
  git init
  git checkout -b main
  git add .
  git commit -m'initial commit'
  ```
2. Create a [new empty repository on github](https://github.com/new)
3. Copy the remote url 
4. Push the initial project to Github:
   ```bash
   git remote add origin [YOUR_REMOTE_URL]
   git push origin main -u
   ```
5. Also create a new empty branch that we'll use for github pages
   ```bash
   git checkout --orphan gh-pages
   git rm --cached -r .
   git commit --allow-empty -m'init'
   git clean -df
   git push origin gh-pages
   git checkout main
   ```
6. Enable GitHub pages for the `/ (root)` of `gh-pages` branch under the `Settings` of your repository
   
&nbsp;

## Create a github workflow to build the sites artifacts in CI

{% capture content %}

### Sidetrack: why a custom build?

You might now think: Aren't we creating a jekyll page? Why should we build it ourself instead of using the [builtin jekyll that GitHub is using for pages](https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/setting-up-a-github-pages-site-with-jekyll)?

1. In this article jekyll serves as a real-life placeholder for [hugo](https://gohugo.io/), [Next.js](https://nextjs.org/), or any other static page generator.
2. Even with jekyll, since we'll be hosting multiple versions of the page we need to build each one and move it to the right place on the `gh-pages` branch
3. In fact we'll [disable the default jekyll build](https://github.blog/2009-12-29-bypassing-jekyll-on-github-pages/)

&lt;/Sidetrack&gt;
{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

Add a `.github/workflows/deploy.yml` file to your project

```yml
name: deploy

on: push

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - uses: actions/setup-ruby@v1
        with:
          ruby-version: "2.x"

      - name: bundle install
        run: |
          bundle config path vendor/bundle
          bundle install --jobs 4 --retry 3

      - name: jekyll build
        run: |
          bundle exec jekyll build
          touch _site/.nojekyll
```

Once this is committed and pushed to github you should see a deploy workflow under 
the `Actions` tab of your repository. Currently it will only build the jekyll page
and do nothing with it...

&nbsp;

## Update gh-pages branch with our build artifacts

jekyll builds the page to a `_site` folder. This is the folder we 
want to move to `gh-pages` branch.

Add the following lines to `.github/workflows/deploy.yml`

{% capture content %}

```yml
# ... config to build your page to _site

      - uses: actions/setup-node@v1
        with:
          node-version: 12.x

      - name: deploy gh-pages
        env:
          GITHUB_TOKEN: {% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}
        run: |
          DEST=.
          git remote set-url origin https://git:${GITHUB_TOKEN}@github.com/[YOUR_GITHUB_USER]/[YOUR_REPO_NAME].git
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git config user.name "github-actions[bot]"
          npx gh-pages\
              --branch gh-pages\
              --dist _site\
              --dest $DEST\
              --add\
              --dotfiles
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

> Make sure to update `[YOUR_GITHUB_USER]` and `[YOUR_REPO_NAME]` to fit your setup.
> You can also use your email and user instead of the `github-actions[bot]`

Once this is running, it will put the latest build to `gh-pages` branch.
And you should be able to visit the deployed page to see your site.

&nbsp;

## Create previews for feature branches

At this point each commit will update your page. Also WIP-features
on a unmerged branch. Leading to some sort of race-condition.

In order to solve this we will put the artifacts of all branches except `main` 
to a `preview/[BRANCHNAME]` subfolder.

For this to work we need to update the build step to make the pages aware of their
new, nested location. This will cause internal links to point to `/preview/[branchname]/link`
instead of back to the production deploy of `main`.

{% capture content %}

```yml
      - name: jekyll build
        run: |
          BRANCH=${GITHUB_HEAD_REF##*/}
          PROD_URL=[YOUR_DEFAULT_BASE_URL]
          BASE_URL=$([ "$BRANCH" == "main" ] && echo $PROD_URL || echo "$PROD_URL/preview/$BRANCH")
          bundle exec jekyll build --baseurl $BASE_URL
          touch _site/.nojekyll
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

Next we configure the gh-pages deploy to move the `_site` folder to different destinations
based on the current branch.

{% capture content %}

```yml
      - name: deploy gh-pages
        env:
          GITHUB_TOKEN: {% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}
        run: |
          BRANCH=${GITHUB_HEAD_REF##*/}
          DEST=$([ "$BRANCH" == "main" ] && echo "." || echo "preview/$BRANCH")
          # git remote set-url ...
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

Now when you branch of from `main`, call it `test`, make some changes and push the `test` branch to GitHub, it will be deployed under `[GITHUB_USER].github.io/[PROJECT_NAME]/preview/test`

&nbsp;

## Clean up previews of merged or deleted branches

Now that this is working we might want to clean up old previews once the branch has 
been deleted or merged to production. 

For this we'll add a [custom `beforeAdd` script to gh-pages](https://github.com/tschaub/gh-pages#optionsbeforeadd) that checks our existing preview folders and removes those
that do not have a unmerged remote branch counterpart.

Replace the `npx gh-pages\ ...` call with the following:

{% capture content %}

```yml
          npm i cleanup-gh-pages-previews
          npx gh-pages\
              --branch gh-pages\
              --beforeAdd cleanup-gh-pages-previews\
              --dist _site\
              --dest $DEST\
              --add\
              --dotfiles
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

&nbsp;

## Bonus: Automatically add preview-links to pull requests

Now that everything works, wouldn't it be great to automatically add a link to 
Pull Requests?

> Memo to self: less rhetorical questions

We need to change the action so that it also runs on pull-requests to the `main` branch otherwise the PR-id can not be found. 
Also in order to not have duplicated runs we should also limit the builds on push to only `main`.

{% capture content %}

```yml
name: deploy

# on: push
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

# rest of the config ...
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

Now we can add another step at the bottom of the config

{% capture content %}

```yml
      - name: decorate PR
        if: github.event_name == 'pull_request'
        env:
          GITHUB_TOKEN: {% raw %}${{ secrets.GITHUB_TOKEN }}{% endraw %}
        run: |
          npx decorate-gh-pr -r -c "<a href=\"[YOUR_GITHUB_USER].github.io/[YOUR_PROJECT_NAME]/preview/${GITHUB_HEAD_REF##*/}\"><img src=\"https://img.shields.io/badge/published-gh--pages-green\" alt=\"published to gh-pages\" /></a><hr />"
```

{% endcapture %}

<div class="sidetrack">
{{ content | markdownify }}
</div>

&nbsp;

## And that's it!

See [`deploy.yml` of my example repository](https://github.com/Xiphe/example-ghpage-feature-preview/blob/main/.github/workflows/deploy.yml)
for the optimized version that includes caching of dependencies.
