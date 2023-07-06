# Как настроить Telegram бота через Python

1. Создает телеграм бота через бот @BotFather
2. Скачать последнюю версию библиотеки pyTelegramBotAPI через PyCharm:
    * Снизу, справа в углу нажимает на Python => Interpretator Settings => Нажимаем на + => Вставляем в поисковик pyTelegramBotAPI(нажимаем на галочку Specify version)
3. Создаем файл main.py
```python
import telebot
from telebot.types import ReplyKeyboardMarkup, ReplyKeyboardRemove

# Объект, который позволяет интегрироваться с телеграм ботом через токен
bot = telebot.TeleBot('Ваш Токен')


# Обработчик срабатывающий при написании команды /start
@bot.message_handler(commands=['start'])
def start(message):
    # Объект для создания кнопок
    markup = ReplyKeyboardMarkup(resize_keyboard=True)
    row_1 = ['Кнопка 1', 'Кнопка 2']
    row_2 = ['Кнопка 3', 'Кнопка 4']
    # add - метод который добавляет кнопку в объект(контейнер) markup
    markup.add(*row_1)
    markup.add(*row_2)
    # msg - это просто переменная которая хранит в себе информацию о сообщении
    msg = bot.send_message(chat_id=message.chat.id, text='Вы нажали на команду /start', reply_markup=markup)
    # register_next_step_handler(1 - Что отправить, 2 - Куда отправить)
    bot.register_next_step_handler(msg, menu)


def menu(message):
    if message.text == 'Кнопка 1':
        bot.send_message(chat_id=message.chat.id, text='Вы нажали на Кнопка 1',
                               reply_markup=ReplyKeyboardRemove()) # Удаляет клавиатуру с экрана
    elif message.text == 'Кнопка 2':
        bot.send_message(chat_id=message.chat.id, text='Вы нажали на Кнопка 2',
                               reply_markup=ReplyKeyboardRemove()) # Удаляет клавиатуру с экрана
    elif message.text == 'Получить':
        bot.send_message(chat_id=message.chat.id, text='Вы нажали на Кнопка 3',
                               reply_markup=ReplyKeyboardRemove()) # Удаляет клавиатуру с экрана
    elif message.text == 'Удалить':
       bot.send_message(chat_id=message.chat.id, text='Вы нажали на Кнопка 4',
                               reply_markup=ReplyKeyboardRemove()) # Удаляет клавиатуру с экрана
    else:
        msg = bot.send_message(chat_id=message.chat.id, text='Не правильная команда!, попробуйте еще раз!')
        bot.register_next_step_handler(msg, menu)

# infinity_polling - Метод для непрерывной работы телеграм бота
bot.infinity_polling(skip_pending=True)
