Методическая информация
=======================

Данный документ содержит описание и методическую информацию по используемуу
контуру

## PostgreSQL "Steroids" for Vanessa Users

> у каждого разработчика 1С - должен стоять локально PostgreSQL, чтобы не расслабляться

Шаги для первоначальной настройки:

* скачать Vagrant https://www.vagrantup.com
* скачать VirtualBox https://www.virtualbox.org/wiki/Downloads
* скачать git (msgit) https://git-for-windows.github.io/

установить вышеуказанные программы и запустить командную строку в каталоге для эксперментов, в которой выполнить:

```
git clone https://github.com/VanessaDockers/pgclass20.git
cd pgsteroids
vagrant up
vagrant ssh
cd /vagrant
./build.sh
./run.sh
```

Включайте 1С сервер и создавайте свои базы 1С.

* сервер 1С лучше использовать в виде запуска из командной строки
* запускать через командную строку `ragent <параметры сервера>`

адрес подключения сервера PostgreSQL

```
host=ВАШ-IP port=5432 user=postgres passw=strange
```

не забудьте заглянуть на порты `8888` и `8081` c теми же пользователем и паролем

Текущий релиз `0.6` содержит

* 1 образ PostgreSQL - 9.6.2
* контейнеры:
 * pgHero - базовый онлайн контроль
 * POWA - расширенный онлайн контроль
 * pgBadger - аналитический контроль
 * pgStudio - онлайн выполнение запросов


и дополнительно включенные необходимые для 1С расширения.

[![Открытый чат проекта https://gitter.im/VanessaDockers/pgsteroids](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/VanessaDockers/pgsteroids?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

### Vagrant

Эти команды сделают виртуальную машину

```
vagrant up
vagrant ssh
cd /vagrant
```

* объем выделенной оперативной памяти 1Gb (можно менять в `Vagrantfile`)
* 4 диска - расширяемых до 250gb

значения меняйте через Virtual Box GUI в нужную Вам сторону.
  

## Ну и конечно run

ваши переменные стоит подсмотреть в файле run.sh и создать файл `.env` со своими значениями.

```
cd /vagrant
run.sh
```

* базируется на улучшенном дистриубтиве PostgreSQLPro с уточнениями

* **порт 5432** - PostgreSQLPro1C http://1c.postgrespro.ru/
* **порт 8888** - POWA http://dalibo.github.io/powa/
* **порт 9999** - PGHero (требует уже созданной 1С базы) https://github.com/ankane/pghero

все службы находятся на **внешнем** IP вашего проверочного комьютера

* база создается средствами 1С

> сервер 1С лучше на Windows (не спрашивайте почему) и версии старше 8.3.6.1760 (смотрите свойсва libpq.dll в составе 1С платформы)

* после создания базы обратите внимание на шаблонные postgresql.conf

```
vagrant ssh
cd vagrant
pconf/postgresql_conf.ERP # для ERP 2.1
pconf/postgresql_conf.UT # для Управления торговли
pconf/postgresql_conf.PostgreSQLPro # Удобный для быстрого старта

```

но для первых экспериментов их лучше не смотреть, а сделать запуск тюнинга конфигурационного файла

### Скрипты

Несколько примеров скриптов SQL `./pscripts` - позапускайте их

* для просмотра сжатия колонок вашей базы
* для поиска отсутствующих индексов

#### Управляющие скрипты администратора

* **пример получения bloat**

```
vagrant ssh
cd /vagrant
./vendors/bloat/pg_bloat_check.py --create_view -c "host=localhost dbname=<ИмяБазы> user=postgres password=strange"
./vendors/bloat/pg_bloat_check.py -c "host=localhost dbname=<ИмяБазы> user=postgres password=strange"
```

* **пример принудительного сжатия bloat**

```
vagrant ssh
cd /vagrant
perl ./vendors/compactable/bin/pgcompacttable -h localhost -p 5432 -u postgres -w strange -d <БазуВставьте> -n public
```

* **пример построения статистики использования для анализа**

```
vagrant ssh
cd /vagrant
./tools/checkpoint-reports.sh
```

отчет возникнет в каталоге `./temp/wwwreports/out.html

* **запуск PGHero на вашей экспериментальной БД**

```
vagrant ssh
cd /vagrant
./pghero-database.sh <ИмяБазыДанных>
```

для администраторов - существуют 2 скрипта

* вход в ssh PG хоста - `./tools/enter-to-pg.sh`
* вход в `psql` PG хоста - `./tools/enter-to-psql.sh`

### Для экспериментов используйте

* Видео на Инфостарте http://infostart.ru/webinars/463095/
* Структуру просмотра имен таблиц 1C http://infostart.ru/public/147147/

#### Список конфигураций и публикаций для тестирования 1С

* Fragister конфигурацию (убийцу фрагов) http://infostart.ru/public/173394/
* "Тест Гилева" http://www.gilev.ru/tpc1cgilv/
* статью "Нагрузочное тестирование 1С:Документооборот" http://infostart.ru/public/440094/

и другие тестовые конфигурации, [в том числе со сценариями](https://github.com/silverbulleters/vanessa-behavior)

### Список расширений PostgreSQL

* pg_prewarm
* pg_bufferscache
* pg_stat_statements
* btree_gist
* hypopg

## Известные проблемы

* Windows 10 x64 в режиме чистой установки не содержит некоторых DLL для работы Vagrant - если вы получаете ошибку `could not be found or
could not be accessed in the remote catalog` тогда установите дополнительный пакет [как это описано тут](https://github.com/mitchellh/vagrant/issues/6764#issuecomment-210226230)

### Цели, авторы, благодарности

мы считаем что разработчик 1С должен проверять свои решения под работой не только MSSQL, но и PostgreSQL. Для быстроты запуска такого контура и создан этот репозиторий.

`(c) allustin, pumbaEO and some secret people`

отдельная благодарность

* http://infostart.ru/
* https://github.com/postgrespro/
* https://github.com/PostgreSQL-Consulting
* https://github.com/2ndQuadrant
* http://dalibo.github.io/

> LICENSE - Mozilla Pubic License (see it at LICENSE file)
