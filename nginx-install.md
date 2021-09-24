# Установка и базовая настройка Nginx

## Варианты установки Nginx
+ Из пакетного менеджера
+ Из исходников
+ В Docker-контейнере

## Установка Nginx

### Установка Nginx из пакетного менеджера в Debian-подобных дистрибутивах

Пакет доступен в стандартных репозиториях:

```bash

sudo apt install nginx
```
После установки нужно настроить файервол. Если это UFW, то следующим образом

```bash
sudo ufw allow 'Nginx HTTP'

# Либо 
sudo ufw allow 80
sudo ufw allow 443
```
Если в качестве файервола используется iptables, то 

Чтобы проверить работает ли веб-сервер:

```bash
sudo systemctl status nginx
```

При установке из пакетов, Nginx воспринимается демоном и управляется systemctl стандартными методами

```bash
sudo systemctl stop nginx
sudo systemctl start nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl disable nginx
sudo systemctl enable nginx
```

### Установка Nginx из пакетного менеджера в CENTOS

Для начал нужно установить связь с epel-репозиторием

```bash
yum install epel-release
```
Сама установка

```bash
yum install nginx
```

Настройка файервола

```bash
firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --zone=public --permanent --add-service=https

# Перезапуск файервола
firewall-cmd --reload
```

В Centos также используется systenctl, поэтому команды управления демноном аналогичны.

### Установка Nginx из пакетного менеджера в Arch Linux

Пакет устанавливается командой

```bash 
sudo pacman -S nginx
```

### Установка из исходников

Не могу назвать для себя установку из исходников рациональным времяпровождением по причине того, что настолько тонкая настройка для меня избыточна, да и опыт оказался не особо успешным. Если кратко, то я выполнил следующую последовательность команд:

## Базовая настройка Nginx

Предполагаем, что базовой настройкой будет считаться установка LEMP-стека (Linux, (e)N(dj)inx, MySQL, PHP)

### Устанока PHP 8

PPA с пакетами PHP 8:

```bash
apt-get install -y software-properties-common
apt-add-repository ppa:ondrej/apache2 -y
apt-add-repository ppa:ondrej/php -y
```

Сам интерпретатор PHP + пакеты. Копируй не сразу!!!!! Почитай до конца подзаголовка!!!!

```bash
sudo apt install -y php8.0-cli php8.0-dev php8.0-pgsql php8.0-sqlite3 php8.0-gd php8.0-curl php8.0-memcached php8.0-imap php8.0-mysql php8.0-mbstring php8.0-xml php8.0-imagick php8.0-zip php8.0-bcmath php8.0-soap php8.0-intl php8.0-readline php8.0-common php8.0-pspell php8.0-tidy php8.0-xsl php8.0-opcache php8.0-apcu
```

Есть ещё такой ряд команд (приведено для наглядности):
+ **php-common** 
Это пакет php, который включает в себя общие файлы для пакетов PHP, этот пакет содержит общие утилиты, общие для всех упакованных версий PHP. Пакет php-common содержит файлы, используемые как пакетом php, так и пакетом php-cli.
<br>

   ```bash
   sudo apt install php8.0-common
   ```

+ **php-cli**
Интерфейс командной строки. CLI позволяет запускать программы на PHP не через привычную нам клиент-серверную архитектуру, а как простые программы в командной строке.
<br>

   ```bash
   sudo apt install php8.0-cli
   ```

+ **php-curl**
Пакет, который позволяет работать с HTTP запросами по URL в PHP. 
<br>

+ **php-intl**
Модуль интернационализации. Он связан с библиотекой ICU - это средство, позволяющее работать с локалью.
<br>

+ **php-mysql**
Модуль для работы с СУБД MySQL. Данная система обладает своим API и модуль php-mysql с ним взаимодействует.
<br>

+ **php-readline**
Модуль также необходим для работы с консольной версией языка.
<br>

+ **php-xml**
Модуль для работы с XML-файлами. Модуль для JSON уже входит в пакет common
<br>

+ **php-mbstring**
Модуль для обработки многобайтных кодировок, например та же UTF-8. Русские (кирилические) символы занимают 2 байта.
<br>

Необходимые модули всегда можно установить отдельно, поэтому нет смысла устанавливать всё бездумно - в конце концов так или иначе это влияет на быстродействие сайта. 

Требуемые модули можно установить одной командой:

```bash
sudo apt-get install -y software-properties-common && sudo apt-add-repository ppa:ondrej/apache2 -y && sudo apt-add-repository ppa:ondrej/php -y && sudo apt install -y php8.0-{common,cli,curl,intl,mysql,readline,xml,mbstring}
```
#### Установка FPM

