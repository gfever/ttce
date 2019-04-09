#Test task

При решении использовались Laravel 5.7, php 7.2, mysql 8.0

Mysql современная субд, возможножности которой в полной мере покрывают нужды данной задачи
Для кэша использовалась memcache

Проект разворачивался в докере, файлы котороо также приложены, но работа которого с первого раза без вмешательств не гарантируется

---
Комментарии к задаче:

1. На каждую операцию с кошельком создается транзакция, которая содержит в себе пользователя(кошелек), сумму, напраеление +-;

2. Суммы в транзакциях всегде в валюте кошелька

3. Для перевода используется служебная модель трансфера, которая создает две транзакции. Для учета переаодов можно было также в модель добать ссылки на транзакции, в рамках этой задачи это не требуется.

4. Загрузка рэйтов валю осуществляется через апи бесплатного стороннего сервиса https://frankfurter.app/. Этот сервис должен быть доступен для работы приложения.

5. Регистрация максимально упрощена, представляет из себя сохранения данных пользователя в бд.

6. Список валют загружается при миграции, его можно найти в `database/seeds/CurrenciesSeeder.php`. Но работать будут только валюты полученные из 4 пункта. Валидации списка валют нет.

7. requests/api.http есть примеры api запросов

8. `php artisan db:seed --class=TestDataSeeder`  - можно забить базу рэйтами и пользователями

---

Интерфейс отчета доступен по корневому uri /
Выборка производится по имени пользователя, есть фозможность фильтровать по датам и скачивать отчет в csv.

---
Описание api:


1. `POST /api/rate` - обновления рэйтов на сегодняшний день. Обязательно запускать, если рэйтов не будет, приложение работать тоже не будет

2. `POST /api/user,  BODY: [
                'name' => 'required|string|unique:users,name',
                'email' => 'required|email|unique:users,email',
                'country' => 'required|string',
                'password' => 'required|string',
                'city' => 'required|string',
                'currency_code' => 'required'
   ]`- регистрация пользователя
   
3. `PUT /api/wallet/{user_id},  BODY: [
                'password' => 'required|string', 
                'amount' => 'numeric|required'
   ]` - пополнение кошелька, пароль пользователя заданный при регистрации
   
4. `POST /api/transfer, BODY [
                'sender_password' => 'required|string',
                'sender_id' => 'required|integer|exists:users,id',
                'recipient_id' => 'required|integer|different:sender_id|exists:users,id',
                'currency_code' => 'required|exists:currencies,code',
                'amount' => 'required|numeric'
    ]` - перевод между кошельками, currency_code - это валюта суммы в переводе, она должна быть равна валюте одного из кошельков участников. Тут также требуется пароль отправителя
