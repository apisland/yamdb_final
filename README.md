![yamdb_workflow](https://github.com/github/docs/actions/workflows/main.yml/badge.svg)

# Проект: Yamdb_final
Учебный проект спринт 16 Яндекс.Практикум


## Запуск проекта:
- Клонировать репозиторий:
```
git@github.com:apisland/yamdb_final.git
```
```
cd yamdb_final/
```
- Создать виртуальное окружение:
```
python -m venv env или python3 -m venv env
```
```
source env/Scripts/activate или . env/bin/activate
```
- установить зависимости из файла **requirements.txt**:
```
python3 -m pip install --upgrade pip
```
```
pip install -r api_yamdb/requirements.txt
```

- Из директории **infra/:**
-  Запустить проект в контейнере
```
docker-compose up -d --build
```
- Выполнить миграции:
```
docker-compose exec web python manage.py migrate
```
- Загрузить CSV файлы в базу данных из __yamdb_final/api_yamdb/static/data/__ используя скрипт (при наличии файлов *.csv):
```
docker-compose exec web python manage.py load_csv_data
```
Внимание! После массового импорта данных в БД происходит
рассинхронизация последовательности первичных ключей (primary key).
Для исправления, необходимо указать следующую последовательность вручную.
Для это необходимо проделать следующее:
Подключится к postgres:
```
docker-compose exec db psql -U <example_username> -d <example_password>
```
**<example_username>** - user указанный в .env POSTGRES_USER;
**<example_password>** - password указанный в .env POSTGRES_PASSWORD;

Далее проверим максимальное значение ID в таблице **reviews_review** и
текущее значение последовательности (выполнить поочередно):

**NB!** Имя таблицы у вас может отличаться, что бы посмотреть список таблиц,
после подключения к postgres введите команду **\dt**
```
	select max(id) from reviews_review;
	select nextval('"reviews_review_id_seq"');
```
Убеждаемся, что первое значение больше второго. Задаем значение
для следующего числа последовательности:
```
select setval('reviews_review_id_seq', (select max(id) from reviews_review)+1);
```
Пример выполнения команд
```
postgres=# select max(id) from reviews_review;
max
75
(1 row)
postgres=# select nextval('"reviews_review_id_seq"');
nextval
1
(1 row)
postgres=# select setval('reviewsreviewidseq', (select max(id) from reviews_review)+1)
postgres-# ;
setval
76
(1 row)
postgres=# exit
```

- Создать суперпользователя:
```
docker-compose exec web python manage.py createsuperuser
```
- Собрать статику:
```
docker-compose exec web python manage.py collectstatic --no-input
```
Если проект разворачивается на Ubuntu и статика не подгрузилась,
необходимо сделать следующее:
При запущенных контейнерах запустить команду
```
docker ps
```
Найти <container_id> для контейнера nginx, затем выполнить последовательно команды:
```
docker exec -it <CONTAINER ID> /bin/sh
ls
cd /var/html/
ls -la /var/html/
```
Если у nginx нет прав на использование папки static, то добавляем эти права:
```
chmod a+rx /var/html/static/
ls -la /var/html/
exit
```
- Копируем файл с базами данных из папки /infra в папку /app (при наличии файла БД *.json):
```
docker cp fixtures.json <id контенера web>:/app
```
- Команда для заполнения БД:
```
docker-compose exec web python manage.py loaddata fixtures.json
```
- Остановить проект в контейнере:
```
docker-compose down -v
```
- Шаблон для наполнения файла .env:
```
DB_ENGINE=django.db.backends.postgresql # указываем, что работаем с postgresql
DB_NAME=postgres # имя базы данных
POSTGRES_USER=postgres # логин для подключения к базе данных
POSTGRES_PASSWORD=postgres # пароль для подключения к БД (установите свой)
DB_HOST=db # название сервиса (контейнера)
DB_PORT=5432 # порт для подключения к БД
```
## Примеры обращения к API сервису:
- Создание пользователя        http://127.0.0.1:8000/api/v1/auth/signup/
```
{ "email": "string", "username": "string" }
```
- Получение Jwt Token      http://127.0.0.1:8000/api/v1/auth/token/
```
{ "username": "string", "confirmation_code": "string" }
```
- Категории                 http://127.0.0.1:8000/api/v1/categories/
- Жанры                     http://127.0.0.1:8000/api/v1/genres/
- Произведения              http://127.0.0.1:8000/api/v1/titles/
- Отзывы                    http://127.0.0.1:8000/api/v1/titles/1/reviews/
- Комментарии               http://127.0.0.1:8000/api/v1/titles/1/reviews/1/comments/
- Пользователи              http://127.0.0.1:8000/api/v1/users/
- Профиль пользователя      http://127.0.0.1:8000/api/v1/users/me/

## Проект выполнен:
[Valentin Klimov](https://github.com/apisland)