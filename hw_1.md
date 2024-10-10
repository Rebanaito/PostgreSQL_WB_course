## ДЗ №1

Environment: Ubuntu 22, fresh install

Устанавливаем docker через менеджер Ubuntu Software.

Скачиваем docker-образ 16 версии PostgreSQL:

`docker pull postgres:16`

Скачиваем sql-файл по инструкции отсюда (маленькая версия на 600 МБ):

`wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz`

Запускаем docker-контейнер с базой данных:

`docker run --name test_db -d -p 5432:5432 postgres:16`

Копируем sql-файл в контейнер. Для этого нам потребуется сначала посмотреть ID контейнера:

`
docker ps

docker cp thai.sql <container_id>:/tmp
`

Запускаем утилиту psql в контейнере и подключаемся к базе данных:

`
docker exec -it test_db psql -U postgres
`

`
\l
`

`
\c postgres
`

Исполняем sql-файл, чтобы наполнить базу данными. Проверяем, что создалась новая схема и таблицы с данными:

`
\i /tmp/thai.sql
`

`
\dn
`

`
\dt book.*
`

Делаем select-запрос для подсчета строк с данными в таблице book.tickets:

`
select count(*) from book.tickets;
`

Результат: 5185505
