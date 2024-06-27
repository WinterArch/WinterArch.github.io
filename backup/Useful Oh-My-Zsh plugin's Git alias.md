| alias  | command                                 | note                          |
| ------ | --------------------------------------- | ----------------------------- |
| g      | git                                     |                               |
| ga     | git add                                 |         |
| gaa    | git add --all                           | ' .' can do as well  |
| gb     | git branch                              |                               |
| gc     | git commit                              |                               |
| gc!    | git commit -v -amend                    |                               |
| gcmsg  | git commit -m                           |                               |
| gd     | git diff                                |                               |
| gds    | git diff --staged                       |                               |
| gdup   | git diff @{upstream}                    | need 'upstream'             |
| gf     | git fetch                               |                               |
| gfo    | git fetch origin                        |                               |
| glol   | git log --graph --pretty='...'          | Red Hash, White Msg, Green Rel Date, Blue Author |
| glod   | git log --graph --pretty='...'          |Red Hash, White Msg, Green Abs Date, Blue Author |
| glog   | git log --oneline --decorate --graph    |                               |
| gm     | git merge                               |                               |
| grb    | git rebase                              |                               |
| grbi   | git rebase --interactive                |                               |
| grev   | git revert                              |                               |
| grs    | git restore                             |                               |
| grst   | git restore --staged                    |                               |
| gst    | git status                              |                               |
| gss    | git status -s                           |  s for \[s\]hort                             |
| gsta   | git stash push                          |                               |
| gstall | git stash push --all                    |    |
| gstp   | git stash pop                           |                               |
| gstl   | git stash list                          |                               |
| gstaa  | git stash apply                         |                               |
| gsw    | git switch                              |                               |
| gswc   | git switch -c                           |  c for \[c\]reate                             |
| gswm   | git switch $(git_main_branch)           | need 'main'           |
| gswd   | git switch $(git_develop_branch)        | need 'dev'             |
| gluc   | git pull upstream $(git_current_branch) | need 'upstream'             |
| gwt    | git worktree                            |                               |
| gwta   | git worktree add                        |                               |
| gwtls  | git worktree list                       |                               |
| gwtmv  | git worktree move                       |                               |
| gwtrm  | git worktree remove                     |                               |
