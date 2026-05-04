Вот пошаговая инструкция и готовый код для создания приложения **«Random Task Generator»** на Python с использованием библиотеки `tkinter` для графического интерфейса, `random` для генерации, `json` для сохранения истории и `git` для контроля версий.

## 1. Структура проекта

Создайте папку проекта, например, `RandomTaskGenerator`. Внутри неё будут следующие файлы:

*   `main.py` — основной код приложения.
*   `tasks.json` — файл для хранения истории (создаётся автоматически).
*   `README.md` — описание проекта.

## 2. Код приложения (main.py)

Этот код реализует все требования: генерацию, историю, фильтрацию, валидацию и сохранение.

```python
import tkinter as tk
from tkinter import ttk, messagebox
import random
import json
import os

# --- Настройки ---
HISTORY_FILE = "tasks.json"
DEFAULT_TASKS = [
    {"text": "Прочитать статью", "type": "учёба"},
    {"text": "Сделать зарядку", "type": "спорт"},
    {"text": "Написать отчёт", "type": "работа"},
    {"text": "Посмотреть обучающее видео", "type": "учёба"},
    {"text": "Разобрать почту", "type": "работа"},
    {"text": "Погулять на свежем воздухе", "type": "отдых"},
]

class TaskGeneratorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Генератор случайных задач")
        self.root.geometry("500x500")
        
        # Загрузка данных
        self.tasks = self.load_tasks()
        
        # --- Виджеты ---
        # Фильтр по типу
        self.filter_var = tk.StringVar(value="все")
        filter_frame = tk.Frame(root)
        filter_frame.pack(pady=5, fill=tk.X)
        
        tk.Label(filter_frame, text="Фильтр по типу:").pack(side=tk.LEFT)
        filter_options = ["все"] + sorted(set(task["type"] for task in self.tasks))
        ttk.Combobox(filter_frame, textvariable=self.filter_var, 
                    values=filter_options, state="readonly", width=15).pack(side=tk.LEFT, padx=5)
        self.filter_var.trace_add("write", self.update_history_list)

        # Кнопка генерации
        tk.Button(root, text="Сгенерировать задачу", 
                 bg="#4CAF50", fg="white", command=self.generate_task).pack(pady=10, fill=tk.X)

        # Поле текущей задачи
        self.current_task_label = tk.Label(root, text="Ваша задача появится здесь", 
                                          font=('Arial', 12, 'bold'), wraplength=400)
        self.current_task_label.pack(pady=10)

        # История задач
        history_frame = tk.Frame(root)
        history_frame.pack(pady=10, fill=tk.BOTH, expand=True)
        
        tk.Label(history_frame, text="История задач:", font=('Arial', 10, 'bold')).pack(anchor='w')
        self.history_listbox = tk.Listbox(history_frame, height=10)
        self.history_listbox.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        
        scrollbar = tk.Scrollbar(history_frame)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        self.history_listbox.config(yscrollcommand=scrollbar.set)
        scrollbar.config(command=self.history_listbox.yview)
        
        # Кнопки управления
        btn_frame = tk.Frame(root)
        btn_frame.pack(pady=5, fill=tk.X)
        
        # Добавление новой задачи
        add_frame = tk.Frame(btn_frame)
        add_frame.pack(fill=tk.X, pady=5)
        
        self.new_task_entry = tk.Entry(add_frame, width=30)
        self.new_task_entry.pack(side=tk.LEFT, expand=True)
        
        self.new_task_type = ttk.Combobox(add_frame, values=["учёба", "работа", "спорт", "отдых"], 
                                        state="readonly", width=10)
        self.new_task_type.set("работа")
        self.new_task_type.pack(side=tk.LEFT, padx=5)
        
        tk.Button(add_frame, text="Добавить в список", 
                 command=self.add_new_task).pack(side=tk.LEFT)

    def load_tasks(self):
        """Загрузка задач из JSON или создание файла с дефолтными задачами."""
        if os.path.exists(HISTORY_FILE):
            with open(HISTORY_FILE, 'r', encoding='utf-8') as f:
                try:
                    return json.load(f)
                except json.JSONDecodeError:
                    return DEFAULT_TASKS.copy()
        
        # Если файла нет, создаем его с дефолтными задачами
        with open(HISTORY_FILE, 'w', encoding='utf-8') as f:
            json.dump(DEFAULT_TASKS.copy(), f, ensure_ascii=False, indent=2)
            
        return DEFAULT_TASKS.copy()

    def save_tasks(self):
        """Сохранение списка задач в JSON."""
        with open(HISTORY_FILE, 'w', encoding='utf-8') as f:
            json.dump(self.tasks, f, ensure_ascii=False, indent=2)

    def update_history_list(self, *args):
        """Обновление виджета списка истории с учетом фильтра."""
        self.history_listbox.delete(0, tk.END)
        
        filter_type = self.filter_var.get()
        
        for task in reversed(self.tasks): # Показываем новые задачи сверху
            if filter_type == "все" or task["type"] == filter_type:
                self.history_listbox.insert(tk.END, f"{task['text']} ({task['type']})")

    def generate_task(self):
        """Генерация случайной задачи."""
        if not self.tasks:
            messagebox.showwarning("Предупреждение", "Список задач пуст! Добавьте новые задачи.")
            return

        selected_task = random.choice(self.tasks)
        
        # Отображаем задачу в главном лейбле
        self.current_task_label.config(
            text=f"Задача: {selected_task['text']}\nТип: {selected_task['type'].capitalize()}",
            bg="#f0f0f0", relief="solid"
        )

    def add_new_task(self):
        """Добавление новой задачи с валидацией."""
        task_text = self.new_task_entry.get().strip()
        
        if not task_text:
            messagebox.showerror("Ошибка", "Поле задачи не может быть пустым!")
            return

        task_type = self.new_task_type.get()
        
        new_task = {"text": task_text, "type": task_type}
        
        # Добавляем в начало списка (как самую новую) и сохраняем
        self.tasks.insert(0, new_task) 
        self.save_tasks()
        
        # Обновляем интерфейс
        self.update_history_list()
        
        # Очищаем поля ввода
        self.new_task_entry.delete(0, tk.END)
        
if __name__ == "__main__":
    root = tk.Tk()
    app = TaskGeneratorApp(root)
    app.update_history_list() # Первоначальная загрузка истории
    root.mainloop()
```

