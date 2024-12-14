# Настройка общего git репозитория в режиме многосайтовости

1. После установки основного сайта и остальных. Переходим к настройке общего репозитория. Для этого переходим в домашний каталог SSH пользователя созданного через ISPConfig при развертывании первого сайта. (Обратите внимание если вы создавали в ISPConfig клиентов и/или еще сайта пути могут отличаться от примера. А так же будет отличаться логин SSH пользователя)
    ```bash
    cd /var/www/clients/client0/web1/home/defaultuser/private
    ```
2. Создаем каталоги для каждого сайта проекта
    ```bash
    mkdir site1.ru
    mkdir site2.ru
    ```
3. Авторизуемся как root `su -` либо `sudo -i`. И в файле fstab добавляем строки задающие монтирование
    ```bush
    /var/www/clients/client0/web1/web  /var/www/clients/client0/web1/home/defaultuser/private/site1.ru/ none bind,nofail 0,0
    /var/www/clients/client0/web2/web /var/www/clients/client0/web1/home/defaultuser/private/site2.ru/ none bind,nofail 0,0
    ```
    И перезагружаем сервер
    ```bush
    shutdown -r now
    ```
4. В каталоге /var/www/clients/client0/web1/home/defaultuser/private git или загружаем с другого источника
5. При необходимости клонируем на рабочий компьютер
    ```bush
    git clone ssh://defaultuser@site1.ru/var/www/clients/client0/web1/private
   ```


