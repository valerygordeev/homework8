```
Homework8
1. Вход в систему без пароля.
   - Способ 1
       Входим в меню параметров загрузки, нажатием кнопки "e" в меню выбора ядра загрузки.
       Вставляем в конце параметров загрузки init=/bin/sh и перезагружаем ОС с помощью ctrl+x.
       После загрузки ОС рутовая файловая система монтируется в режиме Read-Only. 
       Для перевода ее в режим Read-Write, необходимо перейти в режим суперпользователя и перемонтировать систему
       mount -o remount,rw /
       Для проверки работы системы в режиме Read-Write перейдем домашний каталог рута и запишем файл 
       touch file
       Или выведем свойства системы
       mount | grep root
       Продолжим загрузку 
       exec /sbin/init
   - Способ 2
       Входим в меню параметров загрузки, нажатием кнопки "e" в меню выбора ядра загрузки.
       Вставляем в конце параметров загрузки rd.break и перезагружаем ОС с помощью ctrl+x.
       Попадаем в emergency mode, в котором рутовая файловая система также монтируется в режиме Read-Only.
       Перемонтируем систему на корневую систему sysroot
       mount -o remount,rw /sysroot
       Логинимся и задаем новый пароль
       chroot /sysroot
       passwd root
       Сбрасываем контекст с файла /etc/shadow, создавая пустой файл в корне root
       touch /.autorelabel
       Продолжим загрузку 
       exec /sbin/init
   - Способ 3
       Входим в меню параметров загрузки, нажатием кнопки "e" в меню выбора ядра загрузки.
       Заменяем в строке параметров загрузки ro на rw и дополняем init=/sysroot/bin/sh. Перезагружаем ОС с помощью ctrl+x.
       Попадаем в emergency mode, рутовая файловая система уже смонтирована в режиме Read-Write.
       Логинимся и задаем новый пароль
       chroot /sysroot
       passwd root
       Сбрасываем контекст с файла /etc/shadow, создавая пустой файл в корне root. 
       touch /.autorelabel
       Продолжим загрузку 
       exec /sbin/init
       В процессе дальнейшей загрузки будет проведено восстанавление контекста всех файлов при запуске selinux.

Переименование VG
   1.Проверяем наименование группы:
     vgs
   2.Переименовываем группу:
     vgrename centos_centos7 OtusRoot
   3.Переименовываем группу в конфигурационных файлах:
     sed -i 's/centos_centos7/OtusRoot/g' /etc/fstab
     sed -i 's/centos_centos7/OtusRoot/g' /etc/default/grub
     sed -i 's/centos_centos7/OtusRoot/g' /boot/grub2/grub.cfg
   4.Пересобираем initrd image с новым наименованием VG
     mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
   5.Перезагружаем VM
   6.Проверяем наименование VG
     vgs

Добавление скриптов в initrd
   1.Создаем директорию 01test
     mkdir /usr/lib/dracut/modules.d/01test
   2.Создаем скрипты:
     module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh
        #!/bin/bash

        check() {
            return 0
        }

        depends() {
            return 0
        }

        install() {
            inst_hook cleanup 00 "${moddir}/test.sh"
        }
   test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик
        #!/bin/bash

        exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
        cat <<'msgend'
        Hello! You are in dracut module!
         ___________________
        < I'm dracut module >
         -------------------
           \
            \
                .--.
               |o_o |
               |:_/ |
              //   \ \
             (|     | )
            /'\_   _/`\
            \___)=(___/
        msgend
        sleep 10
        echo " continuing...."
   3.Пересобираем образ initrd
     mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
   или
     dracut -f -v
   4.Можно проверить загружен ли модуль test в образ:
     lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
      Что бы вывести на экран скрипт test.sh нужно убрать отображение анимации при загрузки системы (rghb) и "тихий" режим (quiet), скрытие       отображения загрузки системы.
      Этого можно достигнуть двумя способами:
         - Временный - при загрузки системы войти в режим edit и убрать значения rghb и quiet в блоке linux16
         - Постоянно - убрать опции rghb и queit в файле /boot/grub2/grub.cfg, с последующей сборкой dracut -f -v
```