## 3. Использование Git и публикация

1.  **Инициализация репозитория**
    Откройте терминал в папке проекта и выполните команды:
    ```bash
    git init
    git add .
    git commit -m "Initial commit: Random Task Generator app"
    ```
2.  **Создание репозитория на GitHub/GitLab**
    *   Создайте новый (пустой) репозиторий на вашем сервисе (например, GitHub).
    *   Добавьте удаленный репозиторий:
    ```bash
    git remote add origin <ссылка_на_ваш_репозиторий>
    git push -u origin master # или main, в зависимости от настроек сервиса
    ```
3.  **Выложите проект.**
    Теперь ваш код доступен по ссылке на GitHub/GitLab.

## 4. README.md (Пример содержания)

Создайте файл `README.md` в корне проекта со следующим содержимым:

```
# Random Task Generator

**Автор:** [Ваше Имя Фамилия]
**Язык:** Python 3.x

## Описание

Простое графическое приложение (GUI) для генерации случайных задач из предопределенного или пользовательского списка. Приложение позволяет сохранять историю сгенерированных задач и добавлять новые.

## Возможности

*   Генерация случайной задачи по нажатию кнопки.
*   Встроенный список задач по категориям (Учёба, Работа, Спорт).
*   Добавление пользовательских задач с указанием типа.
*   История задач: все сгенерированные задачи сохраняются в файл.
*   Фильтрация истории по типу задач.
*   Сохранение истории в формате JSON.
*   Валидация ввода (проверка на пустую строку при добавлении).

## Технологии

*   **Python:** Основной язык.
*   **Tkinter:** Библиотека для создания графического интерфейса.
*   **Random:** Для случайного выбора задачи.
*   **JSON:** Для сохранения и загрузки истории.
*   **Git:** Система контроля версий.

## Как запустить

1.  Убедитесь, что у вас установлен Python 3.
2.  Скачайте или клонируйте этот репозиторий.
3.  Запустите файл `main.py`:
    ```bash
    python main.py
    ```
4.  Приложение откроется в новом окне.
```

## Примеры использования (тесты)

1.  **Тест генерации:**
    *   Нажмите кнопку «Сгенерировать задачу».
    *   В верхней части окна должна появиться новая задача (например: «Задача: Сделать зарядку. Тип: Спорт»).
2.  **Тест добавления:**
    *   Введите «Купить молоко» в поле ввода.
    *   Выберите тип «отдых».
    *   Нажмите «Добавить в список».
    *   Задача должна появиться в истории. Попробуйте отфильтровать по типу «отдых» — она должна быть видна.
3.  **Тест ошибки (валидация):**
    *   Оставьте поле ввода пустым и нажмите «Добавить в список».
    *   Должно появиться окно с ошибкой: «Поле задачи не может быть пустым!».
4.  **Тест сохранения:**
    *   Закройте приложение.
    *   Откройте файл `tasks.json`.
    *   В нем должны быть сохранены все добавленные вами задачи в формате JSON.
