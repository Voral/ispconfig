# Развертывание сайта на Битрикс Управление Сайтом

> Все действия в консоли необходимо выполнять от root. Воспользовавшись классической авторизацией либо `su -`
> либо `sudo -i`

1. В ISPConfig создаем сайт. На вкладке Options:
   * параметру "PHP open_basedir" задаем значение `none`
   * в параметр "Custom php.ini settings" копируем значение из секции php файла [Bitrix](/Bitrix)
   * в параметр "nginx Directives" копируем значение из секции nginx файла [Bitrix](/Bitrix)
2. В ISPConfig создаем пользователя БД
3. В ISPConfig создаем базу данных
4. В ISPConfig создаем пользователя SSH для этого сайта.
5. Авторизуемся под соответствующим пользователем и переходим в каталог сайта
    ```bash
    cd ~/web/  
    ```
6. Скачивание скрипта установки
    * Если необходима установка нового сайта или копии сайта находящейся в облаке или доступной по ссылке скачиваем
      скрипт установки Bitrix
    ```bash
    wget https://www.1c-bitrix.ru/download/scripts/bitrixsetup.php  
    ```
    * Если необходима установка сайта из резервной копии которую мы загружаем самостоятельно на сервер скачиваем скрипт
      восстановления резервных копий Bitrix
    ```bash
    wget https://www.1c-bitrix.ru/download/files/scripts/restore.php  
    ```
7. Устанавливаем сайт

