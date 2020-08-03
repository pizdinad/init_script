# init_script

## Рекомендации по запуску :

1. init_script - рекомендованы для использования с Ubuntu 16.04
2. **ВНИМАТЕЛЬНО !!!** Копируем все скрипты в директорию /root/src/init_script/ (изначально в системе директория не создана)
3. init_script - файлы должны быть исполняемыми (# chmod u=rwx,g=,o= /root/src/init_script/* )
4. **ВНИМАТЕЛЬНО !!!** Читаем описание для каждого init_script перед запуском
5. init_script - написаны для определенных версий дистрибутивов
6. init_script - будут использовать абсолютные пути к исполняемым файлам, по-этому при изменении версий может потребоваться изменить пути к файлам
7. Запуск скрипта  
\# cd /root/src/init_script/  
\# ./namescript 
8. Утилиты:  
Если используем временное **ssh** соединение, можно воспользоваться **screen** , чтобы избежать прерывания скрипта  
https://help.ubuntu.ru/wiki/screen  
\# echo **$STY**   - так можно узнать, находитесь ли в сеансе экрана  
2847.sessionname   (OR "") 

## Текущие версии :

1. mysql-server v.5.7
2. boost v.1_59_0 (for mysql)
3. httpd v.2.4.29
4. mod_fcgid-2.3.9 (for apache2)
5. curl-7.59.0 (for php)
6. php-7.3.11
7. composer v1.8.5 (пакетный менеджер, например используется в zend framework)

## Описание init_script по номерам :

0. обновление системы и установка самого необходимого минимального ПО (# reboot)
1. **УВЕЛИЧЕНИЕ RAM - СОЗДАЕМ SWAP FILE** размером в 1ГБ (проверить текущий статус: # swapon --show ) ;  
Для автоматического подключения добавляем запись без кавычек "/swapfile none swap sw 0 0" в файл /etc/fstab (в конце должны быть пустая строка) ; (# mcedit /etc/fstab )  
\# reboot  
\# free -h
2. зависимости для всех сервисов (apache, php, mysql, ...)
3. mysql.server , конфигурационный файл my.cnf ; назначаем пароль для 'root'@'localhost' (mysql) ;;  
(или клонируем директорию data)  
(или клонируем весь mysql/ со старого сервера)  
(важно перетащить юзеров, разрешения, и тд. ; возможно позже с дампом) 
4. apache2 , конфигурационные файлы  
5. **зависимости:**  
tidyp - https://www.php.net/manual/ru/book.tidy.php  
curl - https://www.php.net/manual/ru/book.curl.php  
Функции readline (libedit-dev) - https://www.php.net/manual/ru/book.readline.php  (--with-libedit)  
**php ./configure:**  
CFLAGS='-D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64'   - https://www.php.net/manual/ru/intro.filesystem.php
6. composer (пакетный менеджер, например используется в zend framework) https://getcomposer.org/download/  
заходим на сайт, **меняем sha384** в скрипте 
7. xdebug  
https://xdebug.org/docs/install - **(см.ОБЯЗАТЕЛЬНО)**  
https://www.php.net/manual/ru/install.pecl.phpize.php 
999. установка путей исполняемых файлов в переменную **$PATH** системы (проверка существующих переменных: # printenv )

## После запуска всех init-scripts :

1. setting, sites for apache2  
читаем /usr/local/apache2/conf/README.md  
**fed@mac $ rsync -zaHviP --executability /local/path/htdocs/ Host:/usr/local/apache2/htdocs/** 
2. загрузить сертификаты для сайтов /usr/local/apache2/ssl.crt/NAME_SITE
3. setting for php  
	* при использовании fcgi (apache2) совместно с php-cgi, по-умолчанию используется индивидуальный php.ini для DocumentRoot (apache2);  
	* https://xdebug.org/docs/install#configure-php  
/usr/local/php/lib/php/extensions/no-debug-non-zts-20180731/
4. загрузить dump из старой базы mysql
	- если server.mysql не настроен на удаленное подключение, то делаем так (пример):  
(важно перетащить юзеров, разрешения, и тд.	)  
old-server# mysqldump --opt --set-gtid-purged=off --databases name_db [name_db name_db ..] > dump.sql  
new-server# mysql < dump.sql
5. настройки git  
\# git config --list  (https://git-scm.com/book/ru/v2/Введение-Первоначальная-настройка-Git)  
**используйте локальный # git ...**
