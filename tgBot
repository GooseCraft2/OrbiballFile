from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import (
    Application,
    CommandHandler,
    CallbackQueryHandler,
    MessageHandler,
    filters,
    ContextTypes,
)

# ================= НАСТРОЙКИ =================
TOKEN = "8328372236:AAE6paeLYsb_NTdbjEDHqrO-GQnt31z3MUM"  # вставь свой токен
GROUP_CHAT_ID = -4804094994  # вставь ID своей группы
# ============================================

user_names = {}
pending_games = {}

# ====== ЛС БОТА ======
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "Привет! Введи своё имя для игры."
    )

async def save_name(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    user_names[user_id] = update.message.text
    await update.message.reply_text(
        f"Ок, твоё имя сохранено как *{user_names[user_id]}*.\n"
        f"Напиши /game, чтобы предложить начать игру.",
        parse_mode="Markdown"
    )

# ====== ОТПРАВКА В ГРУППУ ======
async def game(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    if user_id not in user_names:
        await update.message.reply_text("Сначала введи имя в ЛС бота.")
        return

    player_name = user_names[user_id]

    keyboard = [
        [
            InlineKeyboardButton("✅ Я в деле", callback_data=f"accept:{player_name}"),
            InlineKeyboardButton("❌ Отказ", callback_data=f"decline:{player_name}"),
        ]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)

    text = f'Игрок *"{player_name}"* предлагает начать игру!'

    # Отправка только в группу
    message = await context.bot.send_message(
        chat_id=GROUP_CHAT_ID,
        text=text,
        reply_markup=reply_markup,
        parse_mode="Markdown"
    )

    pending_games[message.message_id] = {"host": player_name, "players": []}

# ====== ОБРАБОТКА КНОПОК ======
async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    action, host_name = query.data.split(":")
    user_name = user_names.get(query.from_user.id, query.from_user.first_name)

    game_data = pending_games.get(query.message.message_id)
    if not game_data:
        return

    if action == "accept":
        if user_name not in game_data["players"]:
            game_data["players"].append(user_name)
    elif action == "decline":
        if user_name in game_data["players"]:
            game_data["players"].remove(user_name)

    players_text = ", ".join(game_data["players"]) if game_data["players"] else "никого"
    new_text = f'Игрок *"{host_name}"* предлагает начать игру!\n\n✅ Участники: {players_text}'

    await query.edit_message_text(
        text=new_text,
        reply_markup=query.message.reply_markup,
        parse_mode="Markdown"
    )

# ====== ОСНОВНАЯ ФУНКЦИЯ ======
def main():
    app = Application.builder().token(TOKEN).build()

    # Команды в ЛС
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("game", game))
    
    # Обработка имени только в ЛС
    app.add_handler(MessageHandler(
        filters.TEXT & ~filters.COMMAND & filters.ChatType.PRIVATE,
        save_name
    ))

    # Кнопки в группе
    app.add_handler(CallbackQueryHandler(button_handler))

    print("Бот запущен! Жду команд...")
    app.run_polling()

if __name__ == "__main__":
    main()
