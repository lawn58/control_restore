# control_restore

1. Попасть в систему без пароля несколькими способами
2. Установить систему с LVM, после чего переименовать VG
3. Добавить модуль в initrd


1. Для получения доступа необходимо открыть GUI VirtualBox (или другой системы виртуализации), запустить виртуальную машину и при выборе ядра для загрузки нажать e - в данном контексте edit. Попадаем в окно где мы можем изменить параметры загрузки.


Способ 1. init=/bin/sh●В конце строки начинающейся с linux16 добавляем init=/bin/sh и нажимаем сtrl-x для загрузки в систему●В целом на этом все, Вы попали в систему. Но есть один нюанс. Рутовая файловая система при этом монтируется в режиме Read-Only. Если вы хотите перемонтировать ее в режим Read-Write можно воспользоваться командой:

mount -o remount,rw /

Способ 2. rd.break
●В конце строки начинающейся с linux16 добавляем rd.break и нажимаем сtrl-x для загрузки в систему
●Попадаем в emergency mode. Наша корневая файловая система смонтирована (опять же в режиме Read-Only, но мы не в ней. Далее будет пример как попасть в нее и поменять пароль администратора:
 mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel
●После чего можно перезагружаться и заходить в систему с новым паролем. Полезно когда вы потеряли или вообще не имели пароль администратор.

Способ 3. rw init=/sysroot/bin/sh
●В строке начинающейся с linux16 заменяем ro на rw init=/sysroot/bin/sh и нажимаем сtrl-xдля загрузки в систему
●В целом то же самое что и в прошлом примере, но файловая система сразу смонтирована в режим Read-Write
●В прошлых примерах тоже можно заменить ro на rw


Установить систему с LVM, после чего переименовать VG

●Первым делом посмотрим текущее состояние системы:

vgs

●Нас интересует вторая строка с именем Volume Group
●Приступим к переименованию:

vgrename VolGroup00 OtusRoot

●Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое название на новое. Файлы есть в репозитории.
●Пересоздаем initrd image, чтобы он знал новое название Volume Group

mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

●После чего можем перезагружаться и если все сделано правильно успешно грузимся с новым именем Volume Group и проверяем:

vgs

Добавить модуль в initrd

Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы добавить свой модуль создаем там папку с именем 01test:

mkdir /usr/lib/dracut/modules.d/01test

В нее поместим два скрипта:
1.module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh
2.test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик. Примеры файлов в репозитории.

●Пересобираем образ initrd


mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

Можно проверить/посмотреть какие модули загружены в образ:

lsinitrd -m /boot/initramfs-$(uname -r).img | grep test

●После чего можно пойти двумя путями для проверки:
○Перезагрузиться и руками выключить опции rghb и quiet и увидеть вывод
○Либо отредактировать grub.cfg убрав эти опции
●В итоге при загрузке будет пауза на 10 секунд и вы увидите пингвина в выводе терминала


