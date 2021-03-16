# HTTPS_server

------ГЕНЕРАЦИЯ СЕРТЕФИКАТА------
Для работы TLS/SSL использует комбинацию открытого сертификата и закрытого ключа. Закрытый ключ хранится на сервере и не разглашается. 
SSL-сертификат используется открыто и доступен всем пользователям, запрашивающим контент.

Чтобы создать самоподписанный сертификат и ключ, создайте следующую директорию:

sudo mkdir /etc/nginx/ssl/

Затем запустите команду:

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nel.key -out /etc/nginx/ssl/ned.crt

Команда задаст ряд вопросов.
Заполните появившиеся поля данными о сервере, которые будут отображаться в сертификате.

Самой важной строкой является Common Name (введите полное доменное имя  сервера (FQDN) или свое имя). Как правило, в эту строку вносят доменное имя, с которым нужно связать сервер. В случае если доменного имени нет, внесите в эту строку IP-адрес сервера. В целом эти поля выглядят примерно так:

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


