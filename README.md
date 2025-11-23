# EXPENSE-MANAGER-
import sqlite3
from tkinter import *
from tkinter import ttk, messagebox
from tkcalendar import DateEntry

# Database setup
conn = sqlite3.connect("expense_tracker.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS expenses(
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    date TEXT,
    payee TEXT,
    description TEXT,
    amount REAL,
    mode TEXT
)
""")

conn.commit()

# FUNCTIONS 

def add_expenses():
    date = date_entry.get()
    payee = payee_entry.get()
    desc = desc_entry.get()
    amount = amount_entry.get()
    mode = mode_entry.get()

    if not (date and payee and desc and amount and mode):
        messagebox.showerror("Error", "All fields are required!")
        return

    try:
        float(amount)
    except ValueError:
        messagebox.showerror("Error", "Amount must be a number!")
        return

    cursor.execute("INSERT INTO expenses(date, payee, description, amount, mode) VALUES(?,?,?,?,?)",
                   (date, payee, desc, amount, mode))
    conn.commit()

    load_expenses()
    clear_fields()
    messagebox.showinfo("Success", "Expense added successfully!")


def load_expenses():
    for row in table.get_children():
        table.delete(row)

    cursor.execute("SELECT * FROM expenses")
    rows = cursor.fetchall()

    for row in rows:
        table.insert("", END, values=row)


def clear_fields():
    payee_entry.delete(0, END)
    desc_entry.delete(0, END)
    amount_entry.delete(0, END)
    mode_entry.delete(0, END)


def delete_expense():
    selected = table.focus()
    if not selected:
        messagebox.showerror("Error", "Select an expense to delete")
        return

    values = table.item(selected, "values")
    exp_id = values[0]

    cursor.execute("DELETE FROM expenses WHERE id=?", (exp_id,))
    conn.commit()

    load_expenses()
    messagebox.showinfo("Deleted", "Expense removed successfully!")


def update_expense():
    selected = table.focus()
    if not selected:
        messagebox.showerror("Error", "Select an expense to update")
        return

    values = table.item(selected, "values")
    exp_id = values[0]

    date = date_entry.get()
    payee = payee_entry.get()
    desc = desc_entry.get()
    amount = amount_entry.get()
    mode = mode_entry.get()

    cursor.execute("""
        UPDATE expenses SET
        date=?, payee=?, description=?, amount=?, mode=? 
        WHERE id=?
    """, (date, payee, desc, amount, mode, exp_id))

    conn.commit()
    load_expenses()
    messagebox.showinfo("Updated", "Expense updated successfully!")


def load_selected_expense(event):
    selected = table.focus()
    if not selected:
        return

    values = table.item(selected, "values")

    date_entry.set_date(values[1])
    payee_entry.delete(0, END); payee_entry.insert(0, values[2])
    desc_entry.delete(0, END); desc_entry.insert(0, values[3])
    amount_entry.delete(0, END); amount_entry.insert(0, values[4])
    mode_entry.delete(0, END); mode_entry.insert(0, values[5])


# GUI 
root = Tk()
root.title("Expense Manager")
root.geometry("1200x600")
root.resizable(False, False)

title = Label(root, text="Expense Tracker System", font=("Arial", 22, "bold"))
title.pack(pady=10)

# Entry form
form_frame = Frame(root)
form_frame.pack(pady=10)

Label(form_frame, text="Date", font=("Arial", 12)).grid(row=0, column=0, padx=10, pady=5)
date_entry = DateEntry(form_frame, width=18, background="darkblue", foreground="white")
date_entry.grid(row=0, column=1)

Label(form_frame, text="Payee", font=("Arial", 12)).grid(row=1, column=0, padx=10, pady=5)
payee_entry = Entry(form_frame, width=20)
payee_entry.grid(row=1, column=1)

Label(form_frame, text="Description", font=("Arial", 12)).grid(row=2, column=0, padx=10, pady=5)
desc_entry = Entry(form_frame, width=20)
desc_entry.grid(row=2, column=1)

Label(form_frame, text="Amount", font=("Arial", 12)).grid(row=3, column=0, padx=10, pady=5)
amount_entry = Entry(form_frame, width=20)
amount_entry.grid(row=3, column=1)

Label(form_frame, text="Mode of Payment", font=("Arial", 12)).grid(row=4, column=0, padx=10, pady=5)
mode_entry = Entry(form_frame, width=20)
mode_entry.grid(row=4, column=1)

# Buttons
button_frame = Frame(root)
button_frame.pack(pady=10)

Button(button_frame, text="Add Expense", command=add_expenses, width=15).grid(row=0, column=0, padx=10)
Button(button_frame, text="Update", command=update_expense, width=15).grid(row=0, column=1, padx=10)
Button(button_frame, text="Delete", command=delete_expense, width=15).grid(row=0, column=2, padx=10)
Button(button_frame, text="Clear", command=clear_fields, width=15).grid(row=0, column=3, padx=10)

# Table
table_frame = Frame(root)
table_frame.pack(pady=20)

columns = ("ID", "Date", "Payee", "Description", "Amount", "Mode")

table = ttk.Treeview(table_frame, columns=columns, show="headings")
for col in columns:
    table.heading(col, text=col)
    table.column(col, width=150)

table.pack(side=LEFT)

scroll = Scrollbar(table_frame, orient=VERTICAL, command=table.yview)
scroll.pack(side=RIGHT, fill=Y)
table.configure(yscrollcommand=scroll.set)

table.bind("<<TreeviewSelect>>", load_selected_expense)

load_expenses()

root.mainloop()
cursor.execute("DROP TABLE IF EXISTS expenses")
conn.commit()

expense_tracker.db
