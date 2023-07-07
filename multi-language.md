# Мультиязычный телеграм бот

1. Создаете файл **content.py** для хранения текстов в виде словаря
```python
greeting_text = 'Выберите язык \nChoose the language \n👇👇👇'

menu_text = {
    'ru': 'Выберите раздел',
    'en': 'Choose the section'
}

menu_btns = {
    'ru': {'opt_1': 'Кнопка 1', 'opt_2': 'Кнопка 2', 'opt_3':'Кнопка 3', 'opt_4':'Кнопка 4'},
    'en': {'opt_1': 'Button 1', 'opt_2': 'Button 2', 'opt_3':'Button 3', 'opt_4':'Button 4'}
}
```

2. Создаете файл **keyboard.py** для создания кнопок
```python
from telebot.types import ReplyKeyboardMarkup
import content

def get_language():
    text = content.greeting_text
    # Объект для создания кнопок
    markup = ReplyKeyboardMarkup(resize_keyboard=True)
    row = ['Русский', 'English']
    # add - метод который добавляет кнопку в объект(контейнер) markup
    markup.add(*row)
    return markup, text


def get_menu(language):
    text = content.menu_text[language]
    markup = ReplyKeyboardMarkup(resize_keyboard=True)
    lst = [value for value in content.menu_btns[language].values()]
    # Срез списка
    row_1 = lst[:2]
    row_2 = lst[2:]
    markup.add(*row_1)
    markup.add(*row_2)
    return markup, text
```
3. Создаете файл **utils.py** для различных функций например: Валидация
```python
def language_validator(language):
    if language == 'Русский':
        return 'ru'
    elif language == 'English':
        return 'en'
    else:
        return None
```
4. Создаем файл main.py
```python
import telebot
import keyboard
import content
import utils

# Объект, который позволяет интегрироваться с телеграм ботом через токен
bot = telebot.TeleBot('token')


@bot.message_handler(commands=['start'])
def start(message):
    # Деструктуризация
    markup, text = keyboard.get_language()
    # msg - это просто переменная, которая хранит в себе сообщение для отправки
    msg = bot.send_message(chat_id=message.chat.id, text=text, reply_markup=markup)
    # Метод для перехода на другую функцию
    # register_next_step_handler() - Принимает два аргумента:
    # 1 - сообщение, 2 - название функции для перехода, 3 - отправка дополнительно аргумента(необязательно)
    bot.register_next_step_handler(msg, menu)


def menu(message):
    # Функция для валидации языка
    language = utils.language_validator(message.text)
    if language is not None:
        markup, text = keyboard.get_menu(language)
        msg = bot.send_message(chat_id=message.chat.id, text=text, reply_markup=markup)
        bot.register_next_step_handler(msg, menu_options, language=language)
    else:
        markup, text = keyboard.get_language()
        msg = bot.send_message(chat_id=message.chat.id, text=text, reply_markup=markup)
        bot.register_next_step_handler(msg, menu)


def menu_options(message, language):
    if message.text == content.menu_btns[language]['opt_1']:
        bot.send_message(chat_id=message.chat.id, text='1', )
    elif message.text == content.menu_btns[language]['opt_2']:
        bot.send_message(chat_id=message.chat.id, text='2', )
    elif message.text == content.menu_btns[language]['opt_3']:
        bot.send_message(chat_id=message.chat.id, text='3', )
    elif message.text == content.menu_btns[language]['opt_4']:
        bot.send_message(chat_id=message.chat.id, text='4', )
    else:
        markup, text = keyboard.get_menu(language)
        msg = bot.send_message(chat_id=message.chat.id, text=text, reply_markup=markup)
        bot.register_next_step_handler(msg, menu_options, language=language)


# infinity_polling - Метод для непрерывной работы телеграм бота
bot.infinity_polling(skip_pending=True)
```
