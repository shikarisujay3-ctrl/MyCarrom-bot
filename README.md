import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, MessageHandler, filters, ContextTypes

TOKEN = 8311628791:AAEj8Zwfcv7dyfEUsoJztBEP0pKQzZF2nbk
ADMIN_ID = 123456789  # 👉 নিজের Telegram ID দাও

users = set()

logging.basicConfig(level=logging.INFO)

# START MENU
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    users.add(update.effective_user.id)

    keyboard = [
        [InlineKeyboardButton("📌 KOS ENGINE", callback_data="kos")],
        [InlineKeyboardButton("🐍 SNAKE ENGINE", callback_data="snake")],
        [InlineKeyboardButton("👑 AIM KING", callback_data="aim")],
        [InlineKeyboardButton("🪙 COINS PRICE", callback_data="coins")],
        [InlineKeyboardButton("💎 GEMS & SHOTS", callback_data="gems")],
        [InlineKeyboardButton("🎫 CARROM PASS", callback_data="pass")],
        [InlineKeyboardButton("🛒 OTHER ORDER", callback_data="other")]
    ]

    reply_markup = InlineKeyboardMarkup(keyboard)

    await update.message.reply_text("🔥 *WELCOME TO CARROM STORE* 🔥\n\nSelect Option 👇", reply_markup=reply_markup, parse_mode="Markdown")


# BUTTON HANDLER
async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    data = query.data

    if data == "kos":
        text = """📌 *KOS ENGINE AUTOPLAY*

24 hr 👉 ₹110
7 days 👉 ₹300
15 days 👉 ₹500
30 days 👉 ₹850

📩 Buy: https://t.me/technomind40
"""

    elif data == "snake":
        text = """🐍 *SNAKE ENGINE AUTOPLAY*

3 days 👉 ₹180
10 days 👉 ₹400
30 days 👉 ₹900

📩 Buy: https://t.me/swarnendu111
"""

    elif data == "aim":
        text = """👑 *AIM CARROM KING*

7 days 👉 ₹450
30 days 👉 ₹1200
90 days 👉 ₹2350

📩 Buy: https://t.me/technomind40
"""

    elif data == "coins":
        text = """🪙 *CARROM COINS*

1cr ₹249
2cr ₹499
5cr ₹999
10cr ₹1999

FB ID & Google ID Available
No Coins? No Problem ✅
"""

    elif data == "gems":
        text = """💎 *GEMS & GOLDEN SHOT*

₹129 = 4,400 💎
₹249 = 11,000 💎
₹499 = 22,000 💎
₹1249 = 55,000 💎
₹2499 = 110,000 💎

⚠ Limited Offer
"""

    elif data == "pass":
        text = """🎫 *CARROM PASS*

Premium 👉 ₹100
Premium Plus 👉 ₹300
"""

    elif data == "other":
        text = "✍️ What do you want?\n\nType your order..."
        context.user_data["custom"] = True

    else:
        text = "Error ❌"

    await query.edit_message_text(text, parse_mode="Markdown")


# CUSTOM ORDER → ADMIN
async def message_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if context.user_data.get("custom"):
        user = update.effective_user

        msg = f"""📩 *NEW ORDER*

👤 ID: {user.id}
👤 Name: {user.first_name}

📝 Message:
{update.message.text}
"""

        await context.bot.send_message(chat_id=ADMIN_ID, text=msg, parse_mode="Markdown")

        await update.message.reply_text("✅ Order sent to admin!")
        context.user_data["custom"] = False


# BROADCAST
async def broadcast(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_user.id != ADMIN_ID:
        return

    msg = " ".join(context.args)

    for u in users:
        try:
            await context.bot.send_message(u, msg)
        except:
            pass

    await update.message.reply_text("✅ Broadcast Sent!")


# MAIN
app = ApplicationBuilder().token(TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(CallbackQueryHandler(button))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, message_handler))
app.add_handler(CommandHandler("broadcast", broadcast))

app.run_polling()
