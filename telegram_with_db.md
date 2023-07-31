1. Создаем файл main.py
```python
import telebot
import users


# Объект, который позволяет интегрироваться с телеграм ботом через токен
bot = telebot.TeleBot('token')


@bot.message_handler(commands=['start'])
def start(message):
    # Функция которая проверяет наличин user в бд
    users.is_exist(message.chat.id)
    if not users.is_exist(message.chat.id):
        # Функция для создрания нового user в бд
        users.create(
            telegram_id=message.chat.id,
            username=message.from_user.username,
            first_name=message.from_user.first_name,
            last_name=message.from_user.last_name,
        )


# infinity_polling - Метод для непрерывной работы телеграм бота
bot.infinity_polling(skip_pending=True)
```

2. Создаем файл db.py
```python
import sqlite3

with sqlite3.connect('test.db') as db:
    # объект для работы с базой данных
    cursor = db.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            telegram_id INTEGER,
            first_name TEXT,
            last_name TEXT,
            username TEXT 
        )
    ''')
```

3. Создаем файл users.py
```python
import sqlite3


def create(username, first_name, last_name, telegram_id):
    with sqlite3.connect('test.db') as db:
        cursor = db.cursor()
        sql = f'''
            INSERT INTO users (telegram_id, first_name, last_name, username) 
            VALUES (
                '{telegram_id}',
                '{first_name if first_name else None}', 
                '{last_name if last_name else None}', 
                '{username if username else None}'
                )
            '''
        cursor.execute(sql)
        db.commit()


def is_exist(telegram_id):
    with sqlite3.connect('test.db') as db:
        cursor = db.cursor()
        sql = f'''
            SELECT telegram_id FROM users WHERE telegram_id == {telegram_id}
        '''
        cursor.execute(sql)
        data = cursor.fetchone()
        if data:
            return True
```
