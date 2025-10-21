To deploy a Hugo static site base on Github Pages
# clone
```shell
git clone https://github.com/WinterArch/WinterArch.github.io.git
cd WinterArch.github.io
git submodule init #Registering themes or modules
git submodule update #Clone them.
```

# local preview
Hugo should be installed before the script was run.
```shell
hugo -D #(D)rafts were included
hugo server -D #To host a 'localhost:dddd' site
```
Then, we can visit the site in brower util Hoster was shut down(Ctrl+c).

# new content
Several steps be excuting by Hugo during `hugo new`.
```
hugo new content content/posts/YourNewContent.md #Command and Correct path is required.
```

# drafts
To modify baseURL in `/hugo.yaml`
```yaml
baseURL: https://winterarch.github.io/
```
For apply baseURL, run `hugo`.
However, drafts wll be excluded, mark the field 'draft' to false in YourNewContent.md.
The default format look like:
```markdown
---
date: 'some Time'
draft: true
title: 'some Text'
---
```

# partial deployment
In fact, the site's generated Hugo-Source-Code is located at the directory `/public`.
And we can just initialize a local git repository in `/public` then push upto Github. In this way, deployment is very eazy.
Open YourGithubRepository/Settings/Pages/Build and deployment/Branch, and set `main/root`.
However, it is also very Annoying to do a series of command like hugo-build|check-directory|git-push EVERY single time, the rest part that Hugo-Configuration is necessary as well.

# entire deployment
It is feasible to `git push` whole repository.
Open YourGithubRepository/Settings/Pages/Build and deployment/Source, and set `Github Actions`.
Select "configure" under "Static HTML", let's see `static.yaml`.
Search "Upload entire repository" and fill the deployment path with `/public` in `static.yaml` .
Now, hugo|git-push is still required, but that is better.

