from telegram import InlineKeyboardButton, InlineKeyboardMarkup, WebAppInfo, Update
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes

TOKEN = "7516647507:AAFTbash8SoSVGsvSiohuhr7t3OKHmpDvBI"

WEBAPP_URL = "https://your-vercel-project.vercel.app"  # آدرس WebApp خودت رو اینجا بذار

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [InlineKeyboardButton("💎 شروع بازی نات کوین", web_app=WebAppInfo(url=WEBAPP_URL))]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text("برای شروع بازی روی دکمه زیر کلیک کن:", reply_markup=reply_markup)

if __name__ == "__main__":
    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))

    print("🤖 ربات نات کوین اجرا شد.")
    app.run_polling()
