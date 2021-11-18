# Домашнее задание к занятию "3.1. Работа в терминале, лекция 1"

1. [x] Установите средство виртуализации [Oracle VirtualBox](https://www.virtualbox.org/).
2. [x] Установите средство автоматизации [Hashicorp Vagrant](https://www.vagrantup.com/).
3. [x] В вашем основном окружении подготовьте удобный для дальнейшей работы терминал: `iTerm2`.
4. [x] С помощью базового файла конфигурации запустите Ubuntu 20.04 в VirtualBox посредством Vagrant.
    - для того чтобы стартовало на macOS Monterey, прописал в `Vagrantfile`: `vb.gui = true` (см.
      issue: [macOS Monterey and VirtualBox 6.1.28 #12557](https://github.com/hashicorp/vagrant/issues/12557))
5. [x] Ознакомьтесь с графическим интерфейсом VirtualBox, посмотрите как выглядит виртуальная машина, которую создал для
   вас Vagrant, какие аппаратные ресурсы ей выделены.
    - **Какие ресурсы выделены по-умолчанию?**
        - 2 процессора
        - 1024MB памяти
      ```bash
      vagrant@vagrant:~$ cat /proc/meminfo | grep MemTotal
      MemTotal:        1004584 kB
      vagrant@vagrant:~$ nproc --all
      2
      ```
6. [x] Ознакомьтесь с возможностями конфигурации VirtualBox через
   Vagrantfile: [документация](https://www.vagrantup.com/docs/providers/virtualbox/configuration.html).
    - **Как добавить оперативной памяти или ресурсов процессора виртуальной машине?**
      ```terraform
         config.vm.provider :virtualbox do |v|
             # ограничение по использованию процессора хост машины в процентах
             v.customize ["modifyvm", :id, "--cpuexecutioncap", "30"]
             # выделенная память
             v.memory = 2048
             # кол-во процессоров
             v.cpus = 4
         end
      ```
7. [x] Команда `vagrant ssh` из директории, в которой содержится Vagrantfile, позволит вам оказаться внутри виртуальной
   машины без каких-либо дополнительных настроек. Попрактикуйтесь в выполнении обсуждаемых команд в терминале Ubuntu.
8. **Ознакомиться с разделами `man bash`, почитать о настройках самого bash:**
    * **какой переменной можно задать длину журнала `history`, и на какой строчке manual это описывается?**
        - Переменная `HISTFILESIZE` – описывается в разделах `Shell Variables` и `HISTORY`:
      ```bash
        # Manual page bash(1) line 621/3398 21%
        HISTFILESIZE - размер файла истории (кол-во строк) до которого он обрезается.
              The  maximum  number  of  lines  contained in the history file.  When this variable is assigned a value, the history file is truncated, if necessary, to contain no more than that number of
              lines by removing the oldest entries.  The history file is also truncated to this size after writing it when a shell exits.  If the value is 0, the history file is truncated to zero  size.
              Non-numeric values and numeric values less than zero inhibit truncation.  The shell sets the default value to the value of HISTSIZE after reading any startup files.
        # Manual page bash(1) lines: 2271-2279/3398
        HISTORY
           ...
           On startup, the history is initialized from the file named by the variable HISTFILE (default ~/.bash_history).  The file named by the value of HISTFILE is truncated, if necessary, to  contain  no
           more  than  the  number of lines specified by the value of HISTFILESIZE.  If HISTFILESIZE is unset, or set to null, a non-numeric value, or a numeric value less than zero, the history file is not
           truncated.
      ```
        - Не следует путать `HISTFILESIZE` и `HISTSIZE`. `HISTSIZE` – размер списка (буфера) который будет сохранен в `HISTFILE`
          по завершению сессии (после записи файл истории обрезается до `HISTFILESIZE`)
        - Значения по дефолту в виртуалке:
          ```bash
          vagrant@vagrant:~$ set | grep SIZE
          set | grep SIZE
          HISTFILESIZE=2000
          HISTSIZE=1000
          ```
    * **что делает директива `ignoreboth` в bash?**
        - `ignoreboth` – это одно из значений настройки `HISTCONTROL`, которая в свою очередь отвечает за то какие
          команды будут сохранятся в историю
        - `ignoreboth` – является **сокращением для двух следующих значений включенных одновременно**:
           1. `ignorespace` - игнорировать команды с пробелом символом в начале команды. 
           2. `ignoredups` - не записывать в историю идущие подряд одинаковые команды.
        - по дефолту директива в тестовой виртуалке активирована:
          ```bash
          vagrant@vagrant:~$ set | grep HISTCONTROL
          HISTCONTROL=ignoreboth
          ```
9. **В каких сценариях использования применимы скобки `{}` и на какой строчке `man bash` это описано?**
   - для группировки команд (понятие `group command`), в формате: `{ list; }`, где `list` пайплайн из команд, описано на 206 строке:
    ```bash
     { list; }
              list  is simply executed in the current shell environment.  list must be terminated with a newline or semicolon.  This is known as a group command.  The return status is the exit status of
              list. 
   ```
   - для генерации строк на основе списка или интервала значений, раздел в мануале `Brace Expansion` (794 строка), например:
       - `a{d,c,b}e` трансформируется в `ade ace abe`
       - `{1..10}` в `1 2 3 4 5 6 7 8 9 10`
       - `{1..10..3}` в `1 4 7 10` (третий параметр значение инкремента)
     
10. **Основываясь на предыдущем вопросе, как создать однократным вызовом `touch` 100000 файлов?**
    - в качестве аргументов для `touch` задаем интервал имен файлов
      ```bash
      touch file{1..100000}
      # считаем созданные файлы
      ls | wc -l
      100000
      ```
    - **А получилось ли
      создать 300000? Если нет, то почему?**
        - 300000 не создастся из-за ограничения по кол-ву аргументов для баш команд
        - при чем в данном случае не будет создан ни один файл, т.к. одной команде будет передан сразу весь список аргументов, что сразу вызовет ошибку
        ```bash
          vagrant@vagrant:~$ touch file{1..300000}
          bash: /usr/bin/touch: Argument list too long
          ls | wc -l 
          0
        ```
11. **В `man bash` поищите по `/\[\[`. Что делает конструкция `[[ -d /tmp ]]`**
    - инструкция проверяет существует ли в системе директория `/tmp`
    - в двойных квадратных скобках проверяется истинность переданного выражения, возвращает 1 (истина) или 0 (ложь).
    ```bash
    CONDITIONAL EXPRESSIONS
        ...
       -d file
          True if file exists and is a directory.
    ```
12. **Основываясь на знаниях о просмотре текущих (например, `PATH`) и установке новых переменных; командах, которые мы
    рассматривали, добейтесь в выводе `type -a bash` в виртуальной машине наличия первым пунктом в списке:**

```bash
bash is /tmp/new_path_directory/bash
bash is /usr/local/bin/bash
bash is /bin/bash
```
(прочие строки могут отличаться содержимым и порядком)
В качестве ответа приведите команды, которые позволили вам добиться указанного вывода или соответствующие скриншоты.

```bash
vagrant@vagrant:~$ NEW_PATH=/tmp/new_path_directory/bash
vagrant@vagrant:~$ mkdir -pv $NEW_PATH
mkdir: created directory '/tmp/new_path_directory/bash'
vagrant@vagrant:~$ cp -v $(which bash) $NEW_PATH
'/usr/bin/bash' -> '/tmp/new_path_directory/bash/bash'
vagrant@vagrant:~$ PATH=$NEW_PATH:$PATH
vagrant@vagrant:~$ type -a bash
bash is /tmp/new_path_directory/bash/bash
bash is /usr/bin/bash
bash is /bin/bash
```

Тоже самое одной строкой для проверки:
```bash
NEW_PATH=/tmp/new_path_directory/bash && mkdir -pv $NEW_PATH && cp -v $(which bash) $NEW_PATH && PATH=$NEW_PATH:$PATH && type -a bash
```

13. **Чем отличается планирование команд с помощью `batch` и `at`?**
     - `batch` - выполняет команды когда система не сильно загружена (по дефолту когда load average < 1.5). 
        Например, в тестовой ненагруженной среде команды выполняются сразу (единоразово).
     - `at` выполняет команду (или пакет команд) в конкретное указанное время (так же единоразово).

14. **Завершите работу виртуальной машины чтобы не расходовать ресурсы компьютера и/или батарею ноутбука.**
```bash
vagrant@vagrant:~$ exit
logout
Connection to 127.0.0.1 closed.
❯ vagrant suspend
==> default: Saving VM state and suspending execution...
```
