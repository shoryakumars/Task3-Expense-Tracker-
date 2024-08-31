# Task3-Expense-Tracker-
Expense Tracker
 import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3
import matplotlib.pyplot as plt
from datetime import datetime


# Database file
DATABASE_FILE = 'expenses.db'


class ExpenseTracker:
    def _init_(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.root.geometry("600x500")

        self.conn = sqlite3.connect(DATABASE_FILE)
        self.create_table()

       
        self.amount_label = tk.Label(root, text="Amount")
        self.amount_label.pack()

        self.amount_entry = tk.Entry(root)
        self.amount_entry.pack()

        self.category_label = tk.Label(root, text="Category")
        self.category_label.pack()

        self.category_combo = ttk.Combobox(root, values=["Food", "Transport", "Utilities", "Entertainment", "Other"])
        self.category_combo.pack()

        self.desc_label = tk.Label(root, text="Description")
        self.desc_label.pack()

        self.desc_entry = tk.Entry(root)
        self.desc_entry.pack()

        self.date_label = tk.Label(root, text="Date (YYYY-MM-DD)")
        self.date_label.pack()

        self.date_entry = tk.Entry(root)
        self.date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))
        self.date_entry.pack()

        self.add_expense_button = tk.Button(root, text="Add Expense", command=self.add_expense)
        self.add_expense_button.pack()

        self.view_report_button = tk.Button(root, text="View Report", command=self.view_report)
        self.view_report_button.pack()

        self.plot_expenses_button = tk.Button(root, text="Plot Expenses", command=self.plot_expenses)
        self.plot_expenses_button.pack()

    def create_table(self):
        with self.conn:
            self.conn.execute('''
                CREATE TABLE IF NOT EXISTS expenses (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    amount REAL,
                    category TEXT,
                    description TEXT,
                    date TEXT
                )
            ''')

    def add_expense(self):
        try:
            amount = float(self.amount_entry.get().strip())
            category = self.category_combo.get().strip()
            description = self.desc_entry.get().strip()
            date = self.date_entry.get().strip()

            datetime.strptime(date, "%Y-%m-%d")  

            if not category:
                raise ValueError("Category is required")

            with self.conn:
                self.conn.execute('''
                    INSERT INTO expenses (amount, category, description, date) 
                    VALUES (?, ?, ?, ?)''', (amount, category, description, date))

            messagebox.showinfo("Success", "Expense added successfully!")
            self.clear_entries()
        except ValueError as e:
            messagebox.showerror("Error", f"Invalid input: {str(e)}")

    def view_report(self):
        with self.conn:
            cursor = self.conn.execute('''
                SELECT date, category, SUM(amount) 
                FROM expenses 
                GROUP BY date, category
                ORDER BY date DESC
            ''')

            report = ""
            for row in cursor:
                report += f"Date: {row[0]}, Category: {row[1]}, Amount: ${row[2]:.2f}\n"

        messagebox.showinfo("Expense Report", report if report else "No expenses recorded.")

    def plot_expenses(self):
        with self.conn:
            cursor = self.conn.execute('''
                SELECT category, SUM(amount) 
                FROM expenses 
                GROUP BY category
            ''')
            data = cursor.fetchall()

        if data:
            categories, amounts = zip(*data)

            plt.figure(figsize=(8, 6))
            plt.bar(categories, amounts, color='skyblue')
            plt.xlabel('Category')
            plt.ylabel('Total Amount ($)')
            plt.title('Expenses by Category')
            plt.tight_layout()
            plt.show()
        else:
            messagebox.showinfo("Plot Expenses", "No expenses to plot.")

    def clear_entries(self):
        self.amount_entry.delete(0, tk.END)
        self.category_combo.set("")
        self.desc_entry.delete(0, tk.END)
        self.date_entry.delete(0, tk.END)
        self.date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))


if _name_ == "_main_":
    root = tk.Tk()
    app = ExpenseTracker(root)
    root.mainloop()   
    
