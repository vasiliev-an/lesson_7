# **Управление пакетами. Дистрибьюция софта**
### **Создать свой RPM пакет**

Соберем NGINX с поддержкой openssl
Добавляем в vagrantfile строки
``
wget -P /root https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
rpm -i /root/nginx-1.14.1-1.el7_4.ngx.src.rpmmc
wget -P /root https://www.openssl.org/source/latest.tar.gz
tar -C /root -xvf /root/latest.tar.gz
``

Поставим все зависимости чтобы в процессе сборки не было ошибок
``
yum-builddep /root/rpmbuild/SPECS/nginx.spec -y
``

Правим сам spec файл чтобы NGINX собирался с необходимыми нам
опциями.В файле /root/rpmbuild/SPECS/nginx.cpec дуказываем путь до openssl:
в секции %build добавляем строку
``
--with-openssl=/root/openssl-1.1.1d
``
и удаляем строку 
``
--with-debug
``

Сборка RPM пакета:
``
rpmbuild -bb /root/rpmbuild/SPECS/nginx.spec
``

![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/1.png)


Убедимся что пакеты создались:
``
ll /root/rpmbuild/RPMS/x86_64/
``

![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/2.png)


Теперь можно установить наш пакет и убедиться что nginx работает
``
yum localinstall -y /root/rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
``

![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/3.png)

``
systemctl start nginx
systemctl status nginx
``
![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/4.png)

Далее мы будем использовать его для доступа к своему репозиторию


### **Создать свой репозиторий и разместить там ранее собранный RPM**

Теперь приступим к созданию своего репозитория. Директория для статики у NGINX по умолчанию /usr/share/nginx/html. Создадим там каталог repo:
``
mkdir /usr/share/nginx/html/repo
``
Копируем туда наш собранный RPM и, например, RPM для установки репозитория Percona-Server:
``
cp /root/rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/
wget http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm
``
Инициализируем репозиторий командой:
``
createrepo /usr/share/nginx/html/repo/
``
![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/5.png)

Для прозрачности настроим в NGINX доступ к листингу каталога:
В location / в файле /etc/nginx/conf.d/default.conf добавим директиву autoindex on. В результате location будет выглядеть так:

![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/6.png)

Проверяем синтаксис, перезапускаем NGINX и проверяем доступность:

``
nginx -t
nginx -s reload
curl -a http://localhost/repo/
``
![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/7.png)

Все готово для того, чтобы протестировать репозиторий
Добавим его в /etc/yum.repos.d:
``
cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
``

Убедимся что репозиторий подключился и посмотрим что в нем есть:
``
yum repolist enabled | grep otus
 yum list | grep otus
``

![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/8.png)

Установим репозиторий percona-release:
``
yum install percona-release -y
``

![network](https://github.com/vasiliev-an/lesson_7/blob/master/img/9.png)

Все прошло успешно. В случае если вам потребуется обновить репозиторий (а это
делается при каждом добавлении файлов), снова то выполните команду createrepo
/usr/share/nginx/html/repo/



Запускаем и проверяем, какие порты слушаются:

![network](https://github.com/vasiliev-an/lesson_5/blob/master/img/15.png)
