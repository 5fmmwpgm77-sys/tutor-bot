import logging
from telegram import Update, ReplyKeyboardMarkup, KeyboardButton
from telegram.ext import (
    Application,
    CommandHandler,
    MessageHandler,
    ContextTypes,
    filters,
    ConversationHandler,
)

# Включим логирование
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO
)

# Состояния для ConversationHandler
(
    ASK_NAME,
    ASK_SUBJECT,
    ASK_CLASS,
    ASK_GRADE,
    ASK_GOALS,
    ASK_CONTACT,
) = range(6)

# Хранилище данных (в реальном проекте — база данных)
user_data = {}

# Главное меню
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [KeyboardButton("📝 Записаться")],
        [KeyboardButton("👨‍🏫 О репетиторе"), KeyboardButton("❓ FAQ")],
        [KeyboardButton("🎁 Приведи друга")],
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True, one_time_keyboard=False)
    await update.message.reply_text(
        "Привет! 👋\nЯ — помощник Матвея.\n\n"
        "✅ Онлайн или очно (Автозаводский район)\n"
        "✅ Занятия по 55 минут\n"
        "✅ Пробное занятие — бесплатно!\n"
        "✅ Цена: 1000 ₽ за урок\n\n"
        "Выбери действие:",
        reply_markup=reply_markup,
    )

# Обработка кнопки "О репетиторе"
async def about(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "👨‍🏫 Я — репетитор по математике с трехлетним опытом\n\n"
        "• Помогаю ученикам 5–9 классов разобраться в темах и повысить оценки\n"
        "• Готовлю к ОГЭ\n"
        "• Все занятия — 55 минут, онлайн или очно (Автозаводский район)\n"
        "• Первое занятие — бесплатно!\n\n"
        "По биологии сотрудничаю с проверенными репетиторами — запишись, и я передам твои данные лично!"
    )

# Обработка кнопки "FAQ"
async def faq(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "❓ Часто задаваемые вопросы:\n\n"
        "1️⃣ Сколько стоит занятие?\n— 1000 ₽ за 55 минут. Первое — бесплатно.\n\n"
        "2️⃣ Как проходит онлайн-урок?\n— Через  ВК-звонки + интерактивная доска.\n\n"
        "3️⃣ Есть ли занятия по биологии?\n— Да! Я передам тебя проверенному репетитору-партнёру.\n\n"
        "4️⃣ Где очные занятия?\n— Только в Нижнем Новгороде.\n\n"
        "5️⃣ Можно ли перенести урок?\n— Да, если предупредить за 6+ часов.\n\n"
        "6️⃣ Готовите к ОГЭ?\n— Да! Программа под твои цели.\n\n"
        "7️⃣ Как оплатить?\n—  Наличными или на карту после занятия."
    )

# Обработка кнопки "Приведи друга"
async def referral(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🎁 Приведи друга — получи бонус!\n\n"
        "• Приведи друга на **математику** — получи **1000 ₽ скидку** на следующее занятие, а твой друг получит бесплатное занятие и отличного репетитора😁.\n"
        "• Приведи друга на **биологию** — получи **200 ₽ скидку** на математику!\n\n"
        "Как это работает:\n"
        "1. Друг пишет в бота и указывает твой промокод при записи.\n"
        "2. После его первого платного занятия - для тебя доступны бонусы .\n\n"
        "Делись ссылкой: https://t.me/my_tutor_helper_bot  "
    )