FPM - это менеджер процессов FastCGI. Подробнее можно прочитать [здесь](https://www.php.net/manual/ru/install.fpm.php). Устанавливается тоже просто:

```bash
sudo apt install php8.0-fpm
```
Я бы даже не стал выделять его в подзаголовок, но с учётом его жуткой важности и ключевой необходимости, всё-таки отметил именно так. В дальнейшем я собираюсь установить Apache + Nginx на отдельную виртуальную машину и сравнить, что же быстрее - два сервера или Nginx + FPM. Вообще, говорят, что второе, но я это проверю. Заодно и попрактикуюсь. А то эти сервера меня уже порядком задерживают: нужно разобраться с ними настолько чётко, что смогу проводить с закрытими глазами самую детальную настройку. 

Что касается настроки веб-сервера для работы с FPM - об этом после разбора конфигов.

#### В итоге для установки PHP:

```bash
sudo apt-get install -y software-properties-common && sudo apt-add-repository ppa:ondrej/apache2 -y && sudo apt-add-repository ppa:ondrej/php -y && sudo apt install -y php8.0-{common,cli,curl,intl,mysql,readline,xml,mbstring,fpm}
```

## Конфиги Nginx

Самый главный конфиг данного веб-сервера - это файл /etc/nginx/nginx.conf

После установки программы он будет выглядеть следующим образом

```bash
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
# 
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}

```

Из него становится ясно, что веб-сервер работает от пользователя www-data, количество воркер-процессов задаётся автоматически (должно соответствовать количеству ядер процессора), прописан путь к PID-файлу и первый инклюд, который сообщает, что нужно подгрузить конфигурацию модулей (плагинов) Nginx'а.

Здесь стоит уточнить, что в Nginx есть понятие контекста выполнения - это фактически обасть видимости. Я думаю по структуре файла очевидно, что есть какая-то вложенность в настройках. Всё-всё-всё выполняется в контексте main - он напрямую не прописывается, но этот факт подразумевается. Следующими по иерархии идут сразу два: http и events. Как раз следующий абзац описывает "события". Настройка по дефолту всего одна - это количество обрабатываемых подключений одним воркером. Можно указать 2048 - Nginx должен справиться.

Далее идёт описание контекста http-конекста. Здесь мы видим базовые настройки, настройки шифрования, логгирования, сжатия и дана затравка для виртуальных хостов. Далее закоментирован блок mail - это намёк на то, что этот комбайн (Nginx) в состоянии работать и с почтовыми протоколами. Но я пока не буду его использовать в таком качестве. Позднее я разберу подробно работу с postfix - эта штука требует некторого осмысления.

Возвращаясь к контексту http: базовые настройки я пока не буду трогать - они касаются параметров соединения, параметров передаваемых пакетов и хэширования. Также в конце есть подключение файла со списком расширений, чтобы Nginx знал какое расширение как воспринимать. Остальные настройки разбирать тоже пока не буду - мне нужно базово запустить работу сервера с PHP. А для этого нужно последовать по пути, указанному в нижнем открытом инклюде - к папке /etc/nginx/sites-enabled/ . 

Здесь меня ждёт символическая ссылка на файл в папке /etc/nginx/sites-available/ . Очень удобная вещь, эти симлинки - удалил ссылку и файл конфига остался не тронутым, что позволит его спокойно подключить в дальнейшем. А сам файл находится в вышеуказанной папке. При установке как Nginx, так и Apache создаётся папка /var/www/html/ в которой находится сайт-заглушка с приветствием. Как раз этот сайт и конфигурирует файл /etc/nginx/sites-available/default . Когда я буду создавать свои виртуальные хосты, я уже буду писать **\<тра-та-та>.conf** Он, соответственно, выглядит так:

```bash
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	# SSL configuration
	#
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	#
	# Note: You should disable gzip for SSL traffic.
	# See: https://bugs.debian.org/773332
	#
	# Read up on ssl_ciphers to ensure a secure configuration.
	# See: https://bugs.debian.org/765782
	#
	# Self signed certs generated by the ssl-cert package
	# Don't use them in a production server!
	#
	# include snippets/snakeoil.conf;

	root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}

	# pass PHP scripts to FastCGI server
	#
	#location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
	#	fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	#}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {
	#	deny all;
	#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#	listen 80;
#	listen [::]:80;
#
#	server_name example.com;
#
#	root /var/www/example.com;
#	index index.html;
#
#	location / {
#		try_files $uri $uri/ =404;
#	}
#}
```

Контекст server, прослушивается порт 80 со всех адресов, если не указано иначе, то конфигурация сервера из этого файла будет применяться ко всем запросам на данную машину. 

А конфигурация такая: 
+ Папка, откуда будет браться контент для отдачи: /var/www/html .
+ Если в взапросе открыто не указан файл, то по порядку будет производиться поиск следующих файлов: index index.html index.htm index.nginx-debian.html
+ Имя сервера по-дефолту. Опять же под сервером (**server**_name) здесь подразумевается именно виртуальный хост, то есть то, к чему относится описание данного контекста.
+ Дальше находится описание контекста location и данному контексту в качестве признака дан знак слеша. Что  значит "ко всем запросам в корень сайта". И для всех запросов нужно из URI искать имеющийся файл, либо отдавать 404 ответ.

Уже эта конфигурация позволяет Nginx отдавать статику.

## Настройка работы Nginx с PHP-FPM

Нужно для начала проверить по какому пути находится unix-сокет PHP-FPM. Стоит поискать в /var/run/php/ . Мне нужен файл php8.0-fpm.sock. Важно правильно прописать путь к сокету в соответствующем месте конфигов, иначе Nginx выдаёт 502 ответ: Bad Gateway. Собственно о подключении. Как говорилось ранее, в файле /etc/nginx/sites-available/default на момент установки прописан минимум для статики, изменим это.

Для начала допишем в список индексных файлов index.php сразу после имени без расширения. Затем, стоит прописать имя сервера. Если отсутствует домен, то за него сойдёт постоянный IP сервера (глобальный, локальный - без разницы).

В блоке "location ~ \\.php$ " нужно раскомментировать строки 

```bash
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
```
Я привёл последнюю строку в уже исправленном под мои нужды виде. Об этом было в первом абзаце подзаголовка.

Понятно, что само объявление контекста и закрывающую скобку тоже нужно раскомментировать. Потом проверяем:

```bash
sudo nginx -t
```

Ну и всё, если в /var/www/html добавить index.php с любым PHP-кодом, Nginx будет отдавать результаты работы скрипта.
