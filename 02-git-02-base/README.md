# 1. Знакомство с gitlab и bitbucket

1. Создать копию репозитория в гитлабе и битбакете, добавить в качестве remote git в текущий репо.
2. Выполнить push локальной ветки main в новые репозитории. На этом этапе история коммитов во всех трех репозиториях должна совпадать.

---

1. GitHub: [github.com/maynagashev/devops-netology](https://github.com/maynagashev/devops-netology) (origin)
2. Gitlab: [gitlab.com/maynagashev/devops-netology](https://gitlab.com/maynagashev/devops-netology)
3. Bitbucket: [bitbucket.org/maynagashev/devops-netology](https://bitbucket.org/maynagashev/devops-netology/src/main/)

```bash
git remote -v
bitbucket	git@bitbucket.org:maynagashev/devops-netology.git (fetch)
bitbucket	git@bitbucket.org:maynagashev/devops-netology.git (push)
bitbucket-https	https://maynagashev@bitbucket.org/maynagashev/devops-netology.git (fetch)
bitbucket-https	https://maynagashev@bitbucket.org/maynagashev/devops-netology.git (push)
gitlab	git@gitlab.com:maynagashev/devops-netology.git (fetch)
gitlab	git@gitlab.com:maynagashev/devops-netology.git (push)
gitlab-https	https://gitlab.com/maynagashev/devops-netology.git (fetch)
gitlab-https	https://gitlab.com/maynagashev/devops-netology.git (push)
origin	git@github.com:maynagashev/devops-netology.git (fetch)
origin	git@github.com:maynagashev/devops-netology.git (push)
origin-https	https://github.com/maynagashev/devops-netology.git (fetch)
origin-https	https://github.com/maynagashev/devops-netology.git (push)
```

# 2. Теги

Создать легковесный и аннотированный теги, затем пушнуть во все репо.
1. Github: [github.com/maynagashev/devops-netology/tags](https://github.com/maynagashev/devops-netology/tags)
2. Gitlab: [gitlab.com/maynagashev/devops-netology/-/tags](https://gitlab.com/maynagashev/devops-netology/-/tags)
3. Bitbucket: [maynagashev/devops-netology/src/v0.1/](https://bitbucket.org/maynagashev/devops-netology/src/v0.1/) – список тегов расположен в выпадающем меню веток на отдельной вкладке.

# 3. Ветки

1. Создать ветку `fix` на основе одного из предыдущих коммитов.
2. Закоммитить изменения, посмотреть граф: [github.com/maynagashev/devops-netology/network](https://github.com/maynagashev/devops-netology/network) 
3. Посмотреть лог: `git log --oneline --decorate --all --graph`
```bash 
* 93ab11e (origin/fix, origin-https/fix, fix) New line
| * ae1d1d2 (HEAD -> main, tag: v0.1, tag: v0.0, origin/main, origin/HEAD, origin-https/main, gitlab/main, gitlab-https/main, bitbucket/main, bitbucket-https/main) 2.1.2 task description
| * cd3d751 Update task index
| * 5a3c4d0 Small improvements
| * 2745e0d Moved and deleted
|/
* 0d52bae Prepare to delete and move
* a093f79 Added gitignore
* c8b17c9 "First" commit
* f1396ab 1.1. addons
* 45ad237 1.1. devsecops
* 077967b 1.1. devops functions
* 2709584 1.1. app life cycle draft
* 449d92f 1.1. IDE screens
```

# 4. Упрощаем себе жизнь

Закоммитить изменения используя визуальный редактор в IDE.

**Done.**