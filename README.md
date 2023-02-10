# cidocs
Repo for documentation for CI Fuzz

## Local Setup
In general, it is recommended to install jekyll locally for easier testing and updating prior to pushing any changes.

[Install prequisites](https://jekyllrb.com/docs/installation/ubuntu/)

Once you have installed jekyll and bundler, navigate to the repo directory and run `bundle install`. This will install the needed gems based on the `gemfile.lock` file.

For testing, execute `bundle exec jekyll serve --livereload` in a terminal and then open a browser to http://localhost:4000

## Updating this repo
This repo is automatically built on new commits using Jekyll. This page is currently (and temporarily) served at cidocs.netlify.app. 
This repo uses the just-the-docs theme. This theme provides various capabilities. These are described [here](https://just-the-docs.github.io/just-the-docs/).

Currently, all updates need to occur through a pull request. Please create a new branch for each PR.

The core documentation exists in the top level docs folder. All pages should be created using standard markdown. You can include html, css, and javascript, but stick to markdown as much as possible. Netlify will convert the markdown pages to .html and apply theme/layout information automatically.

Each page has a frontmatter section that identifies the page, where it appears in the navigation pane, and can modify it's behavior.

Here is an example:

```yaml
---
layout: default
title: Viewing Logs
parent: Usage
grand_parent: CI Fuzz
nav_order: 4
permalink: viewing-logs
---
```

More detail on [front matter](https://jekyllrb.com/docs/front-matter/)

