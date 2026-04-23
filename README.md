# rtvbgvtb
import tkinter as tk
from tkinter import ttk, messagebox
import json
import os
from datetime import datetime

class WeatherDiary:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary")
        self.root.geometry("700x500")

        # Загрузка записей
        self.records = self.load_records()

        self.setup_ui()

    def setup_ui(self):
        # Фрейм для ввода данных
        input_frame = tk.LabelFrame(self.root, text="Добавить запись")
        input_frame.pack(pady=10, padx=10, fill="x")

        # Поле даты
        tk.Label(input_frame, text="Дата (ДД.ММ.ГГГГ):").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.date_entry = tk.Entry(input_frame, width=20)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)

        # Поле температуры
        tk.Label(input_frame, text="Температура (°C):").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.temp_entry = tk.Entry(input_frame, width=20)
        self.temp_entry.grid(row=1, column=1, padx=5, pady=5)

        # Поле описания погоды
        tk.Label(input_frame, text="Описание погоды:").grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.desc_entry = tk.Entry(input_frame, width=40)
        self.desc_entry.grid(row=2, column=1, padx=5, pady=5)

        # Чекбокс осадков
        self.precipitation_var = tk.BooleanVar()
        tk.Checkbutton(input_frame, text="Осадки", variable=self.precipitation_var).grid(
            row=3, column=0, padx=5, pady=5, sticky="w"
        )

        # Кнопка добавления записи
        tk.Button(input_frame, text="Добавить запись", command=self.add_record, bg="lightblue").grid(
            row=4, column=0, columnspan=2, pady=10
        )

        # Фрейм для фильтрации
        filter_frame = tk.LabelFrame(self.root, text="Фильтрация")
        filter_frame.pack(pady=10, padx=10, fill="x")

        # Фильтр по дате
        tk.Label(filter_frame, text="Фильтр по дате (ДД.ММ.ГГГГ):").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.filter_date_entry = tk.Entry(filter_frame, width=20)
        self.filter_date_entry.grid(row=0, column=1, padx=5, pady=5)

        # Фильтр по температуре
        tk.Label(filter_frame, text="Температура выше (°C):").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.filter_temp_entry = tk.Entry(filter_frame, width=20)
        self.filter_temp_entry.grid(row=1, column=1, padx=5, pady=5)

        # Кнопка применения фильтров
        tk.Button(filter_frame, text="Применить фильтры", command=self.apply_filters).grid(
            row=2, column=0, columnspan=2, pady=5
        )

        # Таблица записей
        columns = ("ID", "Дата", "Температура", "Описание", "Осадки")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings", height=12)

        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=100)

        self.tree.pack(pady=10, padx=10, fill="both", expand=True)

        # Прокрутка для таблицы
        scrollbar = ttk.Scrollbar(self.root, orient=tk.VERTICAL, command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        self.update_display()

    def validate_input(self):
        """Проверка корректности ввода данных"""
        # Проверка даты
        try:
            date_obj = datetime.strptime(self.date_entry.get(), "%d.%m.%Y")
            date_str = date_obj.strftime("%d.%m.%Y")
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты! Используйте ДД.ММ.ГГГГ")
            return None

        # Проверка температуры
        try:
            temperature = float(self.temp_entry.get())
        except ValueError:
            messagebox.showerror("Ошибка", "Температура должна быть числом!")
            return None

        # Проверка описания
        description = self.desc_entry.get().strip()
        if not description:
            messagebox.showerror("Ошибка", "Описание погоды не может быть пустым!")
            return None

        return date_str, temperature, description

    def add_record(self):
        validation_result = self.validate_input()
        if validation_result is None:
            return

        date_str, temperature, description = validation_result
        precipitation = "Да" if self.precipitation_var.get() else "Нет"

        record = {
            "date": date_str,
            "temperature": temperature,
            "description": description,
            "precipitation": precipitation
        }

        self.records.append(record)
        self.save_records()
        self.update_display()

        # Очистка полей ввода
        self.date_entry.delete(0, tk.END)
        self.temp_entry.delete(0, tk.END)
        self.desc_entry.delete(0, tk.END)
        self.precipitation_var.set(False)

    def update_display(self, filtered_records=None):
        """Обновление отображения записей в таблице"""
        for item in self.tree.get_children():
            self.tree.delete(item)

        records_to_show = filtered_records if filtered_records is not None else self.records

        for i, record in enumerate(records_to_show, 1):
            self.tree.insert("", tk.END, values=(
                i,
                record["date"],
                f"{record['temperature']}°C",
                record["description"],
                record["precipitation"]
            ))

    def apply_filters(self):
        """Применение фильтров к записям"""
        filtered_records = self.records.copy()

        # Фильтр по дате
        filter_date = self.filter_date_entry.get().strip()
        if filter_date:
            try:
                datetime.strptime(filter_date, "%d.%m.%Y")  # Проверка формата
                filtered_records = [r for r in filtered_records if r["date"] == filter_date]
            except ValueError:
                messagebox.showerror("Ошибка", "Неверный формат даты для фильтра!")
                return

        # Фильтр по температуре
        filter_temp = self.filter_temp_entry.get().strip()
        if filter_temp:
            try:
                temp_threshold = float(filter_temp)
                filtered_records = [r for r in filtered_records if r["temperature"] > temp_threshold]
            except ValueError:
                messagebox.showerror("Ошибка", "Температура для фильтра должна быть числом!")
                return

        self.update_display(filtered_records)

    def save_records(self):
        """Сохранение записей в JSON-файл"""
        with open("weather_records.json", "w", encoding="utf-8") as f:
            json.dump(self.records, f, ensure_ascii=False, indent=2)

    def load_records(self):
        """Загрузка записей из JSON-файла"""
        if os.path.exists("weather_records.json"):
            with open("weather_records.json
