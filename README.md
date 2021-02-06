# elasticsearch-server
how to install on ubuntu 18.04

+----------------------------------------------Шаг 1 — Установка и настройка Elasticsearch-------------------------------------------------+

+--------------------------------install java 8 и создание переменной----------------------------------+
| sudo apt-get install openjdk-8-jre-headless openjdk-8-jdk-headless                                   |
| update-alternatives --config java                                                                    |
+------------------------------------------------------------------------------------------------------+



Для начала запустите следующую команду для импорта открытого ключа Elasticsearch GPG в APT:
+------------------------------------------------------------------------------------------+
| wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -       |
+------------------------------------------------------------------------------------------+


Затем добавьте список источников Elastic в каталог sources.list.d, где APT будет искать новые источники:
+----------------------------------------------------------------------------------------------------------------------------+
|echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list |
+----------------------------------------------------------------------------------------------------------------------------+ 


sudo apt-get update - Затем обновите списки пакетов, чтобы APT мог прочитать новый источник Elastic:


sudo apt install elasticsearch - Установите Elasticsearch с помощью следующей команды:
 


После завершения установки Elasticsearch редактируем файл конфигурации Elasticsearch с именем elasticsearch.yml.
+-------------------------------------------------------------------------------------+
|sudo nano /etc/elasticsearch/elasticsearch.yml                                       |
+-------------------------------------------------------------------------------------+
Elasticsearch прослушивает весь трафик порта 9200. Чтобы хосты видели порт 9200 elasticsearch
_local_ - любые адреса обратной связи в системе 127.0.0.1.
_site_ -  любые локальные адреса сайта в системе 192.168.0.1.
+------------------------------+
|network.host: _site_,_local_  |
+------------------------------+


чтобы остановить ведение журнала.
+------sudo nano /usr/lib/systemd/system/elasticsearch.service------+
StandardOutput=null
StandardError=null
+-------------------------------------------------------------------+

systemctl daemon-reload

sudo systemctl enable elasticsearch - чтобы активировать Elasticsearch при каждой загрузке сервера:

sudo systemctl start elasticsearch - Затем запустите службу Elasticsearch с помощью systemctl:


curl -X GET "localhost:9200" - протестировать работу службы Elasticsearch, локально 
curl -X GET "пишем ip-адрес сервера:9200" - протестировать работу службы Elasticsearch, по ip адрессу


получим ответ, содержащий базовую информацию о локальном узле:
+------------------------------------------------------------+
| Output                                                     |
| {                                                          |
|  "name" : "ZlJ0k2h",                                       |
|  "cluster_name" : "elasticsearch",                         |
|  "cluster_uuid" : "beJf9oPSTbecP7_i8pRVCw",                |
|  "version" : {                                             |
|    "number" : "6.4.2",                                     |
|    "build_flavor" : "default",                             |
|    "build_type" : "deb",                                   |
|    "build_hash" : "04711c2",                               |
|    "build_date" : "2018-09-26T13:34:09.098244Z",           |
|    "build_snapshot" : false,                               |
|    "lucene_version" : "7.4.0",                             |
|    "minimum_wire_compatibility_version" : "5.6.0",         |
|    "minimum_index_compatibility_version" : "5.0.0"         |
|  },                                                        |
|  "tagline" : "You Know, for Search"                        |
| }                                                          |
+------------------------------------------------------------+

после,проверяем 
netstat -tulpn
должны увидеть сокеты 9200 на локалхосте и на ip адресе


+------------------------------------------------------Шаг 2 — Установка и настройка информационной панели Kiban---------------------------------------+

sudo apt install kibana - установка kibana

+-----------sudo nano /etc/kibana/kibana.yml----------------+
| elasticsearch.url ip-адресс сервера ELK:9200      	      |
| server.host: "0.0.0.0"				                            |
+-----------------------------------------------------------+

sudo systemctl enable kibana - активируем службу 

sudo systemctl start kibana - запускаем службу


http://your_server_ip:5601 - переходим в браузер



+------------------------------установка обратного прокси nginx----------------------------------------+
| sudo apt-get update										                                                               |
| sudo apt-get install nginx 									                                                         |
+------------------------------------------------------------------------------------------------------+


echo "kibanaadmin:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users - команда создаст административного пользователя Kibana и пароль


+-----------------------------------------------------+
| sudo nano /etc/nginx/sites-available/example.com    | - cоздание файла серверного блока Nginx
+-----------------------------------------------------+
server {                                             
    listen 80;                                       
                                                    
    server_name пишем ip- адресс сервера;  
                                                     
    auth_basic "Restricted Access";                  
    auth_basic_user_file /etc/nginx/htpasswd.users;  
                                                     
    location / {                                     
        proxy_pass http://пишем ip- адресс сервера:5601;            
        proxy_http_version 1.1;                      
        proxy_set_header Upgrade $http_upgrade;      
        proxy_set_header Connection 'upgrade';       
        proxy_set_header Host $host;                 
        proxy_cache_bypass $http_upgrade;            
    }                                                
 }                                                   
+-----------------------------------------------------+


sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com - активируем новую конфигурацию, создав символическую ссылку на каталог sites-enabled

sudo nginx -t - проверим конфигурацию на синтаксические ошибки:

sudo systemctl restart nginx - перезапуск службы 

http://your_server_ip/ - переходим в браузер 
