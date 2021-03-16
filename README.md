# HTTPS_server

Данный проект содержит в себе настроенные конфиги nginx на базе Centos8 для создания защищенного соединения https посредством использования
самоподписнного SSL сертификата.

------ГЕНЕРАЦИЯ СЕРТЕФИКАТА------
Для работы TLS/SSL использует комбинацию открытого сертификата и закрытого ключа. Закрытый ключ хранится на сервере и не разглашается. 
SSL-сертификат используется открыто и доступен всем пользователям, запрашивающим контент.

Чтобы создать самоподписанный сертификат и ключ, создайте следующую директорию:

    sudo mkdir /etc/nginx/ssl/

Затем запустите команду:

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nel.key -out /etc/nginx/ssl/ned.crt

Команда задаст ряд вопросов.
Заполните появившиеся поля данными о сервере, которые будут отображаться в сертификате.

Самой важной строкой является Common Name (введите полное доменное имя  сервера (FQDN) или свое имя). Как правило, в эту строку 
вносят доменное имя, с которым нужно связать сервер. В случае если доменного имени нет, внесите в эту строку IP-адрес сервера. 
В целом эти поля выглядят примерно так:

Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York City
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Bouncy Castles, Inc.
Organizational Unit Name (eg, section) []:Ministry of Water Slides
Common Name (e.g. server FQDN or YOUR name) []:server_IP_address
Email Address []:admin@your_domain.com

Файлы ключа и сертификата будут помещены в каталог /etc/nginx/ssl.
При использовании OpenSSL нужно также создать ключи Диффи-Хеллмана.
Для этого введите:

    sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

------НАСТРОЙКА КОНФИГОВ------
После нужно перенести файлы проект в директорию /etc/nginx/
В файле nginx.conf в первом блоке server нужно вставить ip либо домен вашего сервера.

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  your_ip_or_domen;
        root         /usr/share/nginx/html;
        return 302 https://$your_ip_or_domen$request_uri;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;


В файле ssl-params.conf нужно вставить ваш DNS распознаватель для восходящего канала запросов. 
В руководстве для этого используется Google.

    # from https://cipherli.st/
    # and https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
    ssl_ecdh_curve secp384r1;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s; <-- вместо гугла ваш DNS распознаватель.
    resolver_timeout 5s;
    add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

------НАСТРОЙКА БРАНДМАУЭРА------
Брандмауэр firewalld установлен по умолчанию в некоторых дистрибутивах Linux, в том числе во многих образах CentOS 8. 
Однако вам может потребоваться установить firewalld самостоятельно:

    sudo dnf install firewalld

После установки firewalld вы можете активировать службу и перезагрузить сервер. Помните, что в случае активации службы 
firewalld она будет запускаться при загрузке системы. Прежде чем настраивать такое поведение, лучше создать правила 
брандмауэра и воспользоваться возможностью протестировать их во избежание потенциальных проблем.

    sudo systemctl enable firewalld
    sudo systemctl start firewalld

После перезагрузки сервера брандмауэр запускается, сетевые интерфейсы помещаются в настроенные зоны (или возвращаются 
в заданные по умолчанию зоны), и все правила зон применяются к соответствующим интерфейсам.

Чтобы проверить работу и доступность службы, можно использовать следующую команду:

    sudo firewall-cmd --state

Это показывает, что наш брандмауэр запущен и работает с конфигурацией по умолчанию.
Конфигурацию зоны по умолчанию можно распечатать с помощью следующей команды:

    sudo firewall-cmd --list-all

    public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0 eth1
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
  
  На всякий случай можно повторно установить интерфейс
    
    sudo firewall-cmd --zone=public --change-interface=eth0

  Нaм же нужно добавить в активные сервисы разрешения http и https.
  

