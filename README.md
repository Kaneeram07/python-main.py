# python-main.py
from telegram import Update
from telegram.ext import ApplicationBuilder, MessageHandler, ContextTypes, filters
import logging

# Logging for debugging
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# Your bot token and admin user ID
BOT_TOKEN = "7845025620:AAHI056VE0fz9_vJA5p50IbjIMVmiEMCAq8"
ADMIN_ID = 5996998779

# Store user requests temporarily (can upgrade to DB later)
user_requests = {}

# Handle user message (movie/product request)
async def handle_user_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    username = update.message.from_user.username or update.message.from_user.first_name
    message = update.message.text

    user_requests[user_id] = message

    await update.message.reply_text("Thanks! We’ll send your download/buy link within 24 hours.")

    await context.bot.send_message(
        chat_id=ADMIN_ID,
        text=f"New request from @{username} (ID: {user_id}):\n{message}"
    )

# Admin sends /reply user_id message to respond
async def handle_admin_reply(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.message.chat_id != ADMIN_ID:
        return  # Ignore if not from admin

    if update.message.text.startswith("/reply"):
        try:
            _, user_id_str, *reply_msg = update.message.text.split()
            user_id = int(user_id_str)
            response = " ".join(reply_msg)
            await context.bot.send_message(chat_id=user_id, text=response)
            await update.message.reply_text("Reply sent successfully!")
        except Exception as e:
            await update.message.reply_text(f"Error: {e}")

# Setup app
app = ApplicationBuilder().token(BOT_TOKEN).build()
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_user_message))
app.add_handler(MessageHandler(filters.TEXT & filters.COMMAND, handle_admin_reply))
app.run_polling()
python-telegram-bot==20.3
