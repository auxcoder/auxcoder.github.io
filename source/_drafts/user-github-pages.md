---
layout: false
title: "Github user pages"
comments: false
categories: Coding
tags:
- javascript
- github
- Hexo
---

## Generate static site in Github using Hexo
First check Github documentation about [User Organization and Project pages](https://help.github.com/articles/user-organization-and-project-pages/) to get familiar with.

### Configure Hexo to deploy to Git

We can use
```sh
npm install --save hexo-deployer-git
```

Configure *Hexo* to deploy to Github
```yml
deploy:
    type: git
    repo: <repo url>
    branch: <branch>
```

<!---
*References*
+ [How To Build A Blog With Hexo On Github Page](https://commitlogs.com/2016/09/03/how-to-build-blog-with-hexo/)
--->
