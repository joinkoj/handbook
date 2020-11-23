---
title: Fast GitHub Actions deployments for emergencies
author: anand
date: 2020-09-28T11:44:46.102Z
---

We use Sapper and Svelte for the Koj website, and we have a pretty comprehensive CI/CD build process. The process includes running unit tests, Cypress end-to-end tests, Lighthouse audits, Semantic Release versioning, and finally exporting a static version of the site which is then published.

This, however, means that the entire CI flow from commit push to deployment takes between 3 and 4.5 minutes, which might be too late for emergency deployments for important bug fixes. For this, we made a “fast” deployment process that skips all tests and just exports, purges, and deploys. This takes less than a minute, but of course is dangerous and should only be used in exceptional circumstances (luckily, we haven’t had the need for this yet).

For the Fast Deploy GitHub Actions workflow, we make sure that the last commit message includes the text `[fast]`, just like you’d check for the text `[skip ci]` to skip the process altogether. This is a separate workflow that only runs when the `[fast]` flag is present in a commit, so it’s skipped in almost all pushes.

```yaml
name: Fast Deploy
on:
  push:
    branches:
      - master
jobs:
  release:
    name: Deploy
    runs-on: ubuntu-18.04
    if: "contains(github.event.head_commit.message, '[fast]')"
    steps:
      - name: Checkout
        uses: actions/checkout@v1
      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: 12
      - name: Install dependencies
        run: npm ci
      - name: Build static site
        run: npm run export
      - name: GitHub Pages Deploy
        uses: maxheld83/ghpages@v0.2.1
        env:
          BUILD_DIR: "__sapper__/export/"
          GH_PAT: ${{ secrets.GH_PAT }}
      - name: Purge Cloudflare cache
        uses: jakejarvis/cloudflare-purge-action@master
        env:
          CLOUDFLARE_TOKEN: ${{ CLOUDFLARE_TOKEN }}
          CLOUDFLARE_ZONE: ${{ CLOUDFLARE_ZONE }}
```

In this workflow, we only checkout the repository, install Node.js and then the dependencies, and build the static site. Then, we push it to the `gh-pages` branch and deploy that branch on our CDN, while force-purging its cache to make sure all users will see the most recent version.

Of course, this is not recommended for day-to-day use and only for those specific emergencies when you need to push your code really quickly.
