# Телеграм бот с API
1. Создаете пакет **orm** для работы с бд
   
``` 
project/
│
├── orm/
│   │
│   ├── __init__.py
│   │
│   ├── db.py
│   │
│   └── custom_orm.py
│
└── other_files_or_directories/
```

```python
# orm/__init__.py

from .custom_orm import SQLite3ORM

# Простыми словами - экспорт данных
# К примеру, если мы находимся в не пакета orm, на файле main.py и пытаемся импортировать class SQLite3ORM из файла custom_orm.py
# То нам не нужно напрямую импортировать как from orm.custom_orm import SQLite3ORM

```

```python
# orm/db.py

from custom_orm import SQLite3ORM

db_path = "example.db"

with SQLite3ORM(db_path) as db:
    # Create new table
    db.create_table(
        table_name="users",
        columns={
            # UNIQUE - помогает ссылаться на данную таблицу в качестве первичного ключа
            "telegram_id ": "INTEGER UNIQUE",
            "first_name ": "TEXT",
            "last_name ": "TEXT",
            "username  ": "TEXT",
        }
    )

    db.create_table(
        table_name="history",
        columns={
            # Связь таблицы history с users по telegram_id
            "user_id ": "INTEGER REFERENCES users(telegram_id)",
            "name ": "TEXT",
            "price ": "INTEGER",
            "created_at": 'TEXT'
        }
    )
```


```python
# orm/custom_orm.py

import sqlite3

class SQLite3ORM:
    def __init__(self, db_path):
        # Путь к бд
        self.db_path = db_path
        self.connection = None

    # Метод который используется в контекстном менеджере(with)
    def __enter__(self):
        # При создании объекта, автоматом происходит подключение
        self.connection = sqlite3.connect(self.db_path)
        # Это нужно для того, чтобы мы могли вытаскивать записи в виде словаря
        self.connection.row_factory = sqlite3.Row
        self.cursor = self.connection.cursor()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        # Автоматическое закрытие подключения с бд, не нужно использовать метод close()
        if self.cursor:
            self.cursor.close()
        if self.connection:
            self.connection.close()
        if exc_type:
            raise exc_val

    def create_table(self, table_name, columns):
        try:
            sql = f"CREATE TABLE IF NOT EXISTS {table_name} ("
            sql += "id INTEGER PRIMARY KEY AUTOINCREMENT, "

            # Извлекаем из словаря название столбца и его значение
            for col_name, col_type in columns.items():
                sql += f"{col_name} {col_type}, "

            # Срез чтобы убрать запятую
            sql = sql[:-2] + ")"

            self.cursor.execute(sql)
            self.connection.commit()
        except sqlite3.Error as e:
            print(f"Error creating table: {str(e)}")

    def insert(self, table_name, data):
        try:
            # Извлекаем название столбцов из словаря
            keys = ", ".join(data.keys())
            placeholders = ", ".join("?" for _ in data.values())
            # Извлекаем запись столбца из словаря
            values = tuple(data.values())
            sql = f"INSERT INTO {table_name} ({keys}) VALUES ({placeholders})"
            self.cursor.execute(sql, values)
            self.connection.commit()
        except Exception as e:
            print(f"Error while inserting data to table {table_name}: {e}")
            raise e

    def select(self, table_name, columns=None, where=None, order_by=None):
        # Список колонок, которые будут запрошены в SQL-запросе
        if columns is None:
            columns = "*"
        columns = ", ".join(columns)

        sql = f"SELECT {columns} FROM {table_name}"

        # Условие WHERE
        if where is not None:
            # Извлечение столбцов из словаря, разделяя их по AND
            where_clause = " AND ".join([f"{k} = ?" for k in where.keys()])
            # Извлечение записи из словаря
            where_values = tuple(where.values())
            sql += f" WHERE {where_clause}"
        else:
            where_values = ()

        # Сортировка
        if order_by is not None:
            sql += f" ORDER BY {order_by}"

        try:
            self.cursor.execute(sql, where_values)
            rows = self.cursor.fetchall()
            return [dict(row) for row in rows]
        except sqlite3.Error as e:
            print(f"Error while selecting from table {table_name}: {e}")

    def update(self, table_name, set_values, where):
        set_clause = ", ".join(f"{key} = ?" for key in set_values.keys())
        # Извлечение столбцов из словаря, разделяя их по AND
        where_clause = " AND ".join(f"{key} = ?" for key in where.keys())
        # Извлечение записи из словаря
        values = tuple(set_values.values()) + tuple(where.values())
        sql = f"UPDATE {table_name} SET {set_clause} WHERE {where_clause}"
        try:
            self.cursor.execute(sql, values)
            self.connection.commit()
        except sqlite3.Error as e:
            print(f"Error while updating from table {table_name}: {e}")
            self.connection.rollback()

    def delete(self, table_name, where):
        sql = f"DELETE FROM {table_name}"
        where_clause = ""
        values = []
        if where:
            # WHERE arg1 = ? AND arg2 = ?
            where_clause = " WHERE " + " AND ".join(f"{column} = ?" for column in where.keys())
            values = list(where.values())
        sql += where_clause
        try:
            self.cursor.execute(sql, values)
            self.connection.commit()
        except sqlite3.Error as e:
            print(f"Error while deleting from table {table_name}: {e}")
```

