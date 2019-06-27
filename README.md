# CKAN
Installing CKAN from source. UA translation

## Як розгорнути CKAN? Український переклад

У даному матеріалі описано, як встановити CKAN ***"from package"***. Це найшвидший та найпростіший спосіб встановити CKAN, проте він вимагає 64-бітної версії **Ubuntu 16.04** або ж 64-бітної версії **Ubuntu 14.04**. Якщо ж ви не використовуєте 64-бітну версію Ubuntu 16.04 або 64-бітну версію Ubuntu 14.04, або якщо ви встановлюєте CKAN для **розробки**, вам слід користуватися інструкцією встановлення CKAN [***"from source"***](https://docs.ckan.org/en/2.8/maintaining/installing/install-from-source.html). Встановлення CKAN ***"from source"*** працює з іншими версіями Ubuntu, а також з іншими операційними системами (зокрема, RedHat, Fedora, CentOS, OS X).

Також якщо Ви плануєте використовувати операційну систему, засновану не на **Debian**, найкращі способи інсталювання необхідних пакунків у Вашій операційній системі можна переглянути у [цьому розділі](https://github.com/ckan/ckan/wiki/How-to-Install-CKAN).

Натомість у випадку встановлення CKAN ***"from package"*** з допомогою даною інструкції, наприкінці інсталяції ви отримаєте дві запущені веб-програми, **CKAN** і **DataPusher**, окрему службу для автоматичного імпортування даних до розширення [DataStore](https://docs.ckan.org/en/2.8/maintaining/datastore.html) CKAN .

При розгортанні CKAN, є наступні вимоги до портів хоста:

|Сервіс|Порт|Використовується для|
| :---:  | :---:|:---:|
|NGINX	|80	|Proxy|
|Apache2	|8080|	Web Server
|Solr/Jetty|	8983	|Search
|PostgreSQL|	5432	|Database
|Redis|	6379|	Search

### 1. Інсталяція CKAN "from package"

У вашій системі Ubuntu відкрийте **термінал** і по черзі запускайте наступні команди для встановлення CKAN:

#### I. Оновіть індекс пакетів Убунту:

```r
sudo apt-get update
```

#### II. Встановіть пакунки Ubuntu, які вимагає CKAN (та "git", щоб мати змогу встановлювати розширення для CKAN):

```r
sudo apt-get install -y nginx apache2 libapache2-mod-wsgi libpq5 redis-server git-core
```
#### III. Завантажте пакет CKAN:

Для **Ubuntu 16.04**:

```p
wget http://packaging.ckan.org/python-ckan_2.8-xenial_amd64.deb
```

або для **Ubuntu 14.04**:
```p
wget http://packaging.ckan.org/python-ckan_2.8-trusty_amd64.deb
```

#### IV. Встановіть пакет CKAN:
Для **Ubuntu 16.04**:

```p
sudo dpkg -i python-ckan_2.8-xenial_amd64.deb
```

або для **Ubuntu 14.04**:
```p
sudo dpkg -i python-ckan_2.8-trusty_amd64.deb
```

```diff
!Примітка
```
> Якщо ви отримали таку помилку, це означає, що з якоїсь причини модуль Apache WSGI не був включений:

```
Syntax error on line 1 of /etc/apache2/sites-enabled/ckan_default:
Invalid command 'WSGISocketPrefix', perhaps misspelled or defined by a module not included in the server configuration
Action 'configtest' failed.
The Apache error log may have more information.
   ...fail!
```
Ви можете увімкнути його, виконавши по черзі наступні команди в терміналі:

```p
sudo a2enmod wsgi
sudo service apache2 restart
```

