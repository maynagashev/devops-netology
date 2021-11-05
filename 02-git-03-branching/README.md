# 1. Ветвление, merge и rebase.

- Создать ветки `git-merge` и `git-rebase`, произвести параллельные правки в файлы:
  - [merge.sh](branching/merge.sh)
  - [rebase.sh](branching/rebase.sh)  
- Одну ветку смерджить мерджем, вторую с помощью `rebase` (`fast-forward`).
- Порешать конфликты при rebase.

Результат:
```bash 
* fef98aa (HEAD -> main, origin/main, origin/git-rebase, origin/HEAD, origin-https/main, git-rebase) git-rebase 1
*   3f25373 Merge branch 'git-merge' into main
|\
| * 8ed2ddb (origin/git-merge, origin-https/git-merge, git-merge) merge: use shift
| * 6527763 merge: @ instead *
* | 69b6cbb change rebase.sh in main
|/
* c402bfd prepare for merge and rebase
```
