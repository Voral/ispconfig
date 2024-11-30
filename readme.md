# Bitrix on Debian 12 with ISPConfig

## Возможности
- быстрая установка на чистую ОС Debian 12
- удобная и интуитивно понятная панель ISPConfig
- возможность настройки трассировки медленных запросов (php-fpm) 
- почта работает "из коробки", Google Mail, Яндекс Почта, mail.ru получают письма без пометки Спам
- создание нескольких сайтов на одном сервере
- разделение пользователей
- автоматизация получения сертификатов для доменов
- поддержка нескольких версий php на одном сервере для разных сайтов
- улучшенная безопасность. Если злоумышленник взломает один из сайтов, он не сможет получить доступ к сайтам других пользователей.

## Сообщество

1. Если есть какие-то вопросы или предолжения - вступайте в группу телеграм https://t.me/BitrixSe.
2. Если Вы нашли ошибку, у Вас есть замечания или предложения по функционалу, пишите тикет сюда: https://github.com/Wladimir-N/ispconfig/issues.

## Установка Ispconfig
```bash
cd /root/
wget https://raw.githubusercontent.com/Wladimir-N/ispconfig/debian12/ispconfig-debian12.sh
chmod +x ispconfig-debian12.sh
./ispconfig-debian12.sh
   ```

- запускать скрипт под рутом 
- ставить на чистую систему!
- именно классическая авторизация под root
  либо su - 
  либо sudo -i
- директория /root/ispconfig системная и используется при установке, соответственно до запуска скрипта установки не должна существовать
  т.е. такой вариант установки если находимся в /root/ некорректен - git clone https://github.com/Wladimir-N/ispconfig.git && cd ./ispconfig && ...
- логи установки смотрим в /root/ISPConfig.log
- креды для доступа в ISPConfig смотреть в /root/.ispconfig.ini - первая строчка admin и пароль соответственно
- root пароль MySQL - вторая строчка в /root/.ispconfig.ini

## Настройка
- для работы в ISPConfig используется порт 81
- для посещаемых сайтов можно увеличить количество php воркеров (опция PHP-FPM pm.max_children)
### Настройка сайта на движке Битрикс
- заходим в панель, создаём сайт и в закладке Options копируем из файла https://github.com/Wladimir-N/ispconfig/blob/debian12/Bitrix 
  настройки PHP (опция Custom php.ini settings) и nginx (опция nginx Directives)
### Настройка MySQL
- если железо одинаковое и сайт преезжает с BitrixVM для начала можно скопировать файл bvat.cnf с BitrixVM в каталог /etc/mysql/mariadb.conf.d/, но переименовать, например, в 90-my.cnf 
- после суток работы установить запустить Mysql тюнер и поправить настройки согласно рекомендациям
```bash
  apt install mysqltuner
  mysqltuner
```
## Установка и настройка необходимых модулей
- по необходимости доставим в систему утилиты и модули
  (к примеру)
```bash
  apt install mc 
  apt install php8.2-memcache
   ```
  правки можно вносить в конец файла /etc/mysql/mariadb.conf.d/51-server.cnf
- установка sphinxsearch
- ```bash
  sudo apt-get install sphinxsearch
   ```  
  берём конфиг sphinx из админки битрикса, удаляем строчку infix_fields=
  этот конфиг добавляем к дефолтному /etc/sphinxsearch/
  правим /etc/default/sphinxsearch на предмет set START=yes
  ```bash
  sudo service sphinxsearch start
   ```
- установка composer
```bash
apt install --no-install-recommends composer
   ```  

## Настройка логирования трассировок медленных запросов
Для настройки логирования трассировок медленных запросов читаем php-slow-log.txt

## Смена версии PHP
Для установки доп.версий PHP используем скрипт
https://raw.githubusercontent.com/Wladimir-N/ispconfig/debian12/autoinstall.sh
по результату добавления нужной PHP версии в консоль выведет параметры которые прописываем в ISPConfig - Setting - Additional PHP Versions
далее в настройках сайта выбираем эту версию

## Почта
Используется почтовый сервер postfix
Почта работает "из коробки", Google Mail, Яндекс Почта, mail.ru получают письма без пометки Спам.
Очередь сообщений можно глянуть в ISPConfig - Monitor - Mail Queue
Там же рядом логи ISPConfig - Monitor - Mail log

## Настройка сайтов для работы с одним ядром Bitrix
работаем root пользователем:
у пользователя под которым работает второй сайт в /etc/passwd меняем id пользователя и группы на те-же, что у первого сайта
например, было
web1:x:5004:5005:....
web2:x:5005:5005:....
меняем
web2:x:5005:5005:.... -> web2:x:5004:5005:....

переходим в домашнюю папку пользователя web2 - меняем владельца
```bash
cd /var/www/clients/client0/web2/web/bitrix
find . -user 5005 -group 5005 -exec chown 5004:5005 {} +

   ```
логинимся вторым пользователем 
```bash
su - web2 -s /bin/bash
   ```
создаем симлинки на папки /upload/ и /bitrix/
```bash
cd /var/www/clients/client0/web2/web/bitrix
ln -s /var/www/clients/client0/web1/web/upload upload
ln -s /var/www/clients/client0/web1/web/bitrix bitrix
   ```

Для корректной работы обоих сайтов хранение сессий рекомендуется сделать в БД
https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=43&LESSON_ID=14026

## Возможные проблемы и их решения
### Применяю настройки в ISPConfig - по факту ничего не изменяется
При этом в правом верхнем углу ISPConfig наблюдаются красный кружок в цифрой - это не применённые изменения
Проблема вероятно из-за php 8.3 по умолчанию в ОС, решение:
```bash
update-alternatives --config php
```
выбираем php 8.2 ждем более 1 минуты (изменения ISPConfig применяются раз в минуту) - должно работать

### Не корректно запускается cron_events.php
Скорее всего в php.ini не включена опция short_open_tag. Можно задать глобально в конфигах /etc/php/ваша версия}/cli/php.ini

## Рекомендации
### При создании сайта
Если у Вас виртуальный сервер не нужно создавать/использовать клиентов. Создайте сайт под админом.
### Для корректной работы почты 
Должны быть прописаны прямая A и обратные записи PRT для домена сервера (не путать с доменом сайта). В том числе и для IPv6!
Сервис для проверки работы почты https://www.mail-tester.com/ - отправлять письмо нужно непосредственно с сайта, например в PHP консоли mail()