import telebot
import logging
from telebot import types
import datetime

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

TOKEN = '7236885928:AAHU_sIA_akTA83TNJyO4dHypciGAix5bc'


bot = telebot.TeleBot(TOKEN)

reservations = {}

def format_date(date_obj):
  return date_obj.strftime('%d.%m.%Y')


def book_hotel(message):
  bot.send_message(message.chat.id, "Введите данные для бронирования отеля "
                                    "(название, дата заезда, дата выезда):")
  bot.register_next_step_handler(message, process_hotel_booking)

def process_hotel_booking(message):
  try:
    data = message.text.split(',')
    name = data[0].strip()
    date_in = datetime.datetime.strptime(data[1].strip(), '%d.%m.%Y').date()
    date_out = datetime.datetime.strptime(data[2].strip(), '%d.%m.%Y').date()

    reservation_id = len(reservations.get(message.chat.id, []))
    reservations[message.chat.id] = (reservations.get(message.chat.id, []) +
                                     [{"id": reservation_id, "type": "hotel", "name": name,
                                       "date_in": format_date(date_in), "date_out": format_date(date_out)}])
    bot.send_message(message.chat.id, f"Бронирование отеля '{name}' "
                                      f"с {format_date(date_in)} по {format_date(date_out)} "
                                      f"успешно забронировано. ID брони: {reservation_id + 1}")
  except (ValueError, IndexError) as e:
    bot.send_message(message.chat.id, f"Ошибка: {e}. "
                                      f"Пожалуйста, введите данные в правильном формате "
                                      f"(название, дата заезда, дата выезда через запятую, например: "
                                      f"Hilton, 15.10.2024, 17.10.2024).")


def show_reservations(message):
  user_reservations = reservations.get(message.chat.id)
  if user_reservations:
    reservations_list = "\n".join(
        f"ID: {reservation['id']}, Тип: {reservation['type']}, "
        f"Название: {reservation['name']}, Дата заезда: {reservation['date_in']}, "
        f"Дата выезда: {reservation['date_out']}"
        for reservation in user_reservations
    )
    bot.send_message(message.chat.id, reservations_list)
  else:
    bot.send_message(message.chat.id, "У вас нет бронирований.")


@bot.message_handler(commands=['start'])
def start(message):
  markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
  markup.add("Забронировать отель")
  markup.add("Мои бронирования")
  bot.send_message(message.chat.id, "Выберите действие:", reply_markup=markup)
  bot.register_next_step_handler(message, process_start)


def process_start(message):
    if message.text == "Забронировать отель":
        book_hotel(message)
    elif message.text == "Мои бронирования":
        show_reservations(message)
    else:
        bot.reply_to(message, "Неверное действие.")


if __name__ == "__main__":
    bot.infinity_polling()
