# Youngless
Burmalda
import tkinter as tk
from tkinter import messagebox, ttk
import random
import json
import os
from datetime import datetime


class RandomTaskGenerator:
    """
    GUI-приложение «Random Task Generator»
    Генерирует случайные задачи, сохраняет историю, поддерживает фильтрацию
    """
    
    def __init__(self):
        # Ссылка на GitHub репозиторий
        self.git_hub = "https://github.com/ваш_логин/random-task-generator"
        
        # Предопределённые задачи
        self.tasks = {
            "учёба": [
                "📚 Прочитать статью по Python",
                "📖 Выучить 10 новых слов",
                "✍️ Написать конспект по теме",
                "💻 Решить 3 задачи по программированию",
                "📝 Пройти онлайн-курс (1 урок)"
            ],
            "спорт": [
                "🏃 Сделать зарядку (15 мин)",
                "🧘 Позаниматься йогой",
                "💪 Отжаться 20 раз",
                "🚶 Пройти 5000 шагов",
                "🏊 Пойти в бассейн"
            ],
            "работа": [
                "💼 Ответить на письма",
                "📊 Составить отчёт",
                "📅 Запланировать неделю",
                "🤝 Созвониться с коллегой",
                "📁 Разобрать рабочие файлы"
            ]
        }
        
        # Список всех задач для истории
        self.all_tasks = []
        for category, task_list in self.tasks.items():
            for task in task_list:
                self.all_tasks.append({
                    "task": task,
                    "category": category,
                    "date": ""
                })
        
        # История сгенерированных задач
        self.history = []
        self.data_file = "task_history.json"
        self.current_filter = "все"
        
        # Загрузка истории из файла
        self.load_history()
        
        # Создание интерфейса
        self.create_window()
        self.create_widgets()
        
        # Обновление списка истории
        self.update_history_display()
        
        self.root.mainloop()
    
    def create_window(self):
        """Создание главного окна"""
        self.root = tk.Tk()
        self.root.title("🎲 Random Task Generator")
        self.root.geometry("700x600")
        self.root.resizable(True, True)
        self.root.configure(bg='#f0f0f0')
        
        # Центрирование окна
        self.center_window()
    
    def center_window(self):
        self.root.update_idletasks()
        width = 700
        height = 600
        x = (self.root.winfo_screenwidth() // 2) - (width // 2)
        y = (self.root.winfo_screenheight() // 2) - (height // 2)
        self.root.geometry(f'{width}x{height}+{x}+{y}')
    
    def create_widgets(self):
        """Создание всех виджетов"""
        
        # Заголовок
        title_frame = tk.Frame(self.root, bg='#2c3e50', height=70)
        title_frame.pack(fill='x')
        title_frame.pack_propagate(False)
        
        tk.Label(
            title_frame,
            text="🎲 RANDOM TASK GENERATOR 🎲",
            font=("Arial", 20, "bold"),
            bg='#2c3e50',
            fg='white'
        ).pack(expand=True)
        
        # Основной контейнер
        main_frame = tk.Frame(self.root, bg='#f0f0f0')
        main_frame.pack(fill='both', expand=True, padx=20, pady=15)
        
        # ========== ВЕРХНЯЯ ПАНЕЛЬ ==========
        top_frame = tk.LabelFrame(
            main_frame,
            text="🎯 Генерация задачи",
            font=("Arial", 12, "bold"),
            bg='#f0f0f0',
            fg='#2c3e50',
            padx=15,
            pady=10
        )
        top_frame.pack(fill='x', pady=(0, 15))
        
        # Вывод сгенерированной задачи
        self.task_label = tk.Label(
            top_frame,
            text="Нажмите кнопку, чтобы получить задачу!",
            font=("Arial", 14),
            bg='#ecf0f1',
            fg='#2c3e50',
            wraplength=550,
            justify='center',
            relief='groove',
            padx=20,
            pady=20
        )
        self.task_label.pack(fill='x', pady=(0, 10))
        
        # Кнопка генерации
        self.generate_button = tk.Button(
            top_frame,
            text="🎲 СГЕНЕРИРОВАТЬ ЗАДАЧУ 🎲",
            font=("Arial", 14, "bold"),
            bg='#27ae60',
            fg='white',
            activebackground='#229954',
            cursor='hand2',
            padx=20,
            pady=10,
            command=self.generate_task
        )
        self.generate_button.pack()
        
        # ========== ПАНЕЛЬ ДОБАВЛЕНИЯ ==========
        add_frame = tk.LabelFrame(
            main_frame,
            text="➕ Добавить свою задачу",
            font=("Arial", 10, "bold"),
            bg='#f0f0f0',
            fg='#2c3e50',
            padx=15,
            pady=10
        )
        add_frame.pack(fill='x', pady=(0, 15))
        
        entry_frame = tk.Frame(add_frame, bg='#f0f0f0')
        entry_frame.pack(fill='x')
        
        self.new_task_entry = tk.Entry(
            entry_frame,
            font=("Arial", 11),
            relief='solid',
            bd=1
        )
        self.new_task_entry.pack(side='left', fill='x', expand=True, padx=(0, 10))
        self.new_task_entry.bind('<Return>', lambda e: self.add_custom_task())
        
        # Выбор категории
        self.category_var = tk.StringVar(value="работа")
        category_menu = ttk.Combobox(
            entry_frame,
            textvariable=self.category_var,
            values=["учёба", "спорт", "работа"],
            state="readonly",
            width=12
        )
        category_menu.pack(side='left', padx=(0, 10))
        
        tk.Button(
            entry_frame,
            text="➕ Добавить",
            font=("Arial", 10, "bold"),
            bg='#3498db',
            fg='white',
            cursor='hand2',
            padx=15,
            command=self.add_custom_task
        ).pack(side='left')
        
        # ========== ПАНЕЛЬ ФИЛЬТРАЦИИ ==========
        filter_frame = tk.LabelFrame(
            main_frame,
            text="🔍 Фильтрация",
            font=("Arial", 10, "bold"),
            bg='#f0f0f0',
            fg='#2c3e50',
            padx=15,
            pady=10
        )
        filter_frame.pack(fill='x', pady=(0, 15))
        
        filter_buttons_frame = tk.Frame(filter_frame, bg='#f0f0f0')
        filter_buttons_frame.pack()
        
        filters = [("📋 Все", "все"), ("📚 Учёба", "учёба"), ("🏃 Спорт", "спорт"), ("💼 Работа", "работа")]
        
        for text, value in filters:
            tk.Button(
                filter_buttons_frame,
                text=text,
                font=("Arial", 10),
                bg='#95a5a6' if self.current_filter != value else '#3498db',
                fg='white',
                cursor='hand2',
                padx=15,
                pady=5,
                command=lambda v=value: self.set_filter(v)
            ).pack(side='left', padx=5)
        
        # ========== ПАНЕЛЬ ИСТОРИИ ==========
        history_frame = tk.LabelFrame(
            main_frame,
            text="📜 История задач",
            font=("Arial", 12, "bold"),
            bg='#f0f0f0',
            fg='#2c3e50',
            padx=10,
            pady=10
        )
        history_frame.pack(fill='both', expand=True)
        
        # Listbox с прокруткой
        listbox_frame = tk.Frame(history_frame)
        listbox_frame.pack(fill='both', expand=True)
        
        self.history_listbox = tk.Listbox(
            listbox_frame,
            font=("Arial", 10),
            bg='white',
            fg='#333',
            selectbackground='#3498db',
            height=8
        )
        self.history_listbox.pack(side='left', fill='both', expand=True)
        
        scrollbar = tk.Scrollbar(listbox_frame, orient='vertical', command=self.history_listbox.yview)
        scrollbar.pack(side='right', fill='y')
        self.history_listbox.config(yscrollcommand=scrollbar.set)
        
        # Кнопки управления историей
        history_buttons = tk.Frame(history_frame, bg='#f0f0f0')
        history_buttons.pack(fill='x', pady=(10, 0))
        
        tk.Button(
            history_buttons,
            text="🗑 Очистить историю",
            font=("Arial", 10),
            bg='#e74c3c',
            fg='white',
            cursor='hand2',
            padx=15,
            command=self.clear_history
        ).pack(side='left', padx=(0, 10))
        
        tk.Button(
            history_buttons,
            text="💾 Сохранить",
            font=("Arial", 10),
            bg='#27ae60',
            fg='white',
            cursor='hand2',
            padx=15,
            command=self.save_history
        ).pack(side='left')
        
        # ========== СТАТУСНАЯ СТРОКА ==========
        status_frame = tk.Frame(self.root, bg='#34495e', height=35)
        status_frame.pack(side='bottom', fill='x')
        
        self.status_label = tk.Label(
            status_frame,
            text="Готов к работе",
            font=("Arial", 9),
            bg='#34495e',
            fg='white',
            anchor='w'
        )
        self.status_label.pack(side='left', padx=15, pady=8, fill='x', expand=True)
        
        # Ссылка на GitHub
        github_label = tk.Label(
            status_frame,
            text=f"🐙 GitHub: {self.git_hub}",
            font=("Arial", 8),
            bg='#34495e',
            fg='#7f8c8d',
            cursor='hand2'
        )
        github_label.pack(side='right', padx=15)
        github_label.bind('<Button-1>', lambda e: self.open_github())
    
    def open_github(self):
        import webbrowser
        webbrowser.open(self.git_hub)
    
    def generate_task(self):
        """Генерирует случайную задачу"""
        if self.current_filter == "все":
            # Из всех категорий
            category = random.choice(list(self.tasks.keys()))
            task = random.choice(self.tasks[category])
        else:
            # Из выбранной категории
            category = self.current_filter
            if self.tasks[category]:
                task = random.choice(self.tasks[category])
            else:
                task = "Нет задач в этой категории!"
        
        # Добавление в историю
        history_entry = {
            "task": task,
            "category": category,
            "date": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "type": "generated"
        }
        self.history.insert(0, history_entry)
        
        # Отображение задачи
        self.task_label.config(text=f"🎯 {task}", fg='#27ae60')
        self.root.after(2000, lambda: self.task_label.config(fg='#2c3e50'))
        
        # Обновление отображения
        self.update_history_display()
        self.save_history()
        
        self.status_label.config(text=f"✅ Сгенерирована задача: {task[:50]}...", fg='#2ecc71')
        self.root.after(3000, lambda: self.status_label.config(text="Готов к работе", fg='white'))
    
    def add_custom_task(self):
        """Добавляет пользовательскую задачу"""
        task_text = self.new_task_entry.get().strip()
        category = self.category_var.get()
        
        # Проверка корректности ввода
        if not task_text:
            messagebox.showwarning("Ошибка", "Введите текст задачи!")
            self.status_label.config(text="❌ Ошибка: пустая задача", fg='#e74c3c')
            self.root.after(2000, lambda: self.status_label.config(text="Готов к работе", fg='white'))
            return
        
        # Добавление задачи
        self.tasks[category].append(task_text)
        
        # Добавление в историю
        history_entry = {
            "task": task_text,
            "category": category,
            "date": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "type": "custom"
        }
        self.history.insert(0, history_entry)
        
        # Очистка поля
        self.new_task_entry.delete(0, tk.END)
        
        # Обновление отображения
        self.update_history_display()
        self.save_history()
        self.save_tasks_to_file()
        
        self.status_label.config(text=f"✅ Добавлена задача: {task_text[:50]}...", fg='#2ecc71')
        self.root.after(3000, lambda: self.status_label.config(text="Готов к работе", fg='white'))
        
        messagebox.showinfo("Успех", f"Задача добавлена в категорию '{category}'!")
    
    def set_filter(self, filter_value):
        """Устанавливает фильтр для отображения истории"""
        self.current_filter = filter_value
        self.update_history_display()
        
        # Обновление цвета кнопок
        self.status_label.config(text=f"🔍 Фильтр: {filter_value}", fg='#f39c12')
        self.root.after(2000, lambda: self.status_label.config(text="Готов к работе", fg='white'))
    
    def update_history_display(self):
        """Обновляет отображение истории в Listbox"""
        self.history_listbox.delete(0, tk.END)
        
        filtered_history = self.history
        if self.current_filter != "все":
            filtered_history = [h for h in self.history if h["category"] == self.current_filter]
        
        for entry in filtered_history:
            display_text = f"[{entry['date']}] {entry['category']}: {entry['task']}"
            self.history_listbox.insert(tk.END, display_text)
    
    def clear_history(self):
        """Очищает историю"""
        if messagebox.askyesno("Подтверждение", "Очистить всю историю задач?"):
            self.history = []
            self.update_history_display()
            self.save_history()
            self.status_label.config(text="🗑 История очищена", fg='#e74c3c')
            self.root.after(2000, lambda: self.status_label.config(text="Готов к работе", fg='white'))
    
    def load_history(self):
        """Загружает историю из JSON файла"""
        if os.path.exists(self.data_file):
            try:
                with open(self.data_file, 'r', encoding='utf-8') as f:
                    self.history = json.load(f)
            except:
                self.history = []
    
    def save_history(self):
        """Сохраняет историю в JSON файл"""
        try:
            with open(self.data_file, 'w', encoding='utf-8') as f:
                json.dump(self.history, f, ensure_ascii=False, indent=2)
        except Exception as e:
            print(f"Ошибка сохранения: {e}")
    
    def save_tasks_to_file(self):
        """Сохраняет задачи в JSON файл"""
        try:
            with open("tasks.json", 'w', encoding='utf-8') as f:
                json.dump(self.tasks, f, ensure_ascii=False, indent=2)
        except Exception as e:
            print(f"Ошибка сохранения задач: {e}")


if __name__ == "__main__":
    app = RandomTaskGenerator()
