import string
import random
from tkinter import *
from tkinter import messagebox
import sqlite3

# Initialize the database
def init_db():
    with sqlite3.connect("users.db") as db:
        cursor = db.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS users(
                Username TEXT NOT NULL UNIQUE, 
                GeneratedPassword TEXT NOT NULL
            );
        """)
        db.commit()

class PasswordGeneratorApp:
    def __init__(self, master: Tk):
        self.master = master
        self.master.title('Password Generator')
        self.master.geometry('660x500')
        self.master.resizable(False, False)

        self.username = StringVar()
        self.passwordlen = IntVar()
        self.generatedpassword = StringVar()

        self.create_widgets()

    def create_widgets(self):
        Label(self.master, text="Password Generator", anchor=N, fg='darkblue', 
              font='arial 20 bold underline').grid(row=0, column=1, pady=(10, 20))

        Label(self.master, text="Enter user name: ", font='times 15').grid(row=1, column=0, pady=5)
        Entry(self.master, textvariable=self.username, font='times 15', bd=6, 
              relief='ridge').grid(row=1, column=1, pady=5)

        Label(self.master, text="Enter password length: ", font='times 15').grid(row=2, column=0, pady=5)
        Entry(self.master, textvariable=self.passwordlen, font='times 15', 
              bd=6, relief='ridge').grid(row=2, column=1, pady=5)

        Label(self.master, text="Generated password: ", font='times 15').grid(row=3, column=0, pady=5)
        Entry(self.master, textvariable=self.generatedpassword, font='times 15', bd=6, 
              relief='ridge', fg='darkgreen', state='readonly').grid(row=3, column=1, pady=5)

        Button(self.master, text="GENERATE PASSWORD", bd=3, relief='solid', padx=1, pady=1, 
               font='forte 15 bold', fg='white', bg='darkblue', 
               command=self.generate_pass).grid(row=4, column=1, pady=10)

        Button(self.master, text="ACCEPT", bd=3, relief='solid', padx=1, pady=1, 
               font='forte 15', fg='darkblue', bg='white', 
               command=self.accept_fields).grid(row=5, column=1, pady=5)

        Button(self.master, text="RESET", bd=3, relief='solid', padx=1, pady=1, 
               font='forte 15', fg='darkblue', bg='white', 
               command=self.reset_fields).grid(row=6, column=1, pady=5)

    def generate_pass(self):
        name = self.username.get()
        leng = self.passwordlen.get()

        if not name:
            messagebox.showerror("Error", "Name cannot be empty")
            return

        if not name.isalpha():
            messagebox.showerror("Error", "Name must be a string")
            self.reset_fields()
            return

        try:
            length = int(leng)
            if length < 6:
                messagebox.showerror("Error", "Password must be at least 6 characters long")
                return
        except ValueError:
            messagebox.showerror("Error", "Password length must be a number")
            return

        chars = string.ascii_letters + string.digits + "@#%&()?!"
        password = ''.join(random.sample(chars, length))
        self.generatedpassword.set(password)

    def accept_fields(self):
        username = self.username.get()
        generated_password = self.generatedpassword.get()

        if not username or not generated_password:
            messagebox.showerror("Error", "Both username and password must be generated")
            return

        with sqlite3.connect("users.db") as db:
            cursor = db.cursor()
            cursor.execute("SELECT * FROM users WHERE Username = ?", (username,))
            if cursor.fetchone():
                messagebox.showerror("Error", "This username already exists! Please use another username.")
            else:
                cursor.execute("INSERT INTO users (Username, GeneratedPassword) VALUES (?, ?)", 
                               (username, generated_password))
                db.commit()
                messagebox.showinfo("Success", "Password generated successfully")

    def reset_fields(self):
        self.username.set("")
        self.passwordlen.set(0)
        self.generatedpassword.set("")

if __name__ == '__main__':
    init_db()
    root = Tk()
    app = PasswordGeneratorApp(root)
    root.mainloop()
