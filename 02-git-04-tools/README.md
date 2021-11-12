# Инструменты Git

Для выполнения заданий в этом разделе давайте склонируем репозиторий с исходным кодом
терраформа https://github.com/hashicorp/terraform

В виде результата напишите текстом ответы на вопросы и каким образом эти ответы были получены.

## 1. Найдите полный хеш и комментарий коммита, хеш которого начинается на `aefea`.
```bash
git -P show aefea -s --format="%H %s"
aefead2207ef7e2aa5dc81a34aedf0cad4c32545 Update CHANGELOG.md
```
Где:
- `-P` - отключает пагинацию при выводе, сокращение от `--no-pager`
- `-s` - отключает вывод diff, сокращение от `--no-patch` 

```bash
git -P log aefea --format="%H %s" -1
aefead2207ef7e2aa5dc81a34aedf0cad4c32545 Update CHANGELOG.md
```

## 2. Какому тегу соответствует коммит `85024d3`?

```bash
git -P show -s 85024d3 --format="%d"
 (tag: v0.12.23)
```

## 3. Сколько родителей у коммита `b8d720`? Напишите их хеши.

Два родителя (мердж коммит):
```bash
git -P show -s b8d720 --format="%P"
56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b
```

## 4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.

Не включая сами коммиты с тегами:
```bash
git -P log v0.12.23..v0.12.24~1 --format="%h %s %d"
b14b74c49 [Website] vmc provider links
3f235065b Update CHANGELOG.md
6ae64e247 registry: Fix panic when server is unreachable
5c619ca1b website: Remove links to the getting started guide's old location
06275647e Update CHANGELOG.md
d5f9411f5 command: Fix bug when using terraform login on Windows
4b6d06cc5 Update CHANGELOG.md
dd01a3507 Update CHANGELOG.md
225466bc3 Cleanup after v0.12.23 release
```

Включительно:
```bash
git -P log v0.12.23^..v0.12.24 --format="%h %s %d"
33ff1c03b v0.12.24  (tag: v0.12.24)
b14b74c49 [Website] vmc provider links
3f235065b Update CHANGELOG.md
6ae64e247 registry: Fix panic when server is unreachable
5c619ca1b website: Remove links to the getting started guide's old location
06275647e Update CHANGELOG.md
d5f9411f5 command: Fix bug when using terraform login on Windows
4b6d06cc5 Update CHANGELOG.md
dd01a3507 Update CHANGELOG.md
225466bc3 Cleanup after v0.12.23 release
85024d310 v0.12.23  (tag: v0.12.23)
```

## 5. Найдите коммит в котором была создана функция `func providerSource`, ее определение в коде выглядит так `func providerSource(...)` (вместо троеточего перечислены аргументы).

## 6. Найдите все коммиты в которых была изменена функция `globalPluginDirs`.

## 7. Кто автор функции `synchronizedWriters`? 