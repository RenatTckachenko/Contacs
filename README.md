import sqlite3
from tkinter import *
from tkinter import messagebox
from tkinter.ttk import Treeview


# Создание базы данных и таблицы сотрудников
conn = sqlite3.connect('employees.db')
cursor = conn.cursor()
cursor.execute('''CREATE TABLE IF NOT EXISTS employees (id INTEGER PRIMARY KEY AUTOINCREMENT, fio TEXT, phone TEXT, email TEXT, salary REAL)''')
conn.commit()


# Функция для добавления сотрудника
def add_employee(fio, phone, email, salary):
    conn = sqlite3.connect('employees.db')
    cursor = conn.cursor()
    cursor.execute('''INSERT INTO employees (fio, phone, email, salary) VALUES (?, ?, ?, ?)''', (fio, phone, email, salary))
    conn.commit()
    messagebox.showinfo('Успех', 'Сотрудник добавлен успешно.')


# Функция для изменения информации о сотруднике
def update_employee(id, fio, phone, email, salary):
    conn = sqlite3.connect('employees.db')
    cursor = conn.cursor()
    cursor.execute('''UPDATE employees SET fio=?, phone=?, email=?, salary=? WHERE id=?''', (fio, phone, email, salary, id))
    conn.commit()
    messagebox.showinfo('Успех', 'Информация о сотруднике обновлена успешно.')


# Функция для удаления сотрудника
def delete_employee(id):
    conn = sqlite3.connect('employees.db')
    cursor = conn.cursor()
    cursor.execute('''DELETE FROM employees WHERE id=?''', (id,))
    conn.commit()
    messagebox.showinfo('Успех', 'Сотрудник удален успешно.')


# Функция для поиска сотрудника по ФИО
def search_employee(fio):
    conn = sqlite3.connect('employees.db')
    cursor = conn.cursor()
    cursor.execute('''SELECT * FROM employees WHERE fio=?''', (fio,))
    result = cursor.fetchall()
    if result:
        for row in result:
            tree.insert('', END, values=row)
    else:
        messagebox.showinfo('Ошибка', 'Сотрудник не найден.')


# Создание графического интерфейса с использованием библиотеки tkinter
root = Tk()
root.title('Список сотрудников компании')

# Создание виджета Treeview для вывода записей из БД
tree = Treeview(root, columns=('ID', 'ФИО', 'Телефон', 'Email', 'Зарплата'), show='headings')
tree.heading('ID', text='ID')
tree.column('ID', width=30)
tree.heading('ФИО', text='ФИО')
tree.column('ФИО', width=150)
tree.heading('Телефон', text='Телефон')
tree.column('Телефон', width=100)
tree.heading('Email', text='Email')
tree.column('Email', width=150)
tree.heading('Зарплата', text='Зарплата')
tree.column('Зарплата', width=70)
tree.pack(side=TOP, fill=BOTH, expand=1)

# Создание кнопок и полей ввода для операций с сотрудниками
frame = Frame(root)
frame.pack(side=TOP, pady=10)

label_fio = Label(frame, text='ФИО')
label_fio.grid(row=0, column=0)
entry_fio = Entry(frame)
entry_fio.grid(row=0, column=1)

label_phone = Label(frame, text='Телефон')
label_phone.grid(row=0, column=2)
entry_phone = Entry(frame)
entry_phone.grid(row=0, column=3)

label_email = Label(frame, text='Email')
label_email.grid(row=1, column=0)
entry_email = Entry(frame)
entry_email.grid(row=1, column=1)

label_salary = Label(frame, text='Зарплата')
label_salary.grid(row=1, column=2)
entry_salary = Entry(frame)
entry_salary.grid(row=1, column=3)


# Функция для обновления списка сотрудников
def update_employee_list():
    tree.delete(*tree.get_children())
    conn = sqlite3.connect('employees.db')
    cursor = conn.cursor()
    cursor.execute('''SELECT * FROM employees''')
    result = cursor.fetchall()
    for row in result:
        tree.insert('', END, values=row)


# Функция для обработки кнопки добавления сотрудника
def add_employee_button():
    fio = entry_fio.get()
    phone = entry_phone.get()
    email = entry_email.get()
    salary = entry_salary.get()
    add_employee(fio, phone, email, salary)
    update_employee_list()
    entry_fio.delete(0, 'end')
    entry_phone.delete(0, 'end')
    entry_email.delete(0, 'end')
    entry_salary.delete(0, 'end')


# Функция для обработки кнопки изменения сотрудника
def update_employee_button():
    selected_item = tree.focus()
    values = tree.item(selected_item, 'values')
    if not values:
        messagebox.showinfo('Ошибка', 'Выберите сотрудника из списка.')
        return
    id = values[0]
    fio = entry_fio.get()
    phone = entry_phone.get()
    email = entry_email.get()
    salary = entry_salary.get()
    update_employee(id, fio, phone, email, salary)
    update_employee_list()
    entry_fio.delete(0, 'end')
    entry_phone.delete(0, 'end')
    entry_email.delete(0, 'end')
    entry_salary.delete(0, 'end')


# Функция для обработки кнопки удаления сотрудника
def delete_employee_button():
    selected_item = tree.focus()
    values = tree.item(selected_item, 'values')
    if not values:
        messagebox.showinfo('Ошибка', 'Выберите сотрудника из списка.')
        return
    id = values[0]
    delete_employee(id)
    update_employee_list()
    entry_fio.delete(0, 'end')
    entry_phone.delete(0, 'end')
    entry_email.delete(0, 'end')
    entry_salary.delete(0, 'end')


# Функция для обработки кнопки поиска сотрудника
def search_employee_button():
    fio = entry_fio.get()
    if len(fio) > 0:
        tree.delete(*tree.get_children())
        search_employee(fio)
    else:
        messagebox.showinfo('Ошибка', 'Введите ФИО для поиска.')


# Кнопки операций с сотрудниками
button_add = Button(frame, text='Добавить', command=add_employee_button)
button_add.grid(row=2, column=0)

button_update = Button(frame, text='Изменить', command=update_employee_button)
button_update.grid(row=2, column=1)


button_delete = Button(frame, text='Удалить', command=delete_employee_button)
button_delete.grid(row=2, column=2)

button_search = Button(frame, text='Поиск', command=search_employee_button)
button_search.grid(row=2, column=3)

# Обновление списка сотрудников при запуске приложения
update_employee_list()

root.mainloop()
