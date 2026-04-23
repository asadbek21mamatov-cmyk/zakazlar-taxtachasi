# zakazlar_taxtachasi_bot
import telebot
from telebot import types

# Bot tokeningizni kiriting
TOKEN = 'BOT_TOKEN_SHU_YERGA_YOZING'
bot = telebot.TeleBot(TOKEN)

# /start buyrug'i uchun javob
@bot.message_handler(commands=['start'])
def send_welcome(message):
    # Kontakt ulashish tugmasini yaratamiz
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=True)
    contact_button = types.KeyboardButton("📱 Telefon raqamni yuborish", request_contact=True)
    markup.add(contact_button)
    
    bot.send_message(
        message.chat.id, 
        "Assalomu alaykum! Buyurtma berish uchun telefon raqamingizni yuboring.\n\n"
        "Pastdagi '📱 Telefon raqamni yuborish' tugmasini bosing yoki raqamingizni quyidagi formatlardan birida qo'lda yozib yuboring:\n"
        "1) +998 XX XXX XX XX\n"
        "2) XX XXX XX XX", 
        reply_markup=markup
    )

# Kontakt yoki matn kelganda ishlovchi funksiya
@bot.message_handler(content_types=['contact', 'text'])
def handle_phone(message):
    # Raqamni ajratib olamiz
    if message.contact:
        phone = message.contact.phone_number
    else:
        phone = message.text

    # Foydalanuvchi bo'shliqlar (probel) bilan yozgan bo'lsa, ularni olib tashlaymiz
    clean_phone = phone.replace(" ", "")

    # Raqam to'g'riligini tekshirish mantig'i
    is_valid = False
    
    # 1-holat: +998 bilan boshlansa va jami 13 ta belgi bo'lsa (masalan: +998901234567)
    if clean_phone.startswith("+998") and len(clean_phone) == 13 and clean_phone[1:].isdigit():
        is_valid = True
        
    # 2-holat: Faqat 9 ta raqamdan iborat bo'lsa (masalan: 901234567)
    elif len(clean_phone) == 9 and clean_phone.isdigit():
        is_valid = True
        
    # 3-holat: Telegram ba'zan kontaktni "+" belgisiz yuboradi (masalan: 998901234567)
    elif clean_phone.startswith("998") and len(clean_phone) == 12 and clean_phone.isdigit():
        is_valid = True

    # Natijaga qarab javob qaytaramiz
    if is_valid:
        bot.send_message(
            message.chat.id, 
            f"✅ Raqamingiz muvaffaqiyatli qabul qilindi: {phone}\n\nEndi buyurtmangizni yozib yuborishingiz mumkin.",
            reply_markup=types.ReplyKeyboardRemove() # Tugmani ekrandan olib tashlaymiz
        )
        # Shu yerdan boshlab buyurtmani qabul qilish va saqlash mantig'ini yozib ketishingiz mumkin
    else:
        bot.send_message(
            message.chat.id, 
            "❌ Noto'g'ri format!\nIltimos, pastdagi tugma orqali yuboring yoki "
            "raqamni aniq quyidagi formatlardan birida kiriting:\n\n"
            "1) +998 XX XXX XX XX\n"
            "2) XX XXX XX XX"
        )

# Botni uzluksiz ishga tushirish
if __name__ == '__main__':
    print("Bot ishga tushdi...")
    bot.polling(none_stop=True)
