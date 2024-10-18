Создаем таблицу и добавляем в нее две строчки с данными

```sql
create table accounts (
    id integer primary key,
    amount numeric
);

insert into accounts (id, amount) values (1, 21), (2, 42);
```

Открываем второй терминал, открываем транзакцию в обеих сессиях. Пытаемся сделать update обеих строк, только в разном порядке - id 1,2 в первом терминале и id 2,1 во втором.

```sql
--в обоих терминалах
begin;

--в первом терминале
update accounts set amount = 210 where id = 1;

--во втором терминале
update accounts set amount = 420 where id = 2;

--в первом терминале
update accounts set amount = 0 where id = 2;
--команда виснет в ожидании другой транзакции, которая уже взяла блокировку этой строки

--во втором терминале
update accounts set amount = 0 where id = 1;
```

После последней команды происходит deadlock и одна из транзакций (в нашем случае обычно вторая) прерывается, давая другой завершиться. Видим сообщение о deadlock в терминале, транзакция которого была отменена.
Идем проверять логи, которые должны находиться в /var/log/postgresql/postgresql-16-main.log внутри контейнера, но обнаруживаем что в этой директории файлов нет. Проверяем config файл в /var/lib/postgresql/data/postgresql.conf, видим следующую строку

```
log_lock_waits = off
```

Оказывается, по дефолту наша база не логирует ожидания блокировок. Редактируем файл, чтобы выставить значение 'on' (приходится выгружать файл из контейнера через docker cp, так в нем нет ни nano, ни vi/vim).

Повторяем эксперимент, но по-прежнему не находим логов. После небольшого поиска в Интернете выясняем, что конфигурация контейнера может не позволять базе писать в файл.
Вместо чтения файла пробуем найти логи о deadlock через

```bash
docker logs test_db
```

И находим желанные записи:

```
2024-10-17 17:55:57.365 UTC [53] ERROR: deadlock detected
2024-10-17 17:55:57.365 UTC [53] DETAIL: Process 53 waits for ShareLock on transaction 1038; blocked by process 46;
  Process 46 waits for ShareLock on transaction 1039; blocked by process 53;
  Process 53: update accounts set amount = 0 where id = 1;
  Process 46: update accounts set amount = 0 where id = 2;
2024-10-17 17:55:57.365 UTC [53] HINT: See server logs for query details.
2024-10-17 17:55:57.365 UTC [53] CONTEXT: while updating tuple (0,3) in relation "accounts"
2024-10-17 17:55:57.365 UTC [53] STATEMENT: update accounts set amount = 0 where id = 1;
```

Вывод:

Необходимо учитывать механизм работы блокировок и deadlock в частности при многопоточной работе с высоконагруженными базами данных. Излишне долгие транзакции, отсутствие сортировки при вставлении данных и прочие ошибки могут ухудшить производительности базы.
