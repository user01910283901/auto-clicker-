import tkinter as tk
from tkinter import ttk
from threading import Thread
from pynput.mouse import Controller, Button
from pynput.keyboard import Listener, Key
import time

# Глобальные переменные
running = False
click_interval = 1.0
mouse = Controller()
start_stop_key = Key.f6  # FN+F6 или ALT+F6 для запуска/остановки

# Функция для автокликера
def auto_clicker():
    global running
    while running:
        mouse.click(Button.left, 1)
        time.sleep(click_interval)

# Обработчики горячих клавиш
def toggle_clicker(key):
    global running
    if key == start_stop_key:  # Если нажата F6
        running = not running
        if running:
            Thread(target=auto_clicker, daemon=True).start()

# Основное окно
def create_gui():
    global click_interval
    root = tk.Tk()
    root.title("Auto Clicker 13")
    root.geometry("400x300")
    root.configure(bg="#202020")

    # Заголовок
    title = tk.Label(root, text="Auto Clicker 13", font=("Arial", 24, "bold"), bg="#202020", fg="#FFFF00")
    title.pack(pady=10)

    # Выбор интервала
    frame = tk.Frame(root, bg="#202020")
    frame.pack(pady=20)

    tk.Label(frame, text="Интервал кликов:", font=("Arial", 14), bg="#202020", fg="#FFFFFF").grid(row=0, column=0, padx=5)
    interval_entry = tk.Entry(frame, font=("Arial", 14), width=10)
    interval_entry.grid(row=0, column=1, padx=5)

    # Единицы времени
    interval_type = ttk.Combobox(frame, font=("Arial", 14), state="readonly", width=10)
    interval_type['values'] = ["Миллисекунды", "Секунды", "Минуты"]
    interval_type.current(1)  # По умолчанию "Секунды"
    interval_type.grid(row=0, column=2, padx=5)

    # Установка интервала
    def set_interval():
        global click_interval
        try:
            time_value = float(interval_entry.get())
            unit = interval_type.get()
            if unit == "Миллисекунды":
                click_interval = time_value / 1000
            elif unit == "Секунды":
                click_interval = time_value
            elif unit == "Минуты":
                click_interval = time_value * 60
        except ValueError:
            interval_entry.delete(0, tk.END)
            interval_entry.insert(0, "Ошибка!")

    ttk.Button(root, text="Установить интервал", command=set_interval).pack(pady=10)

    # Инструкция
    instructions = tk.Label(root, text="Нажмите F6 для включения/выключения кликов",
                            font=("Arial", 12), bg="#202020", fg="#FFFFFF", justify="center")
    instructions.pack(pady=10)

    # Запуск обработчика клавиш
    def start_listener():
        with Listener(on_press=toggle_clicker) as listener:
            listener.join()

    Thread(target=start_listener, daemon=True).start()

    root.mainloop()

# Запуск интерфейса
create_gui()
