Настройка окружения
===================

Заметки
-------

Получение прав root:

    sudo -s

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
    sudo apt install trash-cli # Утилита для очистки корзины командой `trash-empty`.
    #sudo apt install make libssl-dev libghc-zlib-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip
    # Настройки окружения.
    sudo ln -s /usr/bin/python3.12 /usr/bin/py
    sudo ln -s /usr/bin/python3.12 /usr/bin/python
    # Сразу можно Git настроить.
    # Конфигурация репозитория находится в `.git/config`.
    # Пользовательская конфигурация находится в `~/.gitconfig`.
    # Системная конфигурация `/etc/gitconfig` настраивается через sudo.
    git config --list
    git config --global --list
    git config --system --list
    # Установка системных настроек.
    sudo git config --system core.ignorecase false
    sudo git config --system core.autocrlf false
    sudo git config --system core.safecrlf warn
    sudo git config --system core.symlinks true

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

    apt-cache search openjdk # Посмотреть доступные компоненты.
    # Можно использовать default-jre или default-jdk (содержит default-jre) пакеты.
    sudo apt install default-jdk
    # Для Sonatype Nexus дополнительно нужно поставить JDK 17.
    sudo apt install openjdk-17-jdk
    # Установка переменных окружения через /etc/environment или /etc/environment.d/90java.conf.
    JAVA_HOME="/usr/lib/jvm/default-java"
    #INSTALL4J_JAVA_HOME="/usr/lib/jvm/java-17-openjdk-arm64"
    # Для новых версий Nexus используется встроенная JVM, поэтому устанавливать INSTALL4J_JAVA_HOME скорее всего не нужно.

