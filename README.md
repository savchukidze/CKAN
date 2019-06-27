## Як розгорнути CKAN? Український переклад

У даному матеріалі описано, як встановити CKAN [***"from package"***](https://docs.ckan.org/en/2.8/maintaining/installing/install-from-package.html).
Це найшвидший та найпростіший спосіб встановити CKAN, проте він вимагає 64-бітної версії **Ubuntu 16.04** або ж 64-бітної версії **Ubuntu 14.04**. Якщо ж ви не використовуєте 64-бітну версію Ubuntu 16.04 або 64-бітну версію Ubuntu 14.04, або якщо ви встановлюєте CKAN для **розробки**, вам слід користуватися інструкцією встановлення CKAN [***"from source"***](https://docs.ckan.org/en/2.8/maintaining/installing/install-from-source.html). Встановлення CKAN ***"from source"*** працює з іншими версіями Ubuntu, а також з іншими операційними системами (зокрема, RedHat, Fedora, CentOS, OS X).

Також якщо Ви плануєте використовувати операційну систему, засновану не на **Debian**, найкращі способи інсталювання необхідних пакунків у Вашій операційній системі можна переглянути у [цьому розділі](https://github.com/ckan/ckan/wiki/How-to-Install-CKAN).

Натомість у випадку встановлення CKAN ***"from package"*** з допомогою даною інструкції, наприкінці інсталяції ви отримаєте дві розгорнуті веб-програми, власне, **CKAN** та **DataPusher**, окрему службу для автоматичного імпортування даних до розширення [DataStore](https://docs.ckan.org/en/2.8/maintaining/datastore.html) CKAN .

При розгортанні CKAN, є наступні вимоги до портів хоста:

|Сервіс|Порт|Використовується для|
| :---: | :---:|:---:|
|NGINX	|80	|Proxy|
|Apache2	|8080|	Web Server
|Solr/Jetty|	8983	|Search
|PostgreSQL|	5432	|Database
|Redis|	6379|	Search

### 1. Інсталяція CKAN "from package"

У вашій системі Ubuntu відкрийте **термінал** і по черзі запускайте наступні команди для встановлення CKAN:

#### I. Оновіть індекси пакетів Убунту:

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
> Якщо Ви побачили наступну помилку, це означає, що з якоїсь причини модуль Apache WSGI не був включений:

```
Syntax error on line 1 of /etc/apache2/sites-enabled/ckan_default:
Invalid command 'WSGISocketPrefix', perhaps misspelled or defined by a module not included in the server configuration
Action 'configtest' failed.
The Apache error log may have more information.
   ...fail!
```
> Ви можете увімкнути його, виконавши по черзі наступні команди в терміналі:

```p
sudo a2enmod wsgi
sudo service apache2 restart
```
### 2. Встановлення та налаштування PostgreSQL

> Ви можете розгорнути PostgreSQL та CKAN на різних серверах. У такому разі просто змініть параметр [sqlalchemy.url](https://docs.ckan.org/en/2.8/maintaining/configuration.html#sqlalchemy-url) `(/etc/ckan/default/production.ini)`, щоб він посилався на ваш сервер PostgreSQL.

Встановіть PostgreSQL, виконавши наступну команду в терміналі:

```
sudo apt-get install -y postgresql
```
Перевірте, чи правильно встановлено PostgreSQL, вказавши існуючі бази даних:

```r
sudo -u postgres psql -l
```

Також обов'язково переконайтесь, що кодування Ваших баз даних – `UTF8`. В іншому випадку, Ви точно зіштовхнетеся із рядом проблем.

Далі потрібно створити користувача бази даних, якщо він ще не існує. Створіть нового користувача бази даних PostgreSQL, який називається **ckan_default**, і введіть пароль для користувача під час запиту. Цей пароль Вам знадобиться пізніше:

```r
sudo -u postgres createuser -S -D -R -P ckan_default
```

Створіть нову базу даних PostgreSQL, яка називається **ckan_default**, що належить користувачеві бази даних, яку Ви тільки що створили:
```r
sudo -u postgres createdb -O ckan_default ckan_default -E utf-8
```

```diff
!Примітка
```
> Якщо Ви вирішили запускати PostgreSQL на окремому сервері, то вам потрібно буде проредагувати файли ***postgresql.conf*** та ***pg_hba.conf***. Для PostgreSQL 9.1 на Ubuntu ці файли знаходяться в *etc/postgresql/9.1/main*.

Розкоментуйте параметр `listen_addresses` та вкажіть розділений комами список IP-адрес мережевих інтерфейсів, які повинні бути доступні для PostgreSQL або  вкажіть `*` , щоб усі можливі були доступними. Наприклад,

```r
listen_addresses = 'localhost,192.168.1.21'
```
Як у прикладі нижче, додайте рядок внизу файлу pg_hba.conf, щоб дозволити машині, на якій працює Apache, підключатися до PostgreSQL. Будь ласка, змініть IP-адресу відповідно до ваших налаштувань мережі.

```r
host    all             all             192.168.1.22/32                 md5
```
Насамкінець відредагуйте параметр [sqlalchemy.url](https://docs.ckan.org/en/2.8/maintaining/configuration.html#sqlalchemy-url) у файлі [конфігурації CKAN](https://docs.ckan.org/en/2.8/maintaining/configuration.html#config-file)(/etc/ckan/default/production.ini) та встановіть коректний пароль та назви бази даних та користувача бази даних (дані з попереднього етапу)

### 3. Встановлення та налаштування Solr 
> Ви можете встановити Solr та CKAN на різних серверах. Просто змініть налаштування [solr_url](https://docs.ckan.org/en/2.8/maintaining/configuration.html#solr-url) (/etc/ckan/default/production.ini, щоб посилання на сервер Solr було коректним.

Встановіть Solr, виконавши наступну команду в терміналі:

```r
sudo apt-get install -y solr-jetty
```

У кінці завершення інсталяції Ви побачите наступне:

```p
* Not starting jetty - edit /etc/default/jetty (or /etc/default/jetty8) and change NO_START to be 0 (or comment it out).
```

CKAN використовує Solr в якості власної пошукової системи. CKAN використовує налаштовувані файли схеми Solr, які враховують усю необхідну специфіку пошуку. Тепер, коли ми встановили CKAN, нам також потрібно встановити та налаштувати Solr.

>> У даній інструкції пояснюється, як розгортати Solr за допомогою веб-сервера Jetty, але CKAN не вимагає Jetty - ви можете розгорнути Solr на іншому веб-сервері, наприклад Tomcat, якщо це зручно у вашій операційній системі.

#### I. Відредагуйте файл конфігурації Jetty (/etc/default/jetty8 або /etc/default/jetty) та змініть наступні змінні:

```p
NO_START=0            # (line 4)
JETTY_HOST=127.0.0.1  # (line 16)
JETTY_PORT=8983       # (line 19)
```
Дані налаштування `JETTY_HOST` є коректними лише тоді, коли підключення відбувається з одного сервера. Якщо CKAN не встановлено на тому ж сервері, що й Jetty/Solr, вам доведеться змінити їх на відповідний хост або на 0.0.0.0 (і, можливо, відповідно налаштувати брандмауер).

Запустіть або перезапустіть сервер Jetty.

Для **Ubuntu 16.04**:

```p
sudo service jetty8 restart
```

або для **Ubuntu 14.04**:
```p
sudo service jetty restart
```

>> Будь ласка, ігноруйте будь-яке попередження про те, що Jetty ще не запущено - деякі дистрибутиви Ubuntu не запускають Jetty під час встановлення, але це не важливо.

Тепер Ви повинні бачити вітальну сторінку Solr, якщо ви відкриєте http://localhost:8983/solr/ у вашому веб-браузері (замініть localhost на адресу вашого сервера, якщо це потрібно).

Якщо Ви отримали наступне повідомлення
`Could not start Jetty servlet engine because no Java Development Kit (JDK) was found.` – Вам необхідно відредагувати файл `JAVA_HOME` (/etc/default/jetty), щоб вказати місце інсталяції JDK на вашому сервері.
Наприклад, 
```p
JAVA_HOME=/usr/lib/jvm/java-6-openjdk-amd64/
```
або
```p
JAVA_HOME=/usr/lib/jvm/java-6-openjdk-i386/
```

#### II. Замініть файл **`schema.xml`** символьним посиланням на файл схеми CKAN, що міститься в джерелах.
```p
sudo mv /etc/solr/conf/schema.xml /etc/solr/conf/schema.xml.bak
sudo ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml /etc/solr/conf/schema.xml
```
Тепер перезапустіть Solr:

Для **Ubuntu 16.04**:

```p
sudo service jetty8 restart
```

або для **Ubuntu 14.04**:
```p
sudo service jetty restart
```
Перевірте, чи працює Solr, відкривши http://localhost:8983/solr/ (замініть localhost на адресу вашого сервера, якщо це потрібно).

#### III. Насамкінець змініть налаштування [solr_url](https://docs.ckan.org/en/2.8/maintaining/configuration.html#solr-url) (/etc/ckan/default/production.ini, щоб посилання на сервер Solr було коректним, наприклад:
```p
solr_url=http://127.0.0.1:8983/solr
```
### 4. Оновлення конфігурації та ініціалізація бази даних

#### І. Відредагуйте файл у файлі [конфігурації CKAN](https://docs.ckan.org/en/2.8/maintaining/configuration.html#config-file)(/etc/ckan/default/production.ini), задавши наступні параметри:

**site_id**

Кожен сайт CKAN повинен мати унікальний `site_id`, наприклад:
```r
ckan.site_id = default
```
**site_url**

Укажіть URL-адресу сайту. Наприклад:
```r
ckan.site_url = http://demo.ckan.org
```
#### ІІ. Ініціалізуйте вашу базу даних CKAN, виконавши наступну команду в терміналі:
```p
sudo ckan db init
```
> За бажанням, налаштуйте **DataStore** і **DataPusher**, дотримуючись інструкцій за посиланням [розширення DataStore](https://docs.ckan.org/en/2.8/maintaining/datastore.html).

> Також, за бажання, ви можете увімкнути завантаження файлів, дотримуючись інструкцій за посиланням [FileStore та завантаження файлів](https://docs.ckan.org/en/2.8/maintaining/filestore.html).

### 5. Перезапустіть Apache і Nginx
```r
sudo service apache2 restart
sudo service nginx restart
```
### 6. Ви зробили це!

Відкрийте http://localhost (або адресу Вашого сервера) у браузері. Ви повинні побачити головну сторінку CKAN, яка виглядатиме приблизно ось так:

![](https://docs.ckan.org/en/2.8/_images/9.png)

Тепер ви можете перейти до інших розділів, щоб почати використовувати та налаштувати ваш сайт на CKAN.
Наприклад, Ви можете створити [системного адмінвістатора](https://docs.ckan.org/en/2.8/maintaining/getting-started.html) або змінити [зовнішній вигляд](https://docs.ckan.org/en/2.8/sysadmin-guide.html) Вашого сайту.

Інстукція на англійській занходиться [тут](https://docs.ckan.org/en/2.8/maintaining/installing/install-from-package.html).
Подальші налаштування вашого сайту на CKAN [тут](https://docs.ckan.org/en/2.8/maintaining/getting-started.html).
