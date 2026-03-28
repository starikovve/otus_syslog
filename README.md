# otus_syslog
# Основы сбора и хранения логов 

Что нужно сделать?

1 Поднимаем две машины — web и log.

2 На web поднимаем nginx.

3 Настраиваем центральный лог-сервер на любой системе по выбору:

journald;

rsyslog;

elk.

4 Настраиваем аудит, который будет отслеживать изменения конфигураций nginx.

5 Все критичные логи с web должны собираться и локально и удаленно.

6 Все логи с nginx должны уходить на удаленный сервер (локально только критичные).

7 Логи аудита должны также уходить на удаленную систему.



 Установка nginx на виртуальной машине web 192.168.209.148

 <img width="1174" height="495" alt="image" src="https://github.com/user-attachments/assets/267ceb7e-d7d4-497b-a711-b5d0586a528e" />

<img width="1478" height="405" alt="image" src="https://github.com/user-attachments/assets/879e126d-a84d-40b5-9848-c3470b354032" />


 Настройка центрального сервера сбора логов 192.168.209.146

nano /etc/rsyslog.conf

```
# Принимаем по UDP и TCP
module(load="imudp")
input(type="imudp" port="514")

module(load="imtcp")
input(type="imtcp" port="514")

# Шаблон хранения: разбиваем по хостам и программам
$template RemoteLogs,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs
& stop
```

В конец файла /etc/rsyslog.conf добавляем правила приёма сообщений от хостов:
Данные параметры будут отправлять в папку /var/log/rsyslog логи, которые будут приходить от других серверов. Например, Access-логи nginx от сервера web, будут идти в файл /var/log/rsyslog/web/nginx_access.log
Далее сохраняем файл и перезапускаем службу rsyslog: systemctl restart rsyslog
Если ошибок не допущено, то у нас будут видны открытые порты TCP,UDP 514:


<img width="1276" height="466" alt="image" src="https://github.com/user-attachments/assets/99f330f0-fdf7-4c35-afae-ecb58b689f8d" />


Далее настроим отправку логов с web-сервера


<img width="881" height="215" alt="image" src="https://github.com/user-attachments/assets/3a4dc4de-88d1-4a1b-ac54-1fed859362f7" />

Для Access-логов указываем удаленный сервер и уровень логов, которые нужно отправлять. Для error_log добавляем удаленный сервер. Если требуется чтобы логи хранились локально и отправлялись на удаленный сервер, требуется указать 2 строки. 	
Tag нужен для того, чтобы логи записывались в разные файлы.
По умолчанию, error-логи отправляют логи, которые имеют severity: error, crit, alert и emerg. Если требуется хранить или пересылать логи с другим severity, то это также можно указать в настройках nginx. 


<img width="745" height="91" alt="image" src="https://github.com/user-attachments/assets/7803309d-4c2c-466a-baee-eb966e5a2a01" />

Примечание
```
директива access_log на 7-й строке находится вне блока http {}, то есть она висит в глобальном контексте, где она не разрешена.
access_log допустима только внутри блоков: http, server, location, if, limit_except.
```
<img width="839" height="233" alt="image" src="https://github.com/user-attachments/assets/6447726a-014b-438b-800c-af17976023e7" />

Далее перезапускаем nginx: systemctl restart nginx


Попробуем несколько раз зайти по адресу http://192.168.209.148

смотрим логи

<img width="1318" height="255" alt="image" src="https://github.com/user-attachments/assets/7da0386e-8815-40c6-bae3-cf8bd55e859a" />


Поскольку наше приложение работает без ошибок, файл nginx_error.log не будет создан. Чтобы сгенерировать ошибку, можно переместить файл веб-страницы, который открывает nginx - 

mv /var/www/html/index.nginx-debian.html /var/www/ 


<img width="1310" height="289" alt="image" src="https://github.com/user-attachments/assets/f3d2c144-c81d-4474-b013-efa0424e4d37" />



