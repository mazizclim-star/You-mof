# You-mof
import streamlit as st
import yt_dlp
import whisper
from deep_translator import GoogleTranslator
import os
import datetime

def format_timestamp(seconds: float):
    td = datetime.timedelta(seconds=seconds)
    total_seconds = int(td.total_seconds())
    hours = total_seconds // 3600
    minutes = (total_seconds % 3600) // 60
    secs = total_seconds % 60
    millis = int(td.microseconds / 1000)
    return f"{hours:02d}:{minutes:02d}:{secs:02d},{millis:03d}"

def process_video(url):
    ydl_opts = {
        'format': 'm4a/bestaudio/best',
        'outtmpl': 'temp_audio.%(ext)s',
        'postprocessors': [{'key': 'FFmpegExtractAudio', 'preferredcodec': 'm4a'}],
    }
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        ydl.download([url])
    model = whisper.load_model("tiny")
    result = model.transcribe("temp_audio.m4a")
    srt_content = ""
    translator = GoogleTranslator(source='auto', target='ar')
    for i, segment in enumerate(result['segments']):
        start = format_timestamp(segment['start'])
        end = format_timestamp(segment['end'])
        translated_text = translator.translate(segment['text'])
        srt_content += f"{i+1}\n{start} --> {end}\n{translated_text}\n\n"
    return srt_content

st.title("🎥 You Mof - مترجم يوتيوب")
url = st.text_input("أدخل رابط الفيديو:")
if st.button("ترجم الآن"):
    if url:
        try:
            with st.spinner("جاري العمل..."):
                final_srt = process_video(url)
                st.success("تمت الترجمة!")
                st.download_button("📥 تحميل ملف SRT", data=final_srt, file_name="You_Mof.srt")
                if os.path.exists("temp_audio.m4a"): os.remove("temp_audio.m4a")
        except Exception as e: st.error(f"خطأ: {e}")











        
