# Домашнее задание к занятию "3.1. Работа в терминале, лекция 1"

1. [x] Установите средство виртуализации [Oracle VirtualBox](https://www.virtualbox.org/).
2. [x] Установите средство автоматизации [Hashicorp Vagrant](https://www.vagrantup.com/).
3. [x] В вашем основном окружении подготовьте удобный для дальнейшей работы терминал: `iTerm2`.
4. [x] С помощью базового файла конфигурации запустите Ubuntu 20.04 в VirtualBox посредством Vagrant.
    - для того чтобы стартовало на macOS Monterey прописал в Vagrantfil: `vb.gui = true` (см.
      issue: [macOS Monterey and VirtualBox 6.1.28 #12557](https://github.com/hashicorp/vagrant/issues/12557))
5. [x] Ознакомьтесь с графическим интерфейсом VirtualBox, посмотрите как выглядит виртуальная машина, которую создал для
   вас Vagrant, какие аппаратные ресурсы ей выделены. **Какие ресурсы выделены по-умолчанию?**
   - 2 процессора
   - 1024MB памяти
```bash
vagrant@vagrant:~$ cat /proc/meminfo | grep MemTotal
MemTotal:        1004584 kB
vagrant@vagrant:~$ nproc --all
2
```
6. [ ] Ознакомьтесь с возможностями конфигурации VirtualBox через
   Vagrantfile: [документация](https://www.vagrantup.com/docs/providers/virtualbox/configuration.html). **Как добавить оперативной памяти или ресурсов процессора виртуальной машине?**
7. [x] Команда `vagrant ssh` из директории, в которой содержится Vagrantfile, позволит вам оказаться внутри виртуальной
   машины без каких-либо дополнительных настроек. Попрактикуйтесь в выполнении обсуждаемых команд в терминале Ubuntu.

