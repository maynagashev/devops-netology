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

Аналогично можно использовать `git log`
```bash
git -P log aefea --format="%H %s" -1
aefead2207ef7e2aa5dc81a34aedf0cad4c32545 Update CHANGELOG.md
```

## 2. Какому тегу соответствует коммит `85024d3`?

```bash
git -P show -s 85024d3 --format="%D"
tag: v0.12.23
```

Где:
- `%D` - выводит ref names коммита: теги, названия веток, аналог опции `--decorate`

## 3. Сколько родителей у коммита `b8d720`? Напишите их хеши.

Два родителя (мердж коммит):
```bash
git -P show -s b8d720 --format="%P"
56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b
```

Более детальная информация о родителях:
```bash
❯ git -P show -s b8d720^1 --format="%H %s"
56cd7859e05c36c06b56d013b55a252d0bb7e158 Merge pull request #23857 from hashicorp/cgriggs01-stable
❯ git -P show -s b8d720^2 --format="%H %s"
9ea88f22fc6269854151c571162c5bcf958bee2b add/update community provider listings
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

Для поиска исходного коммита в котором появилась функция подходит `git log -S`, дополнительно выводим дату и меняем сортировку так, чтобы вначале показывались более старые коммиты.
```
git -P log -S "func providerSource(" --format="%h %ad %s" --date=short --reverse
8c928e835 2020-04-02 main: Consult local directories as potential mirrors of providers
```

## 6. Найдите все коммиты в которых была изменена функция `globalPluginDirs`.

Грепаем в каком файле находится функция.
```bash
❯ git -P grep "func.*globalPluginDirs.*("
plugins.go:func globalPluginDirs() []string {
```

Выводим коммиты с изменениями тела функции `globalPluginDirs` в `plugins.go`
```bash
git -P log -L :globalPluginDirs:plugins.go --oneline -s
78b122055 Remove config.go and update things using its aliases
52dbf9483 keep .terraform.d/plugins for discovery
41ab0aef7 Add missing OS_ARCH dir to global plugin paths
66ebff90c move some more plugin search path logic to command
8364383c3 Push plugin discovery down into command package
```

## 7. Кто автор функции `synchronizedWriters`? 
Ищем первое упоминание названия функции в истории, для этого выводим в обратном порядке лог с поиском, датой и автором изменений:
```bash
 git -P log -S "synchronizedWriters" --reverse --format="%h %ad %an" --date=short
5ac311e2a 2017-05-03 Martin Atkins
fd4f7eb0b 2020-10-21 James Bardin
bdfea50cc 2020-11-30 James Bardin
```
Самый старый коммит `5ac311e2a` от 2017-05-03, автор **Martin Atkins**. Проверяем что функция была добавлена в этом коммите
```bash
❯ git -P show 5ac311e2a | grep synchronizedWriters
+		wrapped := synchronizedWriters(stdout, stderr)
+// synchronizedWriters takes a set of writers and returns wrappers that ensure
+func synchronizedWriters(targets ...io.Writer) []io.Writer {
```