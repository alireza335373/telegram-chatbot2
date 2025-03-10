import json
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, CallbackContext
from chatterbot import ChatBot
from chatterbot.trainers import ListTrainer

# ایجاد چت‌بات
chatbot = ChatBot("MyBot")
trainer = ListTrainer(chatbot)

# آموزش اولیه با چند جمله فارسی
trainer.train([
    "سلام",
    "سلام! چطور می‌تونم کمکت کنم؟",
    "اسم تو چیه؟",
    "من یک ربات هوشمند تلگرام هستم!",
    "چه خبر؟",
    "همه چیز خوبه! شما چطورید؟"
])

# توکن ربات تلگرام
TELEGRAM_BOT_TOKEN = "6012990205:AAGOpWHkY1tvEeAN8-bBZnCKimZ7KJH0I_8"

# فایل ذخیره کاربران
USER_DATA_FILE = "users.json"

# بارگذاری اطلاعات کاربران از فایل
def load_users():
    try:
        with open(USER_DATA_FILE, "r", encoding="utf-8") as file:
            return json.load(file)
    except FileNotFoundError:
        return {}

# ذخیره اطلاعات کاربران در فایل
def save_users(users):
    with open(USER_DATA_FILE, "w", encoding="utf-8") as file:
        json.dump(users, file, indent=4, ensure_ascii=False)

# لیست کاربران
users = load_users()

# دستور استارت
async def start(update: Update, context: CallbackContext):
    user_id = str(update.message.from_user.id)
    first_name = update.message.from_user.first_name

    if user_id not in users:
        users[user_id] = {"name": first_name, "messages": []}
        save_users(users)
        await update.message.reply_text(f"سلام {first_name}! از این به بعد تو رو می‌شناسم. 😊")
    else:
        await update.message.reply_text(f"خوش برگشتی {users[user_id]['name']}! دوباره ازم سوال بپرس. 😃")

# پاسخ به پیام‌های کاربران
async def chat(update: Update, context: CallbackContext):
    user_id = str(update.message.from_user.id)
    user_message = update.message.text

    # ذخیره پیام در دیتابیس کاربر
    if user_id in users:
        users[user_id]["messages"].append(user_message)
        save_users(users)

    bot_reply = chatbot.get_response(user_message)
    await update.message.reply_text(str(bot_reply))

# تابع اصلی
def main():
    app = Application.builder().token(TELEGRAM_BOT_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, chat))

    print("✅ Bot is running...")
    app.run_polling()

if __name__ == "__main__":
    main()
