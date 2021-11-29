# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

### 1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`.

Т.к. по дефолту strace выводит в stderr, поэтому просто так его не грепнешь. Для поиска нужных строк перенаправляем `stderr => stdout` и грепаем `stdout`, ищу упоминания папки `/tmp`
```bash
vagrant@vagrant:~$ strace -f /bin/bash -c 'cd /tmp' 2>&1 | grep  /tmp
execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7fff1af69ac8 /* 27 vars */) = 0
stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
chdir("/tmp")                           = 0
```

Искомый системный вызов: `chdir("/tmp")` 

```bash
vagrant@vagrant:~$ man 2 chdir | grep changes
       chdir() changes the current working directory of the calling process to the directory specified in path.
```

### 2. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
   **Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.**

На основе `magiс` файлов:
- конфига шаблонов `/etc/magic`
- бинарника с преднастроенным шаблонами `/usr/share/misc/magic.mgc`

```bash
vagrant@vagrant:~$ strace -f -y -e openat file /bin/bash
...
openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3</etc/magic>
openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3</usr/lib/file/magic.mgc>
...
/bin/bash: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=a6cb40078351e05121d46daa768e271846d5cc54, for GNU/Linux 3.2.0, stripped
```


### 3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).

```bash
# создаем долгоиграющий процесс который пишет в ping.log
vagrant@vagrant:~$ ping ya.ru > ping.log &
[1] 5476
# проверяем наличие файла и удаляем его
vagrant@vagrant:~$ ll ping.log
-rw-rw-r-- 1 vagrant vagrant 1015 Nov 29 13:46 ping.log
vagrant@vagrant:~$ unlink ping.log
# смотрим открытые файлы процесса (сразу с пометкой deleted)
# - в третьей колонке справа 5134 - текущий размер файла
# - 1w - номер файлового дескриптора процесса открытый на запись 
root@vagrant:~# lsof -p 5476 | grep deleted
ping    5476 vagrant    1w   REG  253,0     5134 131112 /home/vagrant/ping.log (deleted)
# проверяем файл повторно размер увеличивается 13457
root@vagrant:~# lsof -p 5476 | grep deleted
ping    5476 vagrant    1w   REG  253,0    13457 131112 /home/vagrant/ping.log (deleted)
# проверяем ссылку на файл оставшуюся в дескрипторах процесса
root@vagrant:~# ll /proc/5476/fd/1
l-wx------ 1 root root 64 Nov 29 13:47 /proc/5476/fd/1 -> '/home/vagrant/ping.log (deleted)'

# очищаем файл по ссылке (дескриптору) - вариантов много (true> , :> , echo>) - смысл в том чтобы записать в файл пустые данные
root@vagrant:~# true > /proc/5476/fd/1
# проверяем что размер обнулился
root@vagrant:~# lsof -p 5476 | grep deleted
ping    5476 vagrant    1w   REG  253,0       71 131112 /home/vagrant/ping.log (deleted)
```

Тем самым файл можно обнулить используя **сохраненную в дескрипторе** ссылку (которую по факту использует и само приложение для записи).
- достаточно записать в дескриптор пустые данные перенаправлением `true > /proc/5476/fd/1`
- можно так же использовать команду `truncate`, обрезав файл до нужного размера:
```bash
root@vagrant:~# truncate -s 100 /proc/5476/fd/1 && lsof -p 5476 | grep deleted
ping    5476 vagrant    1w   REG  253,0      100 131112 /home/vagrant/ping.log (deleted)
```

### 4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

**Зомби-процесс** – это процесс уже завершивший свое исполнение (системным вызовом `exit`), родитель которого еще не обработал закрытие дочернего процесса (системный вызов `wait`).

Зомби **не потребляют основные ресурсы системы**. Занимают только место в списке процессов (которое лимитировано). 

Лимиты текущего пользователя можно посмотреть с помощью `ulimit`:
```bash
vagrant@vagrant:~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7595
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7595
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

### 5. В iovisor BCC есть утилита `opensnoop`:
    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
   **На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).**

`opensnoop-bpfcc` - позволяет отслеживать открытия файлов процессами в реальном времени, использует подсистему ядра Linux: **eBPF/bcc**.

```bash
root@vagrant:~# opensnoop-bpfcc
PID    COMM               FD ERR PATH
832    vminfo              6   0 /var/run/utmp
607    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
607    dbus-daemon        18   0 /usr/share/dbus-1/system-services
607    dbus-daemon        -1   2 /lib/dbus-1/system-services
607    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/
1974   bash                3   0 /etc/ld.so.cache
1974   bash                3   0 /lib/x86_64-linux-gnu/libtinfo.so.6
1974   bash                3   0 /lib/x86_64-linux-gnu/libdl.so.2
1974   bash                3   0 /lib/x86_64-linux-gnu/libc.so.6
1974   bash                3   0 /dev/tty
```

### 6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
### 7. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
   Есть ли смысл использовать в bash `&&`, если применить `set -e`?
### 8. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
### 9. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).

 