# Начало записи
async def handle_signup(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Хорошо! Давай начнём запись.\n\nКак тебя зовут?")
    return ASK_NAME

# Получение имени
async def ask_name(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    name = update.message.text.strip()
    if not name:
        await update.message.reply_text("Пожалуйста, введи своё имя.")
        return ASK_NAME
    user_data[user_id] = {"name": name}
    # Выбор предмета
    keyboard = [
        [KeyboardButton("🔢 Математика")],
        [KeyboardButton("🔬 Биология")],
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True, one_time_keyboard=True)
    await update.message.reply_text("Выбери предмет:", reply_markup=reply_markup)
    return ASK_SUBJECT

# Выбор предмета
async def ask_subject(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    subject = update.message.text
    if subject not in ["🔢 Математика", "🔬 Биология"]:
        await update.message.reply_text("Пожалуйста, выбери предмет из кнопок.")
        return ASK_SUBJECT
    user_data[user_id]["subject"] = "Математика" if "Математика" in subject else "Биология"
    # Класс (для обоих предметов)
    keyboard = [[KeyboardButton(str(i)) for i in range(5, 10)]]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True, one_time_keyboard=True)
    await update.message.reply_text("В каком ты классе? (5–9)", reply_markup=reply_markup)
    return ASK_CLASS

# Класс
async def ask_class(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    try:
        grade = int(update.message.text)
        if grade not in range(5, 10):
            raise ValueError
        user_data[user_id]["class"] = grade
    except ValueError:
        await update.message.reply_text("Пожалуйста, выбери класс от 5 до 9.")
        return ASK_CLASS
    # Текущая оценка
    keyboard = [[KeyboardButton("5"), KeyboardButton("4")], [KeyboardButton("3"), KeyboardButton("2")]]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True, one_time_keyboard=True)
    await update.message.reply_text("Какая у тебя текущая оценка по этому предмету?", reply_markup=reply_markup)
    return ASK_GRADE

# Оценка
async def ask_grade(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    grade = update.message.text
    if grade not in ["5", "4", "3", "2"]:
        await update.message.reply_text("Пожалуйста, выбери оценку из кнопок.")
        return ASK_GRADE
    user_data[user_id]["current_grade"] = grade
    # Цели
    keyboard = [
        [KeyboardButton("Повысить оценку")],
        [KeyboardButton("Подготовка к ОГЭ")],
        [KeyboardButton("Разобрать пробелы")],
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True, one_time_keyboard=True)
    await update.message.reply_text("Какая твоя цель?", reply_markup=reply_markup)
    return ASK_GOALS

# Цели
async def ask_goals(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    goal = update.message.text
    if goal not in ["Повысить оценку", "Подготовка к ОГЭ", "Разобрать пробелы"]:
        await update.message.reply_text("Пожалуйста, выбери цель из кнопок.")
        return ASK_GOALS
    user_data[user_id]["goal"] = goal
    # Контакт
    await update.message.reply_text(
        "Отлично! Остался последний шаг.\n\n"
        "Напиши, как с тобой связаться (Telegram @username или номер телефона):"
    )
    return ASK_CONTACT

# Контакт и завершение
async def ask_contact(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    contact = update.message.text.strip()
    if not contact:
        await update.message.reply_text("Пожалуйста, укажи способ связи.")
        return ASK_CONTACT
    user_data[user_id]["contact"] = contact

    data = user_data[user_id]
    subject = data["subject"]
    partner_msg = " (я передам твои данные проверенному репетитору-партнёру)" if subject == "Биология" else ""

    await update.message.reply_text(
        f"✅ Спасибо, {data['name']}!\n\n"
        f"Ты записался на **бесплатное пробное занятие** по **{subject}** ({data['class']} класс).\n"
        f"Текущая оценка: {data['current_grade']}\n"
        f"Цель: {data['goal']}\n\n"
        f"Связь: {contact}\n\n"
        f"В ближайшее время репетитор напишет тебе, чтобы назначить удобное время.{partner_msg}\n\n"
        f"Удачи! 🌟"
    )
    # Здесь можно отправить уведомление тебе (например, в личку)
    return ConversationHandler.END

# Отмена
async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Запись отменена. Возвращайся в любое время!")
    return ConversationHandler.END

# Обработка неизвестных команд
async def unknown(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    if text == "📝 Записаться":
        return await handle_signup(update, context)
    elif text == "👨‍🏫 О репетиторе":
        return await about(update, context)
    elif text == "❓ FAQ":
        return await faq(update, context)
    elif text == "🎁 Приведи друга":
        return await referral(update, context)
    else:
        await update.message.reply_text("Используй кнопки меню 👇")

# Основная функция
def main():
    # 🔑 ВСТАВЬ СЮДА СВОЙ ТОКЕН ОТ @BotFather
    TOKEN = "7875824140:AAF3M9smNkQ9BVhlXf1kF3V7OKIQQqNsUx4"

    application = Application.builder().token(TOKEN).build()

    # ConversationHandler для записи
    conv_handler = ConversationHandler(
        entry_points=[MessageHandler(filters.Regex("^📝 Записаться$"), handle_signup)],
        states={
            ASK_NAME: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_name)],
            ASK_SUBJECT: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_subject)],
            ASK_CLASS: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_class)],
            ASK_GRADE: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_grade)],
            ASK_GOALS: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_goals)],
            ASK_CONTACT: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_contact)],
        },
        fallbacks=[CommandHandler("cancel", cancel)],
    )

    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.Regex("^👨‍🏫 О репетиторе$"), about))
    application.add_handler(MessageHandler(filters.Regex("^❓ FAQ$"), faq))
    application.add_handler(MessageHandler(filters.Regex("^🎁 Приведи друга$"), referral))
    application.add_handler(conv_handler)
    application.add_handler(MessageHandler(filters.ALL, unknown))

    print("Бот запущен...")
    application.run_polling()

if __name__ == "__main__":
    main()
