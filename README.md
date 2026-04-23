import os
import re
import logging
import telebot
from telebot import types

# 1. Loglarni sozlash (Railway'da xatolarni kuzatib borish uchun juda muhim)
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# 2. Xavfsiz Token: Tokenni GitHub'da ochiq qoldirmaslik uchun muhit o'zgaruvchisidan (Environment Variable) olamiz.
TOKEN = os.getenv('BOT_TOKEN', 'BU_YERGA_TOKENINGIZNI_YOZING_YOKI_BO_SH_QOLDIRING')
bot = telebot.TeleBot(TOKEN)

# Start buyrug'i
@bot.message_handler(commands=['start'])
def send_welcome(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=True)
    contact_button = types.KeyboardButton("📱 Telefon raqamni yuborish", request_contact=True)
    markup.add(contact_button)
    
    bot.send_message(
        message.chat.id, 
        "Assalomu alaykum! Buyurtma berish uchun telefon raqamingizni yuboring.\n\n"
        "Pastdagi tugmani bosing yoki raqamingizni kiriting (Masalan: +998 90 123 45 67):", 
        reply_markup=markup
    )

# Raqamni qabul qilish va tekshirish
@bot.message_handler(content_types=['contact', 'text'])
def handle_phone(message):
    try:
        if message.contact:
            phone = message.contact.phone_number
        else:
            phone = message.text

        # Barcha harf va belgilarni olib tashlab, faqat raqamlarni qoldiramiz
        clean_phone = re.sub(r'\D', '', phone)
        is_valid = False

        # 9 xonali raqam yozilsa (masalan: 901234567) -> avtomat 998 qo'shamiz
        if len(clean_phone) == 9:
            clean_phone = '998' + clean_phone
            is_valid = True
        # 12 xonali va 998 bilan boshlangan bo'lsa
        elif len(clean_phone) == 12 and clean_phone.startswith('998'):
            is_valid = True

        if is_valid:
            # Raqamni chiroyli formatga keltiramiz: +998 90 123 45 67
            formatted_phone = f"+{clean_phone[:3]} {clean_phone[3:5]} {clean_phone[5:8]} {clean_phone[8:10]} {clean_phone[10:12]}"
            
            bot.send_message(
                message.chat.id, 
                f"✅ Raqamingiz qabul qilindi: {formatted_phone}\n\n"
                "📦 Endi nima buyurtma qilmoqchi ekanligingizni matn ko'rinishida yozib yuboring:",
                reply_markup=types.ReplyKeyboardRemove()
            )
            # Botni keyingi qadamga (buyurtmani eshitishga) o'tkazamiz
            bot.register_next_step_handler(message, process_order, formatted_phone)
        else:
            bot.send_message(
                message.chat.id, 
                "❌ Noto'g'ri format!\nIltimos, raqamni to'g'ri kiriting (Masalan: +998901234567 yoki 901234567)."
            )
            
    except Exception as e:
        logging.error(f"Raqamni tekshirishda xatolik: {e}")
        bot.send_message(message.chat.id, "Kechirasiz, tizimda xatolik yuz berdi. Qaytadan urinib ko'ring.")

# Buyurtmani qabul qilish qadami
def process_order(message, phone_number):
    try:
        order_text = message.text
        user_name = message.from_user.first_name
        
        # Bu yerda ma'lumotlarni bazaga saqlashingiz yoki o'zingizning shaxsiy lichkangizga (admin guruhga) yuborishingiz mumkin
        admin_id = 'OZI_ID_RAQAMINGIZNI_YOZING' # O'zingizning Telegram ID raqamingiz
        
        # Foydalanuvchiga javob
        bot.send_message(
            message.chat.id, 
            f"🎉 Rahmat, {user_name}! Buyurtmangiz qabul qilindi.\n\n"
            f"📞 Sizning raqamingiz: {phone_number}\n"
            f"🛒 Buyurtma: {order_text}\n\n"
            "Tez orada operatorlarimiz siz bilan bog'lanishadi."
        )
        
        # Adminga xabar berish (ixtiyoriy, agar admin ID yozilgan bo'lsa ishlaydi)
        if admin_id != 'OZI_ID_RAQAMINGIZNI_YOZING':
            bot.send_message(
                admin_id,
                f"🚨 YANGI BUYURTMA!\n\nMijoz: {user_name}\nRaqam: {phone_number}\nBuyurtma: {order_text}"
            )
            
    except Exception as e:
        logging.error(f"Buyurtmani qabul qilishda xatolik: {e}")
        bot.send_message(message.chat.id, "Kechirasiz, buyurtmani qabul qilishda xatolik yuz berdi.")

if __name__ == '__main__':
    logging.info("Bot muvaffaqiyatli ishga tushdi...")
    # Bot internet uzilib qolsa ham to'xtab qolmasligi uchun timeout va non_stop parametrlarini beramiz
    bot.infinity_polling(timeout=10, long_polling_timeout=5)
