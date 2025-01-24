from telegram import Update
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext
import openai
from moviepy.editor import TextClip, CompositeVideoClip, AudioFileClip

# Set up OpenAI API (for text generation)
openai.api_key = "YOUR_OPENAI_API_KEY"

def generate_video(text):
    # Generate AI voice (or use Google TTS)
    from gtts import gTTS
    tts = gTTS(text)
    tts.save("audio.mp3")
    
    # Create text overlay
    text_clip = TextClip(text, fontsize=50, color='white').set_duration(10)
    
    # Combine with audio
    audio = AudioFileClip("audio.mp3")
    final_clip = CompositeVideoClip([text_clip.set_audio(audio)])
    final_clip.write_videofile("output.mp4", fps=24)
    return "output.mp4"

def handle_text(update: Update, context: CallbackContext):
    text = update.message.text
    video_path = generate_video(text)
    update.message.reply_video(video=open(video_path, 'rb'))

def main():
    bot_token = "YOUR_TELEGRAM_BOT_TOKEN"
    updater = Updater(bot_token, use_context=True)
    dp = updater.dispatcher
    dp.add_handler(MessageHandler(Filters.text & ~Filters.command, handle_text))
    updater.start_polling()
    updater.idle()

if __name__ == "__main__":
    main()
