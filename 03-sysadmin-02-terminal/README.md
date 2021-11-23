# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

## 1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.
- Встроенная в оболочку (в нашем случае bash) команда, не программа.
```bash
vagrant@vagrant:~$ type cd
cd is a shell builtin
```
- Команда меняет текущую рабочую директорию оболочки, т.е. ее внутреннее состояние, поэтому она достаточно жестко привязана к конкретному шеллу и конкретной версии шелла.

## 2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.
- Можно использовать флаг `-c` у `grep`.
```bash
vagrant@vagrant:/$ printf "a\na\na\n" | grep "a" | wc -l
3
vagrant@vagrant:/$ printf "a\na\na\n" | grep -c "a"
3
```
## 3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
```bash
vagrant@vagrant:/$ ps h 1
      1 ?        Ss     0:04 /sbin/init
vagrant@vagrant:/$ ps hc 1
      1 ?        Ss     0:04 systemd
```
Где:
- `h` - отключение отображения заголовка вывода
- `c` - вывод настоящего имени команды

## 4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?

В терминале `/dev/pts/0` запускаем команду и перенаправляем ошибки в `/dev/pts/1`:
```bash
ls not_existing_dir 2>/dev/pts/1
```

**Проверка**
Открываем несколько терминалов и смотрим их названия, текущий терминал pts/2:
```bash
vagrant@vagrant:~$ who
vagrant  pts/0        2021-11-23 03:49 (10.0.2.2)
vagrant  pts/1        2021-11-22 02:15 (10.0.2.2)
vagrant  pts/2        2021-11-23 03:51 (10.0.2.2)
vagrant@vagrant:~$ tty
/dev/pts/2
```
Отправляем из второго терминала ошибки `ls` в нулевой (предварительно проверив ошибку в текущем терминале):
```bash
vagrant@vagrant:~$ ls test
ls: cannot access 'test': No such file or directory
vagrant@vagrant:~$ ls test 2>/dev/pts/0
```
В `/dev/pts/0` при этом выводится ошибка:
```bash
vagrant@vagrant:~$ tty
/dev/pts/0
vagrant@vagrant:~$ ls: cannot access 'test': No such file or directory
```
## 5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
Получится.

**Пример:**
Создаем файл из которого будем читать и смотрим название текущего терминала, чтобы вывести туда вывод используя перенаправление:
```bash
vagrant@vagrant:~$ printf "1\n2\n3\n" > README.md
vagrant@vagrant:~$ tty
/dev/pts/2
```

Читаем из одного файла, записываем результат в другой (для простоты в этот же терминал):
```bash
vagrant@vagrant:~$ wc -l <README.md >/dev/pts/2
3
```
## 6. Получится ли вывести находясь в графическом режиме данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?
Получится, данные наблюдаются в соотв. терминалах.

**Пример:**

PTY терминал: `/dev/pts/2`
```bash
vagrant@vagrant:~$ who
vagrant  tty1         2021-11-23 04:35
vagrant  pts/0        2021-11-23 03:49 (10.0.2.2)
vagrant  pts/1        2021-11-22 02:15 (10.0.2.2)
vagrant  pts/2        2021-11-23 03:51 (10.0.2.2)
vagrant  tty6         2021-11-23 04:37
vagrant@vagrant:~$ tty
/dev/pts/2
vagrant@vagrant:~$ echo "message from $( tty )" > /dev/tty6
vagrant@vagrant:~$ message from /dev/tty6
```
TTY терминал: `/dev/tty6`
![tty](./screen-tty.png)

## 7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?
1. `bash 5>&1` - создается новый (дополнительный) файловый дескриптор для текущего процесса с номером 5, так же создается перенаправление из 5 в 1 дескриптор (`stdout`).
2. `echo netology > /proc/$$/fd/5` - пишем напрямую в файловый дескриптор 5, т.к. ранее мы настроили перенаправление из него в 1 (стандартный вывод), содержимое выводится на текущий терминал.

