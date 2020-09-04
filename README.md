# init_script


## Рекомендации по запуску :
1. **для использования с Ubuntu 16.04**
2. копируем скрипты в локальную директорию:  
../Host/:/root/src/init_script/
3. Create dir on server:  
root@Host:\~# mkdir -p /root/src/init_script  
root@Host:\~# cd /root/src/init_script
4. Синхронизируем (Sublime Text):  
sftp-config.json  
Sync local->remote
5. **файлы должны быть исполняемыми:**  
\# chmod u=rwx,g=,o= /root/src/init_script/*
6. **ВНИМАТЕЛЬНО !!!**  
	* используются абсолютные пути к исполняемым файлам, по-этому при изменении версий может потребоваться изменить пути к файлам  
	* **При редактировании init_script - НЕ ЗАБЫВАЕМ КОММИТИТЬ И ПУШИТЬ !!!**  
	* **Читаем описание для каждого init_script перед запуском**
7. Утилиты:  
	* **screen** - Если используем временное **ssh** соединение, чтобы избежать прерывания скрипта  
https://help.ubuntu.ru/wiki/screen  
\# echo **$STY**   - так можно узнать, находитесь ли в сеансе экрана  
2847.sessionname   (OR "") 
8. **Запуск скрипта**  
\# cd /root/src/init_script/  
**\# ./namescript** 


## Текущие версии :
1. mysql-server v.5.7
2. boost v.1_59_0 (for mysql)
3. httpd v.2.4.29
4. mod_fcgid-2.3.9 (for apache2)
5. curl-7.59.0 (for php)
6. php-7.3.11
7. composer v1.8.5 (пакетный менеджер, например используется в zend framework)


## Описание init_script по номерам :
0. обновление системы и установка самого необходимого минимального ПО  
**(часто требуется интерактивное подтверждение-выбор)**  
1. **УВЕЛИЧЕНИЕ RAM - СОЗДАЕМ SWAP FILE/PART** размером в 1ГБ  
	**tips:**  
	* **UUID** генерируется при создании файловой системы, файла/раздела подкачки;    
	**пример:**  
	командой \# mkswap /swapfile в данном скрипте  
	no label, **UUID=d8bbf2c8-ee62-4e3c-b230-2bb99d22e5c0**  	
	* \# swapon --show   - проверить текущий статус подкачки  
	* \# free -h   - memory allocation  
	* \# blkid /swapfile   - print block device attributes (TYPE , **UUID**)  

	**require:**	
	* \# mcedit /etc/fstab   - добавляем запись (для автоматического монтирования):  
	// используем **UUID** или **/path** в зависимости раздел/файл  
	**/swapfile none swap sw 0 0**  
	or  
	**UUID="d8bbf2c8-ee62-4e3c-b230-2bb99d22e5c0" none swap sw 0 0**  
	**(в конце файла должна быть пустая строка!!!)**  
	* **\# reboot**  
2. зависимости для всех сервисов (apache, php, mysql, ...)
3. mysql.server , конфигурационный файл my.cnf https://github.com/pizdinad/mysql-etc ; назначаем пароль для 'root'@'localhost' (mysql) ;;  
(или клонируем директорию data)  
(или клонируем весь mysql/ со старого сервера)  
(**важно** перетащить юзеров, разрешения, и тд. ; **возможно позже с дампом**) 
4. apache2 , конфигурационные файлы https://github.com/pizdinad/apache2-conf  
5. **зависимости:**  
tidyp - https://www.php.net/manual/ru/book.tidy.php  
curl - https://www.php.net/manual/ru/book.curl.php  
Функции readline (libedit-dev) - https://www.php.net/manual/ru/book.readline.php  (--with-libedit)  
**php ./configure:**  
CFLAGS='-D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64'   - https://www.php.net/manual/ru/intro.filesystem.php
6. composer (пакетный менеджер, например используется в zend framework) https://getcomposer.org/download/  
7. xdebug  
https://xdebug.org/docs/install - **(см.ОБЯЗАТЕЛЬНО)**  
https://www.php.net/manual/ru/install.pecl.phpize.php 
8. установка путей исполняемых файлов в переменную **$PATH** системы  
\# printenv   - проверка существующих переменных
9. удаляем:  
/root/src/*  
.git/  
.gitignore


## После запуска всех init_script :
0. Синхронизируем - sync remote->local (Sublime Text):  
	* **/root/src/**  
	* **конфигурационные файлы (browse remote...):**  
	/root/usr/local/apache2/conf/  
	/root/usr/local/apache2/htdocs/  
	===============================    
	/root/usr/local/mysql/etc  
	===============================  
	/root/usr/local/php/etc
1. setting, sites for apache2:  
\# TZ='Europe/Moscow' last --time-format iso -i  - show a listing of last logged in users  
**firewall, iptables;**  
читаем /usr/local/apache2/conf/README.md https://github.com/pizdinad/apache2-conf#apache2-conf  
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
5. mini test server:  
	* \# curl eth0.me
	* \# apachectl -t
	* \# curl -I -# ip|domain
	* \# netstat -plnt
6. настройки git  
\# git config --list  (https://git-scm.com/book/ru/v2/Введение-Первоначальная-настройка-Git)  
**используйте локальный # git ...**
