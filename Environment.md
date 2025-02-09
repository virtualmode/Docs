Настройка окружения
===================

Медиасервер Ubuntu
------------------

 1. Можно установить Ubuntu с помощью Raspberry Pi Imager 1.8.5 или другим способом.
 2. Для резервного копирования карт памяти удобно использовать DiskGenius 5.6.1.1580 утилиту.
 3. Подойдёт как специальный `*.img` образ, так и `*.iso` формат (правда после записи выводит ошибку FAT32).
 4. В итоге лучше установить серверный образ, автоматически скачанный Raspberry Pi Imager утилитой.
 5. Имя машины можно выбрать коротким, например, `media`, чтобы потом использовать в OpenWRT адрес `media.lan` для доступа к сервисам.
 6. После установки подключаться возможно через SSH, если в настройках был введён публичный ключ.
 7. Необходимо поставить основное программное обеспечение:

    sudo apt update
    sudo apt install lm-sensors # Для использования сенсоров надо ввести `sensors`.
    sudo apt install ca-certificates apt-transport-https
    sudo apt install wget gpg # Ubuntu уже содержит.
    sudo apt install p7zip-rar p7zip-full
    sudo apt install build-essential # Базовые средства разработки.
    #sudo apt install make libssl-dev libghc-zlib-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip
    # Настройки окружения.
    sudo ln -s /usr/bin/python3.12 /usr/bin/py
    sudo ln -s /usr/bin/python3.12 /usr/bin/python

 8. [Опционально] Установка homebrew (не для ARM):

    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    (echo; echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"') >> /home/user/.bash_profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
    nano /home/linuxbrew/.linuxbrew/Homebrew/Library/Homebrew/cmd/shellenv.sh
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
    brew doctor
    brew update
    brew upgrade
    # В домашнем каталоге желательно в файл ~/.bash_profile добавить переменные окружения.
    export HOMEBREW_GIT_PATH=/usr/local/bin/git
    export DOTNET_CLI_TELEMETRY_OPTOUT=1
    export PATH=$PATH:/home/linuxbrew/.linuxbrew/bin/

 9. Поддержка USB-устройств:

    # Вероятно, для Ubuntu и OpenWrt наборы пакетов различаются.
    # Пакеты kmod-usb* отсутствуют в Ubuntu 24.10
    # Пакет usbutils уже установлен.
    # TODO Необходимо дописать раздел при необходимости.

10. Работа с разделами:

    # Пакеты kmod-fs* отсутствуют в Ubuntu 24.10
    # TODO Необходимо дописать раздел при необходимости.

11. Подключение SSD-накопителей:

    lsblk # Посмотреть список устройств и разделов.
    # Получить подробную информацию о имеющихся разделах.
    sudo blkid | grep -i nvme0n1p1
    # /dev/nvme0n1p1: LABEL="ssd0" UUID="0d1400ec-c5aa-4430-a3ff-ca3e764206ac" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="ssd0" PARTUUID="c02af95b-a607-4752-8988-f77da4acffbb"
    sudo blkid | grep -i nvme1n1p1
    # /dev/nvme1n1p1: LABEL="ssd1" UUID="41cd815b-75ca-43e1-a45d-d7abfada01a4" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="ssd1" PARTUUID="9e756a4b-bb5d-4d58-8d33-ac81208542ad"
    # Необходимо отредактировать /etc/fstab для автомонтирования.
    # <device>	<dir>		<type>	<options>	<dump>	<fsck>
    LABEL=ssd0	/mnt/ssd0	ext4	defaults	0		2
    LABEL=ssd1	/mnt/ssd1	ext4	defaults	0		2
    # <fsck> Используется программой fsck для определения того, нужно ли проверять целостность файловой системы.
    # Значение 1 следует указывать только для корневой файловой системы. Для остальных ФС, которые вы хотите проверять, используйте значение 2, которое имеет менее высокий приоритет.
    # Файловые системы, для которых в поле указано значение 0, не будут проверяться fsck.

12. Установка Java:

    # Можно использовать default-jre или default-jdk (содержит default-jre) пакеты.
    sudo apt install default-jdk

13. Установка и настройка Samba:

    sudo apt update
    sudo apt install samba
    # Далее редактируется файл кофнигурации.
    sudo vim /etc/samba/smb.conf
    # Необходимые поля (в скобках указан номер строки).
    ( 40) interfaces = lo wlan0
    ( 47) bind interfaces only = yes
    (251) # Custom shares.
    (252) [Share]
    (253)    comment = Guest share
    (254)    path = /mnt/ssd1/Shared
    (255) ;   public = yes
    (256)    browseable = yes
    (257)    read only = yes
    (258) ;   writeable = yes
    (259) ;   printable = no
    (260)    guest ok = yes
    (261)    write list = virtualmode
    (262) ;   guest account = guest
    (263) ;   create mode = 0777
    (264) ;   directory mask = 0777
    (265)
    (266) [Home]
    (267)    comment = Home share
    (268)    path = /mnt/ssd1/Home
    (269)    browseable = yes
    (270)    read only = yes
    (271)    guest ok = no
    (272)    write list = virtualmode
    # Разрешить доступ к Samba в файрволе.
    sudo ufw allow samba
    # Samba использует свои пароли для каждого пользователя системы. Если пользователя не существует в системе, то пароль не может быть задан.
    # Вручную необходимо задать пароли пользователей общих ресурсов.
    sudo smbpasswd -a username
    # Подключиться к общему ресурсу можно из проводника по `\\media.lan\Shared` адресу.
    # Если логин и пароль вашего пользователя Windows совпадает с логином и паролем в Samba, то вход произойдёт без запроса учётных данных.

14. Установка Universal Media Server:

    # Установка дополнительных компонентов.
    sudo apt-get install mediainfo dcraw vlc-bin mplayer mencoder
    # tsMuxeR можно скачать с https://github.com/justdan96/tsMuxer и установить.
    sudo cp tsMuxeR /usr/bin
    sudo chmod 755 /usr/bin/tsMuxeR
    sudo ln -s /usr/bin/tsMuxeR /usr/bin/tsmuxer
    # Для ARM процессора лучше пересобрать.
    sudo apt-get install build-essential ninja-build cmake checkinstall # Для ARM необходимо собрать вручную `g++-multilib`.
    sudo apt-get install libc6-dev libfreetype6-dev zlib1g-dev
    # Необходимо разархивировать сервер в то место, где он будет работать после установки.
    sudo cp ums-Linux-$VERSION.tgz /opt/
    sudo tar xzvf ums-Linux-$VERSION.tgz
    # Наверное, лучше сменить владельца и группу, под которым будет запускаться приложение.
    sudo chown -R virtualmode:virtualmode /opt/ums-$VERSION
    # Первый запуск сервера (UMS should NOT be run as root).
    cd ums-$VERSION
    ./UMS.sh
    Ctrl+C
    # Не понятно, как правильно устанавливать сервис и под каким пользователем. В итоге установил `ums.service` в `/etc/systemd/system` и включил автозапуск.
    sudo systemctl enable ums
    # Файл сервиса `ums.service`.
    [Unit]
    Description=UMS
    DefaultDependencies=no
    After=network.target

    [Service]
    Type=simple
    User=virtualmode
    Group=virtualmode
    ExecStart=/opt/ums-$VERSION/UMS.sh
    TimeoutStartSec=0
    RemainAfterExit=yes
    Environment="UMS_MAX_MEMORY=500M"

    [Install]
    WantedBy=default.target

15. Установка 