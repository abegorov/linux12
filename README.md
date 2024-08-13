# Systemd - создание unit-файла

## Задание

1. Написать **service**, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в **/etc/default**).
2. Установить **spawn-fcgi** и создать **unit**-файл (**spawn-fcgi.sevice**) с помощью переделки [init-скрипта](https://gist.github.com/cea2k/1318020).
3. Доработать **unit**-файл **Nginx** (**nginx.service**) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

## Реализация

Задание сделано на **almalinux/8** версии **v8.10.20240604** (в **Centos Stream 9** и **AlmaLinux 9** отсутствует пакет **spawn-fcgi**). **81** и **82** порты виртуальной машины проброшены на **127.0.0.1:8081** и **127.0.0.1:8082** на хосте. После загрузки запускается **Ansible Playbook** **[playbook.yml](https://github.com/abegorov/linux12/blob/main/playbook.yml)**, который последовательно запускает 4 роли:

1. **wathclog**. Устанавливает скрипт **/usr/local/bin/watchlog.sh**, который мониторит **/var/log/watchlog.log** раз в 30 секунд на предмет наличия ключевого слова, настраивает его, создаёт **service** и **timer**, а также сам лог файл (при его отсутствии).
2. **spawn-fcgi**. Устанавливает **spawn-fcgi** и создаёт **spawn-fcgi.sevice** для его запуска.
3. **nginx-multinstance** с параметром **first**. Поднимает **nginx@first.service** с конфигурацией в **/etc/nginx/nginx-first.conf** на **81** порту.
4. **nginx-multinstance** с параметром **second**. Поднимает **nginx@second.service** с конфигурацией в **/etc/nginx/nginx-second.conf** на **82** порту (порт добавляется в **http_port_t** политики **SELinux**).

## Запуск

Необходимо скачать **VagrantBox** для **almalinux/8** версии **v8.10.20240604** и добавить его в **Vagrant** под именем **almalinux/8/v8.10.20240604**. Сделать это можно командами:

```shell
curl -OL https://app.vagrantup.com/almalinux/boxes/8/versions/8.10.20240604/providers/virtualbox/amd64/vagrant.box
vagrant box add vagrant.box --name "almalinux/8/v8.10.20240604"
rm vagrant.box
```

## Проверка

1. **journalctl -b -r | grep 'I found word'**:

    ```text
    Aug 13 20:35:53 systemd-init root[12527]: Tue Aug 13 20:35:53 UTC 2024: I found word, Master!
    Aug 13 20:35:09 systemd-init root[12518]: Tue Aug 13 20:35:09 UTC 2024: I found word, Master!
    Aug 13 20:34:40 systemd-init root[12511]: Tue Aug 13 20:34:40 UTC 2024: I found word, Master!
    Aug 13 20:34:29 systemd-init root[12504]: Tue Aug 13 20:34:29 UTC 2024: I found word, Master!
    Aug 13 20:33:44 systemd-init root[12495]: Tue Aug 13 20:33:44 UTC 2024: I found word, Master!
    Aug 13 20:33:17 systemd-init root[12488]: Tue Aug 13 20:33:17 UTC 2024: I found word, Master!
    Aug 13 20:32:55 systemd-init root[12479]: Tue Aug 13 20:32:55 UTC 2024: I found word, Master!
    Aug 13 20:32:09 systemd-init root[12471]: Tue Aug 13 20:32:09 UTC 2024: I found word, Master!
    Aug 13 20:31:57 systemd-init root[12466]: Tue Aug 13 20:31:57 UTC 2024: I found word, Master!
    ```

    В **AlmaLinux 8** используется **systemd** версии **239**, в то время как параметр **FixedRandomDelay** был добавлен для **timer** в весии **247** (из-за этого такой разброс времени выполнения).

2. **systemctl status spawn-fcgi**:

    ```text
    ● spawn-fcgi.service - Spawn-fcgi startup service by Otus
      Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; enabled; vendor preset: disabled)
      Active: active (running) since Tue 2024-08-13 20:31:20 UTC; 1min 45s ago
    Main PID: 12010 (php-cgi)
        Tasks: 33 (limit: 5835)
      Memory: 30.8M
      CGroup: /system.slice/spawn-fcgi.service
              ├─12010 /usr/bin/php-cgi
              ├─12052 /usr/bin/php-cgi
              ├─12053 /usr/bin/php-cgi
              ├─12054 /usr/bin/php-cgi
              ├─12055 /usr/bin/php-cgi
              ├─12056 /usr/bin/php-cgi
              ├─12057 /usr/bin/php-cgi
              ├─12058 /usr/bin/php-cgi
              ├─12059 /usr/bin/php-cgi
              ├─12060 /usr/bin/php-cgi
              ├─12061 /usr/bin/php-cgi
              ├─12062 /usr/bin/php-cgi
              ├─12063 /usr/bin/php-cgi
              ├─12064 /usr/bin/php-cgi
              ├─12065 /usr/bin/php-cgi
              ├─12066 /usr/bin/php-cgi
              ├─12067 /usr/bin/php-cgi
              ├─12068 /usr/bin/php-cgi
              ├─12069 /usr/bin/php-cgi
              ├─12070 /usr/bin/php-cgi
              ├─12071 /usr/bin/php-cgi
              ├─12072 /usr/bin/php-cgi
              ├─12073 /usr/bin/php-cgi
              ├─12074 /usr/bin/php-cgi
              ├─12075 /usr/bin/php-cgi
              ├─12077 /usr/bin/php-cgi
              ├─12078 /usr/bin/php-cgi
              ├─12079 /usr/bin/php-cgi
              ├─12080 /usr/bin/php-cgi
              ├─12081 /usr/bin/php-cgi
              ├─12082 /usr/bin/php-cgi
              ├─12083 /usr/bin/php-cgi
              └─12084 /usr/bin/php-cgi
    ```

3. **systemctl status nginx@first && systemctl status nginx@second**:

    ```text
    ● nginx@first.service - The nginx HTTP and reverse proxy server
      Loaded: loaded (/etc/systemd/system/nginx@.service; enabled; vendor preset: disabled)
      Active: active (running) since Tue 2024-08-13 20:31:06 UTC; 46s ago
    Main PID: 9829 (nginx)
        Tasks: 2 (limit: 5835)
      Memory: 6.8M
      CGroup: /system.slice/system-nginx.slice/nginx@first.service
              ├─ 9829 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-first.conf
              └─12209 nginx: worker process

    Aug 13 20:31:06 systemd-init systemd[1]: Starting The nginx HTTP and reverse proxy server...
    Aug 13 20:31:06 systemd-init nginx[9810]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    Aug 13 20:31:06 systemd-init nginx[9810]: nginx: configuration file /etc/nginx/nginx.conf test is successful
    Aug 13 20:31:06 systemd-init systemd[1]: nginx@first.service: Can't open PID file /run/nginx-first.pid (yet?) after start: No such file or d>
    Aug 13 20:31:06 systemd-init systemd[1]: Started The nginx HTTP and reverse proxy server.
    Aug 13 20:31:20 systemd-init systemd[1]: Reloading The nginx HTTP and reverse proxy server.
    Aug 13 20:31:20 systemd-init systemd[1]: Reloaded The nginx HTTP and reverse proxy server.

    ● nginx@second.service - The nginx HTTP and reverse proxy server
      Loaded: loaded (/etc/systemd/system/nginx@.service; enabled; vendor preset: disabled)
      Active: active (running) since Tue 2024-08-13 20:31:18 UTC; 1min 12s ago
      Process: 12375 ExecReload=/bin/kill -s HUP $MAINPID (code=exited, status=0/SUCCESS)
    Main PID: 11619 (nginx)
        Tasks: 2 (limit: 5835)
      Memory: 9.0M
      CGroup: /system.slice/system-nginx.slice/nginx@second.service
              ├─11619 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx-second.conf
              └─12376 nginx: worker process

    Aug 13 20:31:18 systemd-init systemd[1]: Starting The nginx HTTP and reverse proxy server...
    Aug 13 20:31:18 systemd-init nginx[11615]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    Aug 13 20:31:18 systemd-init nginx[11615]: nginx: configuration file /etc/nginx/nginx.conf test is successful
    Aug 13 20:31:18 systemd-init systemd[1]: nginx@second.service: Can't open PID file /run/nginx-second.pid (yet?) after start: No such file or>
    Aug 13 20:31:18 systemd-init systemd[1]: Started The nginx HTTP and reverse proxy server.
    Aug 13 20:31:20 systemd-init systemd[1]: Reloading The nginx HTTP and reverse proxy server.
    Aug 13 20:31:20 systemd-init systemd[1]: Reloaded The nginx HTTP and reverse proxy server.
    ```

    Также на хосте должны открываться **URL**:

    - [127.0.0.1:8081](http://127.0.0.1:8081)
    - [127.0.0.1:8082](http://127.0.0.1:8082)
