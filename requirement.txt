import telebot
import requests
import os

# Render ke environment variables se keys uthaenge
TELEGRAM_TOKEN = os.environ.get("TELEGRAM_TOKEN")
GEMINI_KEY = os.environ.get("GEMINI_KEY")

bot = telebot.TeleBot(TELEGRAM_TOKEN)

# Gemini API se baat karne ka function
def get_gemini_reply(user_message):
    url = f"https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key={GEMINI_KEY}"
    headers = {'Content-Type': 'application/json'}
    data = {
         "contents": [{"parts":[{"text": user_message}]}]
    }
    
    try:
        response = requests.post(url, headers=headers, json=data)
        result = response.json()
        
        if 'error' in result:
            return f"API Error: {result['error']['message']}"
            
        answer = result['candidates'][0]['content']['parts'][0]['text']
        return answer
    except Exception as e:
        print("API Error:", e)
        return "Sorry bhai, API ya network mein kuch issue hai."

# Telegram se message receive aur reply karne ka function
@bot.message_handler(func=lambda message: True)
def reply_to_user(message):
    try:
        bot.send_chat_action(message.chat.id, 'typing')
        ai_response = get_gemini_reply(message.text)
        bot.reply_to(message, ai_response)
    except Exception as e:
        bot.reply_to(message, "Kuch problem aayi hai bot mein.")
        print("Bot Error:", e)

# Bot start karna
print("Aapka Smart Gemini Bot chalu ho gaya hai!")
bot.infinity_polling()
