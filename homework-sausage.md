# Решение студента
## Задание 1. Запуск фронтенда
1. Создаём пользователя `frontend`:
    ```
    $ sudo useradd -m frontend
    ```
    Изменяем права на директорию `/opt/sausage-store/static`(чтобы пользователь `frontend` мог читать файлы):
    ```
    $ sudo сhmod 777 -R /opt/sausage-store/static
    ```
2. Выполняем сборку статики фронтенда сосисочной:
    ```
    $ cd /opt/sausage-store/static/
    $ sudo npm install
    $ sudo npm run build
    ```
    Результатом выполнения команд сборки статики является директория `/opt/sausage-store/static/dist/frontend` с файлами статики.
3. В качестве веб-сервера для раздачи статики выбираем http-server из пакетов npm:
    - Устанавливаем `http-server`:
        ```
        $ sudo npm install -g http-server
        ```
4. Запускаем
    ```
    $ cd /opt/sausage-store/static/dist/frontend
    $ sudo /usr/bin/http-server -p 80 --proxy http://localhost:8080
    ```
## Задание 2. Запуск бэкенда
1. Скачиваем артефакт jar-файл на виртуальную машину.
2. На виртуальной машине создаём директорию для jar-файла:
    ```
    $ mkdir /jarniki
    $ cp sausage-store-0.0.1.jar /jarniki
    ```
3. Создаём директорию для репортов: `$ sudo mkdir -p /var/sausage-store/reports`
4. Создаём пользователя `backend`:`sudo useradd -m backend`
5. Меняем права на директорию /var/sausage-store для возможности записи бэкендом:
    ```
    sudo сhmod 777 -R /var/sausage-store/
    ```
6. Запускаем:
    ```
    $ sudo /usr/lib/jvm/java-17-openjdk-amd64/bin/java -jar /jarniki/sausage-store-0.0.1.jar
    ```
7. Проверяем, что страница с сосисочной доступна по адресу `http://<ip виртуальной машины>/`.





# Комментарии к решению
- Вы используете `chmod 777` для изменения доступа к директориям. Режим `777` может быть небезопасным, т.к. даёт полные права на чтение, запись и исполнение всем пользователям. Для ограничения прав можно ограничить режим до `755` или даже `750`, а также поменять владельца на пользователя сервиса, например:
    ```
    $ sudo chown -R frontend:frontend /opt/sausage-store/static
    $ sudo chmod -R 755 /opt/sausage-store/static
    ```
    или
    ```
    $ sudo chown -R backend:backend /var/sausage-store/
    $ sudo chmod -R 750 /var/sausage-store/
    ```
- Сборку фронтенда по заданию следует производить под пользователем `frontend`, например:
    ```
    $ sudo -u frontend npm install
    $ sudo -u frontend npm run build
    ```
- Можно использовать systemd-юнит для запуска сервиса, это позволяет запускать приложение автоматически при загрузке системы, например можно создать следующий файл /etc/systemd/system/sausage-frontend.service:
    ```
    [Unit]
    Description=Sausage Store Frontend
    After=network.target
    [Service]
    Type=simple
    User=frontend
    ExecStart=/usr/bin/http-server /opt/sausage-store/static/dist/frontend -p 80 --proxy http://localhost:8080
    [Install]
    WantedBy=multi-user.target
    ```
    и для бэкенда файл /etc/systemd/system/sausage-backend.service:
    ```
    [Unit]
    Description=Sausage Store Backend
    After=network.target
    [Service]
    User=backend
    ExecStart=/usr/bin/java -jar /opt/sausage-store/bin/sausage-store-0.0.1.jar
    Environment="REPORT_PATH=/var/sausage-store/reports"
    Environment="LOG_PATH=/var/sausage-store/logs"
    WorkingDirectory=/opt/sausage-store/bin
    Restart=on-failure
    [Install]
    WantedBy=multi-user.target
    ```
    и включить их в автозагрузку:
    ```
    $ systemd daemon-reload
    $ systemctl enable --now sausage-frontend.service
    $ systemctl enable --now sausage-backend.service
    ```
- JAR-файл бэкенда по заданию следует положить в специальную папку, например:
    ```
    $ mkdir -p /opt/sausage-store/bin
    $ cp sausage-store-0.0.1.jar /opt/sausage-store/bin/
    ```
- Также следует убедиться, что директории для отчётов и логов созданы, например:
    ```
    $ mkdir -p /var/sausage-store/reports
    $ mkdir -p /var/sausage-store/logs
    ```
- Можно добавить вывод команд и/или скриншоты


# Тесты на виртуальной машине студента
- Проверить статус сервисов, например `systemctl status sausage-frontend.service` или `systemctl status sausage-backend.service`
- Проверить, что JAR файл запущен, например: `ps aux | grep -i jar`
- Если есть доступ к веб-браузеру, то можно проверить открывается ли страница фронтенда и сделать на ней заказ
- Можно проверить есть ли какие-нибудь отчёты или логи в папках, например `ls /var/sausage-store/reports` или `ls /var/sausage-store/logs`


# Автоматизация процесса
- Можно создать скрипт для автоматизации процесса, в котором прописать все команды
- Можно использовать системы управления конфигураций, такие как Ansible или Salt
- Можно использовать системы сборки и доставки CI/CD, такие как GitLab или Jenkins