```bash
vagrant@vagrant:~$ ll /proc/$$/fd
total 0
dr-x------ 2 vagrant vagrant  0 Nov 23 04:57 ./
dr-xr-xr-x 9 vagrant vagrant  0 Nov 23 04:57 ../
lrwx------ 1 vagrant vagrant 64 Nov 23 04:57 0 -> /dev/pts/2
lrwx------ 1 vagrant vagrant 64 Nov 23 04:57 1 -> /dev/pts/2
lrwx------ 1 vagrant vagrant 64 Nov 23 04:57 2 -> /dev/pts/2
lrwx------ 1 vagrant vagrant 64 Nov 23 05:04 255 -> /dev/pts/2
vagrant@vagrant:~$ bash 5>&1
```

Видим что после выполнения команды создался новый дескриптор `5`:
```bash
vagrant@vagrant:~$ ll /proc/$$/fd
total 0
dr-x------ 2 vagrant vagrant  0 Nov 23 05:04 ./
dr-xr-xr-x 9 vagrant vagrant  0 Nov 23 05:04 ../
lrwx------ 1 vagrant vagrant 64 Nov 23 05:04 0 -> /dev/pts/2
lrwx------ 1 vagrant vagrant 64 Nov 23 05:04 1 -> /dev/pts/2
lrwx------ 1 vagrant vagrant 64 Nov 23 05:04 2 -> /dev/pts/2
lrwx------ 1 vagrant vagrant 64 Nov 23 05:04 255 -> /dev/pts/2
lrwx------ 1 vagrant vagrant 64 Nov 23 05:04 5 -> /dev/pts/2
vagrant@vagrant:~$ echo netology > /proc/$$/fd/5
netology
```

## 8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа. 9то можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.
Да получится, меняем местами потоки stdout(1)/stderr(2) используя дополнительный дескриптор `5`, по следующей схеме:
```bash
5(new) => 2(err) # сохраняем в 5 ссылку на исходный stderr
2(err) => 1 (out) # перенаправляем stderr в stdout, для работы pipe
1(out) => 5 (new) => 2(err) # а исходный stdout в промежуточный 5 (сохраненный ранее stderr)
```

**Проверка:**
```bash
vagrant@vagrant:~$ ls test 5>&2 2>&1 1>&5 | cat 1>stdout.log 2>stderr.log
vagrant@vagrant:~$ cat stderr.log
vagrant@vagrant:~$ cat stdout.log
ls: cannot access 'test': No such file or directory
```
Тем самым исходная ls ничего не выводит в консоль, ошибка уходит в `pipe` через `stdout`, и там записывается в файлы для проверки содержимого потоков.

Если директория существует, то список файлов отправляется в stderr (т.к. мы поменяли их местами) и не отправляется в pipe (выводится тут же, а потоки в pipe оказываются пустые):
```bash
vagrant@vagrant:~$ ls -la existed 5>&2 2>&1 1>&5 | cat 1>stdout.log 2>stderr.log
total 8
drwxrwxr-x 2 vagrant vagrant 4096 Nov 23 05:46 .
drwxr-xr-x 6 vagrant vagrant 4096 Nov 23 05:46 ..
vagrant@vagrant:~$ cat stdout.log
vagrant@vagrant:~$ cat stderr.log
```

## 9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?

- Выводит значение переменных окружения текущего процесса (с `NUL` символом в качестве разделителя). 
- Для получения аналогично вывода можно использовать `printenv -0` или `env -0`.

```bash
vagrant@vagrant:~$ cat /proc/$$/environ
SHELL=/bin/bashLANGUAGE=en_US:PWD=/home/vagrantLOGNAME=vagrantXDG_SESSION_TYPE=ttyMOTD_SHOWN=pamHOME=/home/vagrantLANG=C.UTF-8 ... SSH_TTY=/dev/pts/2_=/usr/bin/bashvagrant@vagrant:~$
vagrant@vagrant:~$ printenv -0
SHELL=/bin/bashLANGUAGE=en_US:PWD=/home/vagrantLOGNAME=vagrantXDG_SESSION_TYPE=ttyMOTD_SHOWN=pamHOME=/home/vagrantLANG=C.UTF-8 ... SSH_TTY=/dev/pts/2_=/usr/bin/printenv
```
Где: 
- `-0` - выводит значения переменных окружения процесса с разделителем NUL вместо NEWLINE (аналогично как оно выводится в `cat /proc/$$/environ`).

## 10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.

