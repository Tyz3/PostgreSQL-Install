# PostgreSQL-Install
Версия Postgres в этом руководстве: **REL_15_STABLE**

## Шаг 1. Подготовка и компиляция
1. Авторизуемся в системе и входим под учёткой root.
```
su -
```

2. Установка необходимых пакетов для компиляции Postgres.
```
apt -y update && apt -y upgrade
apt install gnupg2 git gcc make flex bison libreadline-dev zlib1g-dev libssl-dev
```

3. Возвращаемся под учётку обычного пользователя.
```
exit
```

4. Клонируем исходники Postgres с github.
```
git clone https://github.com/postgres/postgres.git && cd postgres/
git checkout REL_15_STABLE
```

5. Выполняем конфигурацию проекта и компилируем.
> ./project - каталог, где будут бинарники postgres.
```
./configure --prefix=$HOME/project
time make -j$(nproc) -s
make install
cd ~/project/bin
```

## Шаг 2. Создание и настройка БД

1. Создаём БД в директории `/home/user/db`.
```
./initdb ~/db
```

2. Заходим в файл конфиг БД командой `nano ~/db/postgresql.conf`, настраиваем нужные параметры.
> Основные комбинации клавиш nano: `Ctrl+X` - закрыть файл, `Ctrl+O` - сохранить файл.
```
listen_addresses = 'IP_ADDRESS', или '*', или 'localhost'
port = 5432
max_connections = 5000
```

3. Заходим в файл доступов БД командой `nano ~/db/pg_hba.conf`, настраиваем параметры доступа пользователей.
Для `db_user` нужно указать ip адрес сервера в виде `0.0.0.0/32` или какую-нибудь подсеть (`10.0.0.0/8`), который выполняет подключение к БД.
> Основные комбинации клавиш nano: `Ctrl+X` - закрыть файл, `Ctrl+O` - сохранить файл.
```
host    all             db_user    192.168.1.0/24          md5
```

## Шаг 3. Запуск БД и создание пользователя

1. Запускаем БД из каталога с бинарниками `cd ~/project/bin`.
```
./pg_ctl -D ~/db -l /var/log/pg_logs.txt start
```

2. Создаём пользователя, которому давали доступ к БД (`db_user`).
> Если в postgresql.conf был изменён порт (по умолчанию 5432), то укажите его в команде `-p ваш_порт`.
```
./createuser --login --no-superuser --no-createdb --no-replication --password -h /tmp db_user
```

3. Создаём БД в системе с указанием владельца.
> Если в postgresql.conf был изменён порт (по умолчанию 5432), то укажите его в команде `-p ваш_порт`.
```
./createdb --owner=db_user -h /tmp project_db
```

4. Изменение пароля для `db_user`.
> Если в postgresql.conf был изменён порт (по умолчанию 5432), то укажите его в команде `-p ваш_порт`.
```
./psql -h /tmp -d project_db -c "ALTER USER db_user WITH PASSWORD 'Password123';"
```
либо
```
./psql -h /tmp -d project_db
ALTER USER db_user WITH PASSWORD 'Password123';
\q
```

**Готово!** Теперь можно выполнить подключение к БД.
URI подключения: `postgres://db_user:Password123@localhost:5432/db_name`.

## Шаг 4. Импорт таблиц в БД
1. Пример импорта таблицы из каталога на сервере.
> Если в postgresql.conf был изменён порт (по умолчанию 5432), то укажите его в команде `-p ваш_порт`.
```
./psql -h /tmp -d db_name < /tmp/table_for_import.sql
```

## Шаг 5. Автозагрузка
1. Переходим под учётку root.
```
su -
```

2. Используем планировщик cron для автозапуска БД при старте сервера `nano /etc/crontab`.
> Основные комбинации клавиш nano: `Ctrl+X` - закрыть файл, `Ctrl+O` - сохранить файл.
```
@reboot         user    /home/user/project/bin/pg_ctl -D /home/user/db start
```

## Шаг 6. Дополнительные действия
1. Очистить историю команда в консоли.
```
rm -rf ~/.bash_history
```
