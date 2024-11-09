# python_project
This is a code to summarize the  video faster.
import os
import moviepy.editor as mp
import speech_recognition as sr
from transformers import pipeline

def extract_audio_from_video(video_path, audio_path):
    video = mp.VideoFileClip(video_path)
    video.audio.write_audiofile(audio_path, codec='pcm_s16le')

def transcribe_audio(audio_path):
    recognizer = sr.Recognizer()
    with sr.AudioFile(audio_path) as source:
        audio_data = recognizer.record(source)
        try:
            text = recognizer.recognize_google(audio_data)
            return text
        except sr.UnknownValueError:
            print("Could not understand audio")
        except sr.RequestError:
            print("Could not request results from Google Speech Recognition service")
    return ""

def summarize_text(text):
    summarizer = pipeline("summarization", model="facebook/bart-large-cnn")
    summarized_text = summarizer(text, max_length=150, min_length=50, do_sample=False)
    return summarized_text[0]['summary_text']

def summarize_video(video_path):
    audio_path = "temp_audio.wav"
    extract_audio_from_video(video_path, audio_path)
    text = transcribe_audio(audio_path)
    if text:
        summary = summarize_text(text)
    else:
        summary = "No transcription available."
    if os.path.exists(audio_path):
        os.remove(audio_path)
    return summary

if __name__ == "__main__":
    video_file_path = "your_video.mp4"
    print("Video Summary:")
    print(summarize_video(video_file_path))


