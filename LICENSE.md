from telegram.ext import Updater, CommandHandler
import logging
import os
import time
import threading

# گرفتن توکن از متغیر محیطی Railway
TOKEN = os.getenv("TOKEN")

# شناسه ادمین برای تایید برداشت
ADMIN_ID = 6636522193

# ذخیره‌سازی داده‌ها (موقت - فقط تا وقتی اجرا بازه)
coins = {}
energy = {}
withdraws = {}

# شارژ انرژی هر ۶۰ ثانیه
def recharge():
    while True:
        for uid in energy:
            if energy[uid] < 20:
                energy[uid] += 1
        time.sleep(60)

logging.basicConfig(level=logging.INFO)

def start(update, context):
    uid = update.effective_user.id
    coins.setdefault(uid, 0)
    energy.setdefault(uid, 20)
    update.message.reply_text("👋 سلام! به CatiBot خوش آمدی.\nبا دستور /tap سکه استخراج کن.")

def tap(update, context):
    uid = update.effective_user.id
    if energy.get(uid, 0) > 0:
        energy[uid] -= 1
        coins[uid] = coins.get(uid, 0) + 1
        update.message.reply_text(f"✅ 1 سکه اضافه شد!\n💰 مجموع سکه‌ها: {coins[uid]}\n⚡ انرژی: {energy[uid]}")
    else:
        update.message.reply_text("❌ انرژی شما تمام شده. لطفاً منتظر شارژ بمانید.")

def balance(update, context):
    uid = update.effective_user.id
    update.message.reply_text(f"💰 سکه‌ها: {coins.get(uid,0)}\n⚡ انرژی: {energy.get(uid,0)}")

def withdraw(update, context):
    uid = update.effective_user.id
    if coins.get(uid, 0) < 500:
        update.message.reply_text("حداقل برداشت 500 سکه است.")
        return
    if not context.args:
        update.message.reply_text("آدرس کیف پولت را بنویس:\nمثال: /withdraw EQB....")
        return
    address = context.args[0]
    withdraws[uid] = {"amount": coins[uid], "address": address}
    context.bot.send_message(ADMIN_ID, f"🔔 درخواست برداشت:\n👤 کاربر: {uid}\n💰 مقدار: {coins[uid]}\n🏦 آدرس: {address}")
    update.message.reply_text("درخواست برداشت ارسال شد. پس از تأیید ادمین، واریز انجام می‌شود.")

def approve(update, context):
    if update.effective_user.id != ADMIN_ID:
        return
    if not context.args:
        update.message.reply_text("استفاده: /approve <user_id>")
        return
    uid = int(context.args[0])
    if uid in withdraws:
        context.bot.send_message(uid, f"✅ برداشت شما به مبلغ {withdraws[uid]['amount']} تأیید شد.")
        coins[uid] = 0
        del withdraws[uid]
        update.message.reply_text("تأیید انجام شد.")
    else:
        update.message.reply_text("درخواستی وجود ندارد.")

def main():
    updater = Updater(TOKEN)
    dp = updater.dispatcher

    dp.add_handler(CommandHandler("start", start))
    dp.add_handler(CommandHandler("tap", tap))
    dp.add_handler(CommandHandler("balance", balance))
    dp.add_handler(CommandHandler("withdraw", withdraw, pass_args=True))
    dp.add_handler(CommandHandler("approve", approve, pass_args=True))

    # شروع شارژ انرژی در یک thread جداگانه
    threading.Thread(target=recharge, daemon=True).start()

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