2. Создаете пакет **api** для работы с API
``` 
project/
│
├── orm/
│   │
│   ├── __init__.py
│   │
│   ├── db.py
│   │
│   └── custom_orm.py
│
├── api/
│    │   
│    ├── __init__.py
│    │
│    ├── data.py
│    │
│    └── settings.py
│
└── other_files_or_directories/
```

```python
# api/__init__.py

from .settings import response
from .data import high, low

# response - это список объектов
# high/low - это список объектов по возрастаную/убыванию

```

```python
# api/data.py

from api import response

# lambda - безымянная функция
# def func(x):
#     return x * x

# square = lambda x: x * x
# print(square(5))  # Outputs: 25

# Сортировка словарей списка по 'price'

low = sorted(response, key=lambda x: x['price'])
high = sorted(response, key=lambda x: x['price'], reverse=True)

low = [{'name': obj['name'], 'price': obj['price'], 'image': obj['images'][0]['sm']} for obj in low]
high = [{'name': obj['name'], 'price': obj['price'], 'image': obj['images'][0]['sm']} for obj in high]

```

```python
# api/settings.py

import requests

# url - адрес api
url = "https://burgers-hub.p.rapidapi.com/burgers"

# headers - допольнительные данные для запроса если необходимо
headers = {
    "X-RapidAPI-Key": "98fe327f25msh6192b80cbb958d6p15f618jsnb57fcad8edb1",
    "X-RapidAPI-Host": "burgers-hub.p.rapidapi.com"
}

# response - это список всех объектов полученный из API
response = requests.get(url, headers=headers).json()

```

3. Создаете файл **users.py** для работы с таблицой users
``` 
project/
│
├── orm/
│   │
│   ├── __init__.py
│   │
│   ├── db.py
│   │
│   └── custom_orm.py
│
├── api/
│    │   
│    ├── __init__.py
│    │
│    ├── data.py
│    │
│    └── settings.py
│
├── users.py
└── other_files_or_directories/
```

```python
from orm.custom_orm import SQLite3ORM

# Метод для создания нового пользователя
def create(username, first_name, last_name, telegram_id):
    with SQLite3ORM('orm/example.db') as db:
        data = db.select(table_name='users', where={'telegram_id': telegram_id})
        if not data:
            db.insert(
                table_name='users',
                data={
                    'telegram_id': telegram_id,
                    'username': username,
                    'first_name': first_name,
                    'last_name': last_name,
                }
            )
```

3. Создаете файл **utils.py** для работы с разными функциями

``` 
project/
│
├── orm/
│   │
│   ├── __init__.py
│   │
│   ├── db.py
│   │
│   └── custom_orm.py
│
├── api/
│    │   
│    ├── __init__.py
│    │
│    ├── data.py
│    │
│    └── settings.py
│
├── users.py
├── utils.py
└── other_files_or_directories/
```

```python
from orm.custom_orm import SQLite3ORM

def get_history(telegram_id):
    with SQLite3ORM('orm/example.db') as db:
        data = db.select(table_name='history', where={'user_id': telegram_id})
        if data:
            return data
        else:
            return False
```
3. Создаете файл **main.py** для работы с телеграм ботом
   
``` 
project/
│
├── orm/
│   │
│   ├── __init__.py
│   │
│   ├── db.py
│   │
│   └── custom_orm.py
│
├── api/
│    │   
│    ├── __init__.py
│    │
│    ├── data.py
│    │
│    └── settings.py
│
├── users.py
├── utils.py
└── main.py/
```

