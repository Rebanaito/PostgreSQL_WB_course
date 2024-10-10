## ДЗ №2

Запускаем контейнер и подключаемся к базе с двух терминалов

`
docker container start <container_id>
`

`
docker exec -it test_db psql -U postgres
`

В первой сессии создаем таблицу и добавляем в нее строку

```sql
create table if not exists test_table (
  id serial primary key,
  name text
);

insert into test_table (name) values ('Alex');
```