13. [Опционально] Настройка VPN на OpenWrt:

    # Есть несколько подходящих вариантов VPN: L2TP/IPsec (самый распространённый вариант) и OpenVPN (требует стороннее программное обеспечение на стороне клиента).
    # Основной документ настройки OpenVPN сервера находится по https://openwrt.org/docs/guide-user/services/vpn/openvpn/server ссылке.
    # Дополнительный документ по настройке OpenVPN находится по https://openwrt.org/ru/doc/howto/vpn.openvpn ссылке.
    # Информацию по настройке клиентов OpenVPN можно найти по https://openwrt.org/docs/guide-user/services/vpn/openvpn/client-luci ссылке.
    # Далее настроим самый простой вариант с OpenVPN сервером для роутера на OpenWrt.
    opkg update
    opkg install openvpn-openssl openvpn-easy-rsa luci-app-openvpn
    # Далее в консоли выполняем необходимые команды.
    VPN_DIR="/etc/openvpn"
    VPN_PKI="/etc/easy-rsa/pki"
    VPN_PORT="1194"
    VPN_PROTO="udp"
    VPN_POOL="192.168.9.0 255.255.255.0"
    VPN_DNS="${VPN_POOL%.* *}.1"
    VPN_DN="$(uci -q get dhcp.@dnsmasq[0].domain)"
    VPN_SERV="EXTERNAL_IP_OR_DNS_OF_THE_ROUTER"
    # Используем новую версию EasyRSA для исключения некоторых ошибок.
    wget -U "" -O /tmp/easyrsa.tar.gz https://github.com/OpenVPN/easy-rsa/releases/download/v3.1.7/EasyRSA-3.1.7.tgz
    tar -z -x -f /tmp/easyrsa.tar.gz
    # Конфигурация параметров.
    cat << EOF > /etc/profile.d/easy-rsa.sh
    export EASYRSA_PKI="${VPN_PKI}"
    export EASYRSA_TEMP_DIR="/tmp"
    export EASYRSA_CERT_EXPIRE="3650"
    export EASYRSA_BATCH="1"
    alias easyrsa="/root/EasyRSA-3.1.7/easyrsa"
    EOF
    # Запуск скрипта.
    . /etc/profile.d/easy-rsa.sh
    # Remove and re-initialize PKI directory.
    easyrsa init-pki
    # Generate DH parameters.
    easyrsa gen-dh
    # Create a new CA.
    easyrsa build-ca nopass
    # Generate server keys and certificate.
    easyrsa build-server-full server nopass
    openvpn --genkey tls-crypt-v2-server ${EASYRSA_PKI}/private/server.pem
    # Создание ключей и сертификата для клиента (повторить для каждого клиента).
    easyrsa build-client-full client nopass
    openvpn --tls-crypt-v2 ${EASYRSA_PKI}/private/server.pem --genkey tls-crypt-v2-client ${EASYRSA_PKI}/private/client.pem
    # Configure firewall.
    uci rename firewall.@zone[0]="lan"
    uci rename firewall.@zone[1]="wan"
    uci del_list firewall.lan.device="tun+"
    uci add_list firewall.lan.device="tun+"
    uci -q delete firewall.ovpn
    uci set firewall.ovpn="rule"
    uci set firewall.ovpn.name="Allow-OpenVPN"
    uci set firewall.ovpn.src="wan"
    uci set firewall.ovpn.dest_port="${VPN_PORT}"
    uci set firewall.ovpn.proto="${VPN_PROTO}"
    uci set firewall.ovpn.target="ACCEPT"
    # Применяем настройки и перезагружаем службу.
    uci commit firewall
    service firewall restart
    # Configure VPN service and generate client profiles.
    umask go=
    VPN_DH="$(cat ${VPN_PKI}/dh.pem)"
    VPN_CA="$(openssl x509 -in ${VPN_PKI}/ca.crt)"
    ls ${VPN_PKI}/issued | sed -e "s/\.\w*$//" | while read -r VPN_ID
    do
    VPN_TC="$(cat ${VPN_PKI}/private/${VPN_ID}.pem)"
    VPN_KEY="$(cat ${VPN_PKI}/private/${VPN_ID}.key)"
    VPN_CERT="$(openssl x509 -in ${VPN_PKI}/issued/${VPN_ID}.crt)"
    VPN_EKU="$(echo "${VPN_CERT}" | openssl x509 -noout -purpose)"
    case ${VPN_EKU} in
    (*"SSL server : Yes"*)
    VPN_CONF="${VPN_DIR}/${VPN_ID}.conf"
    cat << EOF > ${VPN_CONF} ;;
    user nobody
    group nogroup
    dev tun
    port ${VPN_PORT}
    proto ${VPN_PROTO}
    server ${VPN_POOL}
    topology subnet
    client-to-client
    keepalive 10 60
    persist-tun
    persist-key
    push "dhcp-option DNS ${VPN_DNS}"
    push "dhcp-option DOMAIN ${VPN_DN}"
    push "redirect-gateway def1"
    push "persist-tun"
    push "persist-key"
    <dh>
    ${VPN_DH}
    </dh>
    EOF
    (*"SSL client : Yes"*)
    VPN_CONF="${VPN_DIR}/${VPN_ID}.ovpn"
    cat << EOF > ${VPN_CONF} ;;
    user nobody
    group nogroup
    dev tun
    nobind
    client
    remote ${VPN_SERV} ${VPN_PORT} ${VPN_PROTO}
    auth-nocache
    remote-cert-tls server
    EOF
    esac
    cat << EOF >> ${VPN_CONF}
    <tls-crypt-v2>
    ${VPN_TC}
    </tls-crypt-v2>
    <key>
    ${VPN_KEY}
    </key>
    <cert>
    ${VPN_CERT}
    </cert>
    <ca>
    ${VPN_CA}
    </ca>
    EOF
    done
    service openvpn restart
    ls ${VPN_DIR}/*.ovpn
    # Далее копируем сформированные профили клиентов для использования на удалённых машинах /etc/openvpn/*.ovpn и перезагружаем роутер.
    reboot

14. [Опционально] Создание сертификата для сервисов:

    # В файле `/etc/gitlab/gitlab.rb` необходимо заменить адрес на https и порт опционально.
    # Конфигурационный файл `home.conf` для генерации сертификата можно скачать из моего основного репозитория.
    # Генерация приватного ключа и сертификата.
    mkdir ~/ssl
    cd ~/ssl
    openssl req -x509 -days 36500 -config home.conf -keyout home.key -out home.crt [-extensions 'v3_req']
    openssl pkcs12 -export -out home.pfx -inkey home.key -in home.crt # Генерировать pfx можно без пароля.
    # Можно игнорировать ошибки самоподписанного сертификата: Domain name does not end with a valid public suffix (TLD).
    # Поле `subjectAltName`, видимо, не поддерживает маску c доменом первого уровня: NET::ERR_CERT_COMMON_NAME_INVALID.
    # Наверное, маска работает при наличии домена второго уровня и распространяется на домены третьего.
    # Поэтому в конфигурационный файл сертификата пришлось добавлять каждый используемый домен.

15. Установка и настройка Samba:

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
    (261)    write list = virtualmode, username
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
    # Добавление нового пользователя.
    sudo adduser --no-create-home --disabled-password --disabled-login username
    sudo smbpasswd -a username
    # Подключиться к общему ресурсу можно из проводника по `\\media.lan\Shared` адресу.
    # Если логин и пароль вашего пользователя Windows совпадает с логином и паролем в Samba, то вход произойдёт без запроса учётных данных.

16. Установка Universal Media Server:

    # Установка дополнительных компонентов.
    sudo apt-get install mediainfo dcraw vlc-bin mplayer mencoder
    # tsMuxeR можно скачать с https://github.com/justdan96/tsMuxer и установить.
    #sudo cp tsMuxeR /usr/bin/
    sudo cp /mnt/ssd1/Home/Distrib/Linux/UMS/tsMuxeR /usr/bin/
    sudo chmod 755 /usr/bin/tsMuxeR
    sudo ln -s /usr/bin/tsMuxeR /usr/bin/tsmuxer
    # [Опционально] Для ARM процессора лучше пересобрать.
    sudo apt-get install build-essential ninja-build cmake checkinstall # Для ARM необходимо собрать вручную `g++-multilib`.
    sudo apt-get install libc6-dev libfreetype6-dev zlib1g-dev
    # Необходимо разархивировать сервер в то место, где он будет работать после установки.
    #sudo cp ums-Linux-$VERSION.tgz /opt/
    #sudo tar xzvf ums-Linux-$VERSION.tgz
    sudo cp /mnt/ssd1/Home/Distrib/Linux/UMS/UMS-Linux-14.9.0-arm64.tgz /opt/
    sudo tar xzvf /opt/UMS-Linux-14.9.0-arm64.tgz -C /opt/
    sudo mv /opt/ums-14.9.0 /opt/ums
    # Наверное, лучше сменить владельца и группу, под которым будет запускаться приложение.
    #sudo chown -R virtualmode:virtualmode /opt/ums-$VERSION
    sudo chown -R virtualmode:virtualmode /opt/ums
    # [Опционально] Первый запуск сервера (UMS should NOT be run as root).
    cd ums-$VERSION
    ./UMS.sh
    Ctrl+C
    # Не понятно, как правильно устанавливать сервис и под каким пользователем. В итоге установил `ums.service` в `/etc/systemd/system` и включил автозапуск.
    sudo cp /mnt/ssd1/Home/Distrib/Linux/UMS/ums.service /etc/systemd/system/
    sudo systemctl daemon-reload
    sudo systemctl enable ums
    sudo reboot
    # Сервис будет доступен по адресу `http://media.lan:9001`.
    # Файл сервиса `ums.service`.
    [Unit]
    Description=UMS
    DefaultDependencies=no
    After=network.target

    [Service]
    Type=simple
    User=virtualmode
    Group=virtualmode
    ExecStart=/opt/ums/UMS.sh
    TimeoutStartSec=0
    RemainAfterExit=yes
    Environment="UMS_MAX_MEMORY=500M"

    [Install]
    WantedBy=default.target

17. Установка Sonatype Nexus:

    sudo tar -zxvf nexus-unix-aarch64-3.78.1-02.tar.gz -C /opt
    sudo mv /opt/nexus-3.78.1-02/ /opt/nexus/
    sudo chown -R virtualmode:virtualmode /opt/nexus/ /opt/sonatype-work/
    sudo vim /opt/nexus/bin/nexus.rc # Заменить строку без комментария на `run_as_user="virtualmode"`.
    # Файл сервиса `nexus.service`.
    [Unit]
    Description=Nexus service
    After=network.target
    [Service]
    Type=forking
    LimitNOFILE=65536
    ExecStart=/opt/nexus/bin/nexus start
    ExecStop=/opt/nexus/bin/nexus stop
    User=virtualmode
    Restart=on-abort
    [Install]
    WantedBy=multi-user.target
    # Подготовка сервиса.
    sudo vim /etc/systemd/system/nexus.service
    sudo systemctl daemon-reload
    sudo systemctl enable nexus
    # Загрузка Nexus занимает в среднем 2-3 минуты.
    # Готовность программы можно проверить в файле nexus.log, найдя строку Started Sonatype Nexus OSS:
    tail -f /opt/sonatype-work/nexus3/log/nexus.log
    # Зайти на сервис можно по адресу `http://media.lan:8081` для настройки.
    cat /opt/sonatype-work/nexus3/admin.password
    # После получения пароля необходимо зайти в учётную запись `admin` и завершить настройку.
    # Дополнительно можно ограничить память JVM, заменив стандартные значения на `-Xms500m` и `-Xmx500m`.
    vim /opt/nexus/bin/nexus.vmoptions
    # Дополнительно можно закомментировать строки.
    #-XX:+LogVMOutput
    #-XX:LogFile=../sonatype-work/nexus3/log/jvm.log

18. Установка GitLab:

    # Т.к. исполняемые файлы GitLab по умолчанию ставятся в `/opt/gitlab`, а данные в `/var/opt/gitlab`, то есть смысл попробовать смонтировать `/var/opt` на другое устройство.
    # Пишут, что есть файл `/etc/gitlab/gitlab.rb` настроек, в котором можно сменить путь для данных, но непонятно, насколько это правильно.
    # Для того, чтобы одно устройство смонтированть на несколько путей, необходимо использовать параметр `-o bund` для `mount` или добавить запись в `/etc/fstab`.
    # Необходимо, чтобы путь `/mnt/ssd0/var/opt` уже существовал.
    /mnt/ssd0/var/opt /var/opt none defaults,bind,nofail 0 2
    # После перезагрузки выполнить если необходимо.
    sudo apt-get update
    sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
    # Можно установить почтовый сервер опционально.
    sudo apt-get install -y postfix bsd-mailx
    # Установил как `Local only` с доменом `mail.lan`.
    # Скачиваем скрипт установки и запускаем его. Для Community Edition - ce суффикс, для Enterprise Edition - ee.
    cd /tmp
    curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
    # Будут добавлены репозитории GitLab, после чего можно продолжить установку.
    sudo apt install gitlab-ce
    # ВНИМАНИЕ! Такой вариант установки может не работать на некоторых операционных системах, для которых нет требуемой версии.
    # Поэтому можно скачать пакет вручную на https://packages.gitlab.com/gitlab/gitlab-ce.
    # В моём случае это https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/noble/gitlab-ce_17.9.2-ce.0_arm64.deb версия.
    # На странице есть вариант установки: sudo apt-get install gitlab-ce=17.9.1-ce.0
    # Лучше скачать второй командой.
    wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/noble/gitlab-ce_17.9.2-ce.0_arm64.deb/download.deb
    sudo dpkg -i gitlab-ce_17.9.2-ce.0_arm64.deb
    # После установки необходимо настроить адрес и порт в файле `sudo vim /etc/gitlab/gitlab.rb`.
    external_url 'https://gitlab.lan'.
    # Можно указать порт и заменить на https, тогда GitLab создаст сертификат с помощью Let’s Encrypt.
    # Если необходимо обновить ключ вручную, то сначала удаляется каталог, если уже был создан.
    sudo rm -rf /etc/gitlab/ssl/
    # Копируем сертификаты с именем домена обязательно.
    sudo mkdir -p /etc/gitlab/ssl
    sudo chmod 755 /etc/gitlab/ssl
    sudo cp ~/ssl/home.crt /etc/gitlab/ssl/gitlab.lan.crt
    sudo cp ~/ssl/home.key /etc/gitlab/ssl/gitlab.lan.key
    sudo cp ~/ssl/home.crt /etc/gitlab/ssl/pages.lan.crt
    sudo cp ~/ssl/home.key /etc/gitlab/ssl/pages.lan.key
    # Дополнительно можно оптимизировать GitLab для низкопроизводительных систем.
    # https://docs.gitlab.com/omnibus/settings/memory_constrained_envs/
    # https://labs.etsi.org/rep/help/administration/gitaly/configure_gitaly.md
    external_url 'https://gitlab.lan'
    pages_external_url 'https://pages.lan'

    pages_nginx['redirect_http_to_https'] = true

    # Custom optimization.
    puma['worker_processes'] = 0

    sidekiq['concurrency'] = 8

    prometheus_monitoring['enable'] = false

    gitlab_rails['env'] = {
      'MALLOC_CONF' => 'dirty_decay_ms:1000,muzzy_decay_ms:1000',
      'WORKHORSE_ARCHIVE_CACHE_DISABLED' => '1'
    }

    gitaly['configuration'] = {
      concurrency: [
        {
          'rpc' => "/gitaly.SmartHTTPService/PostReceivePack",
          'max_per_repo' => 3,
        }, {
          'rpc' => "/gitaly.SSHService/SSHUploadPack",
          'max_per_repo' => 3,
        },
      ],
      # Настройка является alpha-версией и нарушает возможность создавать проекты.
      #cgroups: {
      #  repositories: {
      #    count: 2,
      #    cpu_shares: 384,
      #    cpu_quota_us: 200000,
      #  },
      #  mountpoint: '/sys/fs/cgroup',
      #  hierarchy_root: 'gitaly',
      #  memory_bytes: 500000,
      #  cpu_shares: 384,
      #  cpu_quota_us: 200000,
      #},
    }

    gitaly['env'] = {
      'MALLOC_CONF' => 'dirty_decay_ms:1000,muzzy_decay_ms:1000',
      'GITALY_COMMAND_SPAWN_MAX_PARALLEL' => '2'
    }
    # Реконфигурация GitLab.
    sudo gitlab-ctl reconfigure
    # Заходим под `root` и паролем.
    sudo cat /etc/gitlab/initial_root_password
    # Меняем праоль, настраиваем всё необходимое. Система может подвисать, видимо, из-за каких-то первичных процессов настройки.
    # Репозитории находятся в каталоге `/var/opt/gitlab/git-data/repositories/@hashed`.

19. Настройка Postfix (требуется домен или сторонний SMTP):

    sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.default
    # Заменяем содержимое конфигурации `sudo vim /etc/postfix/main.cf`.
    compatibility_level=3.6
    relayhost =
    smtp_sasl_auth_enable = yes
    smtp_sasl_password_maps = hash:/etc/postfix/private/sasl_passwd
    smtp_sasl_security_options = noanonymous
    smtp_sasl_type = cyrus
    smtp_sasl_mechanism_filter = login
    smtp_sender_dependent_authentication = yes
    smtp_generic_maps = hash:/etc/postfix/private/generic
    sender_dependent_relayhost_maps = hash:/etc/postfix/private/sender_relay
    sender_canonical_maps = hash:/etc/postfix/private/canonical
    smtp_use_tls = yes
    myorigin = media.localdomain
    # Создаем папку `/etc/postfix/private`.
    sudo mkdir /etc/postfix/private
    # Создаем `sudo vim /etc/postfix/private/canonical`.
    @yandex.ru	virtualmode@yandex.ru
    # Создаём `sudo vim /etc/postfix/private/sender_relay`.
    @yandex.ru	smtp.yandex.ru
    # Создаём `sudo vim /etc/postfix/private/sasl_passwd`.
    [smtp.yandex.ru]	virtualmode@yandex.ru:PASSWORD
    # Для настройки отправителей и получателей можно использовать `sudo vim /etc/postfix/private/generic`.
    # В файле разрешено использование комментариев.
    @media.mail						@media.localdomain
    gitlab@gitlab.lan				gitlab@media.localdomain
    root@media.localdomain			virtualmode@yandex.ru
    virtualmode@media.localdomain	virtualmode@yandex.ru
    # Создаем Postfix lookup tables (db для созданных файлов).
    sudo postmap /etc/postfix/private/*
    sudo service postfix restart
    # Проверка отправки почты.
    echo "Text message" | mail -s "Mail subject" virtualmode@yandex.ru
    # Просмотр логов.
    cat /var/log/mail.log

20. Настройка отправки почты через SMTP `sudo vim /etc/gitlab/gitlab.rb`:

    # Оригинальная документация https://docs.gitlab.com/omnibus/settings/smtp/
    # Для полного отключения работы с почтой.
    gitlab_rails['gitlab_email_enabled'] = false
    # Настройка Yandex SMTP.
    # Использование сервиса возможно только с аккаунта с привязанным телефоном и добавленым логином с паролем для приложений в разделе настроек.
    # Также необходимо в настройках почтовых программ разрешить доступ почтовым клиентам IMAP и POP3.
    gitlab_rails['smtp_enable'] = true
    gitlab_rails['smtp_address'] = "smtp.yandex.ru"
    gitlab_rails['smtp_port'] = 465
    gitlab_rails['smtp_user_name'] = "yandex_smtp_app_login"
    gitlab_rails['smtp_password'] = "yandex_smtp_app_password"
    gitlab_rails['smtp_domain'] = "yourdomain_or_yandex.ru"
    gitlab_rails['gitlab_email_from'] = "real_mail@yandex.ru"
    gitlab_rails['smtp_authentication'] = "login"
    gitlab_rails['smtp_tls'] = true
    gitlab_rails['smtp_enable_starttls_auto'] = false
    gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
    # Реконфигурация GitLab.
    sudo gitlab-ctl reconfigure
    gitlab-ctl restart
    # Для отладки необходимо запустить консоль.
    sudo gitlab-rails console
    # Подождать, пока не появится приветствие и ввести необходимые команды.
    Notify.test_email('virtualmode@yandex.ru', 'Message Subject', 'Message Body').deliver_now
