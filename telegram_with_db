1. Создаем файл main.py
```python
import telebot
import keyboard
import content
import utils
import users


# Объект, который позволяет интегрироваться с телеграм ботом через токен
bot = telebot.TeleBot('token')


@bot.message_handler(commands=['start'])
def start(message):
    users.is_exist(message.chat.id)
    if not users.is_exist(message.chat.id):
        users.create(
            telegram_id=message.chat.id,
            username=message.from_user.username,
            first_name=message.from_user.first_name,
            last_name=message.from_user.last_name,
        )


# infinity_polling - Метод для непрерывной работы телеграм бота
bot.infinity_polling(skip_pending=True)
```
