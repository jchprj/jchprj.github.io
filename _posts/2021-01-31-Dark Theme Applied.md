---
layout: post
title: Dark Theme Applied
date: 2021-01-31 12:27
category: jekyll
author: Jue Chen
tags: ["jekyll"]
summary: Applied latest Minima theme with dark skin.
comment_issue_id: 1
---

More than half a year since last post. I nearly forgot to see this site although it's just in the bookmark. And after these times, I'm much more familiar with GitHub now.

Then I see some security issue in the GitHub repository of this site. I upgraded Jekyll to latest to fix the security dependency issue. I also feel it's still not dark theme. I like the Minima theme, so searched its dark theme and got this good news:

[Dark Theme for Minima is here \| Slowbro Blog](https://blog.slowb.ro/dark-theme-for-minima-jekyll/)

But it's not easy to apply the latest changes as needs whole template to be updated. And Minima has not officially released the dark theme, it's only in latest master branch. Finally I made it, could see changes in this PR:

[Dark theme by jchprj · Pull Request #2 · jchprj/jchprj.github.io](https://github.com/jchprj/jchprj.github.io/pull/2)

To summarize, changes include:

- In Gemfile, change minima dependency to GitHub link directly
  `gem "minima", git: "https://github.com/jekyll/minima"`
- Copy all template files from above repo to this site repo, but keep the `_layouts/default.html` for comment function.
- Remove `assets/main.scss` which will cause import issue.
- Update _config.yml to apply latest metadata.

