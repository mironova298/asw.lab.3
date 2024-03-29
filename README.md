
Введение

Apache - это один из самых популярных веб-серверов, который разрабатывается и поддерживается Apache Software Foundation. Он является бесплатным и открытым исходным кодом, и доступен для использования на различных операционных системах, включая Windows, Linux и macOS.

Apache может быть использован для хостинга веб-сайтов, в том числе динамических сайтов с использованием серверных скриптов, таких как PHP, Python и Ruby. Он также может быть настроен для обработки статических файлов, таких как HTML, CSS и JavaScript.

Apache обеспечивает широкий набор функций и настроек для управления и оптимизации производительности веб-сервера. Это включает в себя настройку кэширования, обработку SSL-соединений, аутентификацию пользователей и многое другое.

Виртуальный хостинг - это способ размещения нескольких веб-сайтов на одном физическом сервере, где каждый сайт имеет свой уникальный доменное имя и может быть настроен независимо от других сайтов на сервере.

Виртуальный хостинг позволяет экономить ресурсы, так как несколько сайтов могут быть размещены на одном сервере, и каждый сайт может быть настроен и управляться независимо друг от друга. Кроме того, виртуальный хостинг облегчает процесс управления сервером, так как администратору сервера не нужно настраивать отдельный сервер для каждого сайта.

Цель работы:
Целью данной работы является ознакомление с запуском сайтов (виртуальных хостов) в контейнере. Также студент ознакомится со способом создания кластера контейнеров и рассмотрит установку сайтов на базе Wordpress.

В результате выполнения данной работы будет получен кластер из 3-х контейнеров:
 - __Apache HTTP server__ - контейнер принимает запросы и переправляет их серверу PHP-FPM;
 - __PHP-FPM__ - контейнер с сервером PHP-FPM;
 - __MariaDB__ - контейнер с серером баз данных.


Ход работы:
Создаю папку `asweb03`. В ней будет выполняться вся работа.

Создаю папки `database` - для базы данных, `files` - для хранения конфигураций и `site` - в данной папке будет расположен сайт.
 
Скачал [CMS Wordpress](https://wordpress.org/) и распаковал в папку `site. 


 

Создаю конфигурационный файл для Apache HTTP Server. Для этого в папке files создам еще одну папку httpd и выполню следующие команды:
# команда скачивает образ httpd и запускает на его основе контейнер с именем httpd
docker run -d --name httpd  httpd:2.4
 

# копируем конфигурационный файл из контейнера в папку .\files\httpd
docker cp httpd:/usr/local/apache2/conf/httpd.conf .\files\httpd\httpd.conf


 

# останавливаем контейнер httpd
docker stop httpd
 

# удаляем контейнер
docker rm httpd

 






В созданном файле .\files\httpd\httpd.conf раскоментирую строки, содержащие подключение расширений mod_proxy.so, mod_proxy_http.so, mod_proxy_fcgi.so.

 

В конфигурационном файле нахожу ServerName. Под ним добавляю следующие строки:
# определение доменного имени сайта
ServerName wordpress.localhost:80
# перенаправление php запросов контейнеру php-fpm
ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://php-fpm:9000/var/www/html/$1
# индексный файл
DirectoryIndex /index.php index.php
 

Также найду определение параметра DocumentRoot и задам ему значение /var/www/html, предварительно создав эти папки, как и в следующей за параметром строке.


 

Создаю файл Dockerfile.httpd со следующим содержимым:
 

Создаю файл Dockerfile.php-fpm со следующим содержимым:

 


Создаю файл Dockerfile.mariadb со следующим содержимым:
 


Создаю файл docker-compose.yml со следующим содержимым:

 



В папке лабораторной работы(asweb03) открываю консоль и выполню команду:
docker-compose build
На основе созданных определений докер построит образы сервисов, проект собирался за 340 секунд.


 
Открою страницу в браузере и произведу установку сайта.
 
 





 
# удалить контейнеры
docker-compose rm

# остановить контейнеры
docker-compose down
 
После перезапуска сайт работает!

Ответьте на следующие вопросы:

1.	Как скопировать файл из контейнера в хостовый компьютер?
2.	За что отвечает директива DocumentRoot в конфигурационном файле Apache HTTP Server?
3.	В файле docker-compose.yml папка database хоста монтируется в папку /var/lib/mysql контейнера mariadb. Для чего монтируют к контейнеру базы данных папку?





1. Для того, чтобы скопировать файл из контейнера в хостовый компьютер, можно использовать команду docker cp. Эта команда позволяет копировать файлы между контейнером и хостовой системой.
Шаги по копированию файла из контейнера в хостовый компьютер:
1.1.Узнать идентификатор контейнера, в котором находится нужный файл. Для этого выполните команду docker ps
1.2. Запустить команду docker cp, указав путь к файлу внутри контейнера и путь к месту назначения на хостовом компьютере.
1.3. Проверить, что файл успешно скопирован на хостовый компьютер. Для этого можно перейти в директорию, указанную в качестве места назначения на хостовом компьютере, и проверить наличие скопированного файла.

2. Директива DocumentRoot в конфигурационном файле Apache HTTP Server отвечает за определение корневой директории, откуда будут обслуживаться запросы клиентов. Когда Apache получает запрос на определенный URL, он будет искать соответствующий файл внутри директории, указанной в DocumentRoot.

3. Монтирование папки database хоста в папку /var/lib/mysql контейнера MariaDB в файле docker-compose.yml позволяет сохранить данные базы данных в постоянном хранилище на хостовой машине. Это позволяет сохранять данные между перезапусками контейнера и делает их доступными извне контейнера.

По умолчанию, все данные, созданные в контейнере, хранятся в файловой системе контейнера, которая по умолчанию не сохраняется между перезапусками контейнера. Это означает, что при каждом перезапуске контейнера данные будут потеряны. Монтирование папки хоста в контейнер позволяет избежать этой проблемы, поскольку данные будут сохраняться в постоянном хранилище на хостовой машине.

Также монтирование папки с данными базы данных в контейнер позволяет проще бэкапировать и восстанавливать данные. Например, можно сделать копию папки с данными на хостовой машине и использовать ее для восстановления базы данных при необходимости.





Вывод:
В результате выполнения данной работы я ознакомился с технологией запуска сайтов в контейнерах, научился создавать кластер контейнеров и устанавливать сайты на базе Wordpress. Также я получил практические навыки в работе с контейнеризацией и смогу применять их при создании и развертывании веб-приложений. Результаты работы могут быть полезны для дальнейшего изучения технологий виртуализации и автоматизации процессов разработки и развертывания приложений.


Библиография:
https://hub.docker.com/
https://wordpress.org/download/
https://moodle.usm.md/course/view.php?id=566


