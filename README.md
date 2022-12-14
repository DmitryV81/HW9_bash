Домашняя работа по теме "Bash"

Задание:

Написать скрипт для CRON, который раз в час будет формировать письмо и отправлять на заданную почту.

Необходимая информация в письме:

    Список IP адресов (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта;
    Список запрашиваемых URL (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта;
    Ошибки веб-сервера/приложения c момента последнего запуска;
    Список всех кодов HTTP ответа с указанием их кол-ва с момента последнего запуска скрипта.
    Скрипт должен предотвращать одновременный запуск нескольких копий, до его завершения.
    В письме должен быть прописан обрабатываемый временной диапазон.


Ход работы.

1. В прилагаемом Vagrantfile описывается создание двух виртуальных машин на базе CentOs 7. Первая - server, вторая - client.

На сервере предполагается размещение nginx и почтового клиента mutt

На клиенте предполагается обращение к веб серверу, к примеру, через 

```
curl 192.168.50.10/index.html
curl 192.168.50.10/some_page.html
```

Установка требуемого ПО на сервере:

```
sudo yum install epel-release
sudo yum install nginx mutt
```

* Вместо mutt можно использовать любой другой почтовый клиент

2. Прилагаемый скрипт (файл с именем script) предполагает анализ лог-файлов nginx, расположенных по пути: 

/var/log/nginx/access.log и /var/log/nginx/error.log

3. Далее разбор скрипта:

Скрипт логически поделен на части.

a). Функция parse. Служит для выборки данных из файла access.log за последний час перед запуском скрипта

b). Функция new_hits_with_ip. Служит для вывода списка IP адресов с наибольшим количеством запросов, с указанием количества запросов

c). Функция list_of_urls. Служит для вывода списка наиболее часто запрашиваемых URL с указанием количества запросов

d). Функция all_errors_nginx. Служит для выборки всех записей из файла error.log

e). Функция request_code. Служит для вывода списка всех кодов HTTP-ответа с указанием их количества

Вывод данных из функций производится в создаваемый файл mail.txt, который отправляется на увазанный в скрипте почтовый ящик.

* Для отправки email на сервере должен быть настроен почтовый сервер, либо связка с внешним почтовым сервером.

В скрипте предусмотрена блокировка одновременного повторного его запуска.

Реализация блокировки:

Создается файл LOCKFILE, который располагается по пути /tmp/script_lockfile. Файл не содержит информации и служит для распознавания запущен ли скрипт в 
данный момент времени.

В начале скрипта производится проверка на существование этого файла. Если файл существует, то выводится сообщение о том, что скрипт заблокирован и производится завершение работы скрипта.

Если файл не существует то далее по ходу выполнения скрипта создается этот LOCKFILE

В конце скрипта, после отправки письма с отчетом на email, с помощью trap захватывается сигнал EXIT и происходит удаление файла LOCKFILE

Выполнение скрипта с интервалом раз в час реализуется добавлением записи в crontab:
```
crontab -e
0 * * * * /root/script
```