Смотрим в `man proc`:
1.`/proc/[pid]/cmdline`- содержит полную строку запуска процесса со всеми аргументами.
- файл только для чтения
- у зомби-процессов будет пустой (0 симоволов)
- аргументы командной строки разделяются нулевым байтом `\0` (символ так же добавляется в конце строки аргументов)              
2. `/proc/[pid]/exe` - содержит символическую ссылку на запускаемый (executable) файл процесса.
- доступен в Linux 2.2 и старше
- при попытке открыть ссылку, будет запущена копия текущей команды
- запускаемый файл был удален, то ссылка будет содержать строку '(deleted)'
- в многопотоковых процессах, если завершить основной поток, то ссылка больше будет не доступна.

Пример просмотра информации о процессе tail в соседнем терминале:
```bash
vagrant@vagrant:~$ pgrep tail
15076

vagrant@vagrant:~$ ls -lah /proc/15076/exe
lrwxrwxrwx 1 vagrant vagrant 0 Nov 23 06:30 /proc/15076/exe -> /usr/bin/tail

vagrant@vagrant:~$ cat /proc/15076/cmdline
tail-f/var/log/sysl
```
## 11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.

Из флагов процессора: `sse4_2`
```bash
grep sse /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single pti fsgsbase avx2 invpcid rdseed clflushopt md_clear flush_l1d
```

## 12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:
    ```bash
	vagrant@netology1:~$ ssh localhost 'tty'
	not a tty
    ```
**Почитайте, почему так происходит, и как изменить поведение.**

- При запуске одиночной команды по ssh – по умолчанию псевдотерминал не выделяется (не создается), поэтому команда `tty` возвращает ответ `not a tty`.
- Принудить ssh создать псевдотерминал под команду – можно флагом `-t`.

**Пример:**
```bash
vagrant@vagrant:~$ ssh -t localhost 'tty'
vagrant@localhost's password:
/dev/pts/2
Connection to localhost closed.
```


## 13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.
- По дефолту не дало перенести процесс, т.к. из соображений безопасности разрешено только прямым потомкам делать PTRACE: `kernel.yama.ptrace_scope = 1`
- После выставления `kernel.yama.ptrace_scope = 0` и ребута, тоже не заработало, просто выдавало `Operation not permitted`
- Помогла установка флага `reptyr -T`
```bash
vagrant@vagrant:~$ ps aux | grep tail
vagrant     1297  0.0  0.0   9864   592 pts/2    S+   07:20   0:00 tail -f /var/log/syslog
vagrant@vagrant:~$ reptyr -T 1297
Unable to attach to pid 1297: Operation not permitted
vagrant@vagrant:~$ sudo reptyr -T 1297
Nov 23 07:21:40 vagrant systemd[1]: Started Session 9 of user vagrant.
Nov 23 07:23:54 vagrant systemd[1]: Starting Ubuntu Advantage APT and MOTD Messages...
Nov 23 07:23:58 vagrant systemd[1]: ua-messaging.service: Succeeded.
Nov 23 07:23:58 vagrant systemd[1]: Finished Ubuntu Advantage APT and MOTD Messages.
```
Где:
- `-T` - использование альтернативного режима присоединения к сессии, т.н. **TTY-stealing** и оно требует рута.
  
> In this mode, reptyr will not ptrace(2) the target process, but will attempt to discover the terminal emulator for that process' pty, and steal the master end of the pty. This mode is more reliable and flexible in many circumstances (for in‐
  stance, it can attach all processes on a tty, rather than just a single process). However, as a downside, children of sshd(8) cannot be attached via -T unless reptyr is run as root. See ⟨https://blog.nelhage.com/2014/08/new-reptyr-feature-tty-stealing/⟩ for more information about tty-stealing

## 14. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.
- `tee <file>` - читает поток со `stdin`, и пишет его одновременно:
    1. в `stdout` (в данном случае в консоль)
    2. в указанные файлы (переданные в качестве аргументов)
- `echo string | sudo tee <file>` - берет данные из pipe, и уже от рута **сама** пишет в указанный файл (**не используя перенаправление оболочки**).
  
Поэтому `tee` работает, а например `sudo cat` в следующем примере нет:
```bash
vagrant@vagrant:~$ echo string | sudo cat > /root/new_file2
-bash: /root/new_file2: Permission denied
``` 

**Рабочий вариант:**
```bash
vagrant@vagrant:~$ echo string | sudo tee /root/new_file
string
vagrant@vagrant:~$ sudo cat /root/new_file
string
```
 