```python
import telebot
from datetime import datetime
from orm import SQLite3ORM
import users
import utils
from api import low, high, response

# Объект, который позволяет интегрироваться с телеграм ботом через токен
bot = telebot.TeleBot('ВАШ ТОКЕН')

# Команда low 
@bot.message_handler(commands=['low'])
def low_start(message):
    # Создание нового пользователя если его нет в бд
    users.create(
        telegram_id=message.chat.id,
        username=message.from_user.username,
        first_name=message.from_user.first_name,
        last_name=message.from_user.last_name,
    )
    # bot.send_message(chat_id, text) - метод для отправки сообщения
    msg = bot.send_message(chat_id=message.chat.id, text='Напишите кол-во для вывода') # переменная, которая содержить в себе id чата и текст
    # bot.register_next_step_handler(message, function_name) - метод для перехода с одной функции на другую
    bot.register_next_step_handler(msg, low_price_list)


def low_price_list(message):
    # Преброзование message.text в int(), так как message.text является строкой по умолчанию
    number = int(message.text)
    # isdigit() - метод строки который определяет является ли символ числом
    if message.text.isdigit():
        # API
        for obj in low[:number]:
            text = ''
            image = obj['image']
            text += f"<b>{obj['name']}\n</b>"
            text += f"{obj['price']}\n"
            # bot.send_photo(chat_id, caption(необ.), photo, parse_mode(необ.)) - метод для отправки фото + текст если надо
            bot.send_photo(chat_id=message.chat.id, caption=text, photo=image, parse_mode='HTML')
            # Добавляем новую запись в бд
            with SQLite3ORM('orm/example.db') as db:
                db.insert(
                    table_name='history',
                    data={
                        'user_id': message.chat.id,
                        'name': obj['name'],
                        'price': obj['price'],
                        'created_at': datetime.now()
                    }
                )
    # Валидация
    else:
        msg = bot.send_message(chat_id=message.chat.id, text='Введите число а не текст')
        bot.register_next_step_handler(msg, low_price_list)


# Команда high 
@bot.message_handler(commands=['high'])
def high_start(message):
    users.create(
        telegram_id=message.chat.id,
        username=message.from_user.username,
        first_name=message.from_user.first_name,
        last_name=message.from_user.last_name,
    )
    msg = bot.send_message(chat_id=message.chat.id, text='Напишите кол-во для вывода')
    bot.register_next_step_handler(msg, high_price_list)


def high_price_list(message):
    number = int(message.text)
    if message.text.isdigit():
        # API
        for obj in high[:number]:
            text = ''
            image = obj['image']
            text += f"<b>{obj['name']}\n</b>"
            text += f"{obj['price']}\n"
            bot.send_photo(chat_id=message.chat.id, caption=text, photo=image, parse_mode='HTML')
            with SQLite3ORM('orm/example.db') as db:
                db.insert(
                    table_name='history',
                    data={
                        'user_id': message.chat.id,
                        'name': obj['name'],
                        'price': obj['price'],
                        'created_at': datetime.now()
                    }
                )
    else:
        msg = bot.send_message(chat_id=message.chat.id, text='Введите число а не текст')
        bot.register_next_step_handler(msg, high_price_list)


# Команда custom 
@bot.message_handler(commands=['custom'])
def start(message):
    users.create(
        telegram_id=message.chat.id,
        username=message.from_user.username,
        first_name=message.from_user.first_name,
        last_name=message.from_user.last_name,
    )

    msg = bot.send_message(chat_id=message.chat.id, text='Введите минимальную сумму')
    bot.register_next_step_handler(msg, custom_start)


def custom_start(message):
    if message.text.isdigit():
        msg = bot.send_message(chat_id=message.chat.id, text='Введите максимальную сумму')
        bot.register_next_step_handler(msg, custom_end, min_sum=message.text)

    else:
        msg = bot.send_message(chat_id=message.chat.id, text='Введите число а не текст')
        bot.register_next_step_handler(msg, custom_start)


def custom_end(message, min_sum):
    if message.text.isdigit():
        msg = bot.send_message(chat_id=message.chat.id, text='Напишите кол-во для вывода')
        bot.register_next_step_handler(msg, custom_list, min_sum=min_sum, max_sum=message.text)
    else:
        msg = bot.send_message(chat_id=message.chat.id, text='Введите число а не текст')
        bot.register_next_step_handler(msg, custom_end, min_sum)


def custom_list(message, min_sum, max_sum):
    number = int(message.text)
    if message.text.isdigit():
        # Если не понятно, повторите тему генератор списков
        filtered_lst = [obj for obj in response if obj['price'] >= int(min_sum) and obj['price'] <= int(max_sum)]
        for obj in filtered_lst[:number]:
            text = ''
            image = obj['images'][0]['sm']
            text += f"<b>{obj['name']}\n</b>"
            text += f"{obj['price']}\n"
            bot.send_photo(chat_id=message.chat.id, caption=text, photo=image, parse_mode='HTML')
            with SQLite3ORM('orm/example.db') as db:
                db.insert(
                    table_name='history',
                    data={
                        'user_id': message.chat.id,
                        'name': obj['name'],
                        'price': obj['price'],
                        'created_at': datetime.now()
                    }
                )
    else:
        msg = bot.send_message(chat_id=message.chat.id, text='Введите число а не текст')
        bot.register_next_step_handler(msg, custom_list, min_sum, max_sum)


@bot.message_handler(commands=['history'])
def history_start(message):
    users.create(
        telegram_id=message.chat.id,
        username=message.from_user.username,
        first_name=message.from_user.first_name,
        last_name=message.from_user.last_name,
    )

    data = utils.get_history(telegram_id=message.chat.id)
    if data:
        # API => Сами сделаете
        pass
    else:
        bot.send_message(chat_id=message.chat.id, text='Пусто')


# Команда - help
@bot.message_handler(commands=['help'])
def start(message):
    text = '/low — вывод минимальных показателей (с изображением товара/услуги/и так далее)\n'
    text += '/high — вывод максимальных (с изображением товара/услуги/и так далее)\n'
    text += '/custom — вывод показателей пользовательского диапазона (с изображениемтовара/услуги/и так далее)\n'
    text += '/history — вывод истории запросов пользователей'
    bot.send_message(chat_id=message.chat.id, text=text)


# infinity_polling - Метод для непрерывной работы телеграм бота
bot.infinity_polling(skip_pending=True)

```
