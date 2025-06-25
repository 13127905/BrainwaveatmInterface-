import tkinter as tk
from tkinter import messagebox
from PIL import Image, ImageTk
import threading
import os

# Enable or disable sound
try:
    from playsound import playsound
    SOUND_ENABLED = True
except ImportError:
    SOUND_ENABLED = False

def safe_play(sound_file):
    if SOUND_ENABLED and os.path.exists(sound_file):
        threading.Thread(target=lambda: playsound(sound_file)).start()

# Sample users
user_data = {
    '0000': {'name': 'Sanjith', 'balance': 10000.0},
    '1234': {'name': 'Ravi', 'balance': 8000.0},
    '1111': {'name': 'Anjali', 'balance': 15000.0},
    '4321': {'name': 'Alex', 'balance': 5000.0}
}

class ATMApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Brainwave ATM")
        self.root.attributes("-fullscreen", True)
        self.root.configure(bg="#003366")
        self.current_pin = ''
        self.current_user = None

        self.logo_path = r"C:\Users\Ravibharathi\Downloads\1b5a8b5a-5587-4790-9a0b-48cfbd02af27.jpg"
        self.card_insert_screen()

    def card_insert_screen(self):
        self.clear_screen()
        if os.path.exists(self.logo_path):
            logo_img = Image.open(self.logo_path).resize((600, 100))
            self.logo = ImageTk.PhotoImage(logo_img)
            tk.Label(self.root, image=self.logo, bg="#003366").pack(pady=(20, 0))

        tk.Label(self.root, text="Welcome to Brainwave ATM", font=('Helvetica', 24, 'bold'),
                 bg="#00bfff", fg="white").pack(fill="x", pady=10)

        tk.Button(self.root, text="Insert ATM Card", font=('Helvetica', 18), width=25, height=2,
                  bg="#4caf50", fg="white", command=self.insert_card).pack(pady=50)

    def insert_card(self):
        safe_play("card_insert.wav")
        self.root.after(1000, self.pin_entry_screen)

    def pin_entry_screen(self, next_screen=None):
        self.clear_screen()
        self.next_screen = next_screen
        tk.Label(self.root, text="Enter Your PIN", font=('Helvetica', 24, 'bold'),
                 bg="#00bfff", fg="white").pack(fill="x", pady=10)
        self.pin_display = tk.Label(self.root, text='', font=('Helvetica', 32, 'bold'),
                                    bg="#003366", fg="white")
        self.pin_display.pack(pady=20)

        btn_frame = tk.Frame(self.root, bg="#003366")
        btn_frame.pack()

        for i, num in enumerate("123456789"):
            tk.Button(btn_frame, text=num, font=('Helvetica', 18), width=6, height=2,
                      command=lambda n=num: self.add_pin_digit(n)).grid(row=i//3, column=i%3, padx=15, pady=10)

        tk.Button(btn_frame, text="Clear", font=('Helvetica', 18), width=6, height=2,
                  command=self.clear_pin).grid(row=3, column=0, padx=15, pady=10)
        tk.Button(btn_frame, text="0", font=('Helvetica', 18), width=6, height=2,
                  command=lambda: self.add_pin_digit("0")).grid(row=3, column=1, padx=15, pady=10)
        tk.Button(btn_frame, text="Enter", font=('Helvetica', 18), width=6, height=2,
                  command=self.check_pin).grid(row=3, column=2, padx=15, pady=10)

        tk.Button(self.root, text="Cancel", font=('Helvetica', 18), bg="#9e9e9e", fg="white",
                  command=self.card_insert_screen).pack(pady=20)

    def add_pin_digit(self, digit):
        safe_play("beep.wav")
        if len(self.current_pin) < 6:
            self.current_pin += digit
            self.pin_display.config(text='*' * len(self.current_pin))

    def clear_pin(self):
        self.current_pin = ''
        self.pin_display.config(text='')

    def check_pin(self):
        if self.current_pin in user_data:
            self.current_user = self.current_pin
            self.current_pin = ''
            if self.next_screen:
                self.next_screen()
            else:
                self.dashboard()
        else:
            messagebox.showerror("Error", "Invalid PIN")
            self.clear_pin()

    def dashboard(self):
        self.clear_screen()
        user_name = user_data[self.current_user]['name']
        if os.path.exists(self.logo_path):
            logo_img = Image.open(self.logo_path).resize((600, 100))
            self.logo = ImageTk.PhotoImage(logo_img)
            tk.Label(self.root, image=self.logo, bg="#003366").pack(pady=(20, 0))

        tk.Label(self.root, text=f"Welcome, {user_name}!", font=('Helvetica', 24, 'bold'),
                 bg="#00bfff", fg="white").pack(fill="x", pady=10)

        main_frame = tk.Frame(self.root, bg="#003366")
        main_frame.pack(expand=True, fill="both")

        left_frame = tk.Frame(main_frame, bg="#003366")
        left_frame.pack(side="left", expand=True)

        right_frame = tk.Frame(main_frame, bg="#003366")
        right_frame.pack(side="right", expand=True)

        btn_config = {"width": 25, "height": 3, "font": ('Helvetica', 20), "bg": "#1e90ff", "fg": "white", "padx": 10, "pady": 20}

        tk.Button(left_frame, text="Withdrawal", command=self.withdraw_screen, **btn_config).pack(pady=10)
        tk.Button(left_frame, text="Balance Inquiry", command=self.check_balance, **btn_config).pack(pady=10)
        tk.Button(left_frame, text="Transfer", command=self.transfer_screen, **btn_config).pack(pady=10)
        tk.Button(left_frame, text="Change PIN", command=self.change_pin_screen, **btn_config).pack(pady=10)

        tk.Button(right_frame, text="Payment", command=self.payment_screen, **btn_config).pack(pady=10)
        tk.Button(right_frame, text="Recent Transactions", command=self.recent_transactions, **btn_config).pack(pady=10)
        tk.Button(right_frame, text="Deposit", command=self.deposit_screen, **btn_config).pack(pady=10)
        tk.Button(right_frame, text="Fast Cash", command=self.fast_cash_screen, **btn_config).pack(pady=10)

    def withdraw_screen(self):
        self.clear_screen()
        tk.Label(self.root, text="Enter amount to withdraw", font=('Helvetica', 24), bg="#003366", fg="white").pack(pady=30)
        self.withdraw_entry = tk.Entry(self.root, font=('Helvetica', 20))
        self.withdraw_entry.pack(pady=10)
        tk.Button(self.root, text="Withdraw", font=('Helvetica', 18), command=self.withdraw).pack(pady=10)
        tk.Button(self.root, text="Back", font=('Helvetica', 18), command=self.dashboard).pack()

    def withdraw(self):
        try:
            amount = float(self.withdraw_entry.get())
            if amount <= 0 or amount > user_data[self.current_user]['balance']:
                raise ValueError
            user_data[self.current_user]['balance'] -= amount
            safe_play("loud_cash_dispensed.wav")  # louder sound
            messagebox.showinfo("Cash Dispensed", f"Collect ₹{amount:.2f}")
            self.dashboard()
        except ValueError:
            messagebox.showerror("Error", "Insufficient balance or invalid amount")

    def fast_cash_screen(self):
        self.clear_screen()
        tk.Label(self.root, text="Select Fast Cash Amount", font=('Helvetica', 24), bg="#003366", fg="white").pack(pady=30)
        btn_frame = tk.Frame(self.root, bg="#003366")
        btn_frame.pack(pady=20)

        for i, amt in enumerate([500, 1000, 2000, 3000, 5000]):
            tk.Button(btn_frame, text=f"₹{amt}", font=('Helvetica', 20), width=15, height=2,
                      command=lambda a=amt: self.fast_cash_withdraw(a)).grid(row=i//2, column=i%2, padx=20, pady=10)

        tk.Button(self.root, text="Back", font=('Helvetica', 18), command=self.dashboard).pack(pady=20)

    def fast_cash_withdraw(self, amount):
        if user_data[self.current_user]['balance'] >= amount:
            user_data[self.current_user]['balance'] -= amount
            safe_play("loud_cash_dispensed.wav")
            messagebox.showinfo("Cash Dispensed", f"Collect ₹{amount:.2f}")
            self.dashboard()
        else:
            messagebox.showerror("Error", "Insufficient balance")

    def check_balance(self):
        balance = user_data[self.current_user]['balance']
        messagebox.showinfo("Balance", f"Your balance is ₹{balance:.2f}")
        self.dashboard()

    def deposit_screen(self):
        self.clear_screen()
        tk.Label(self.root, text="Enter amount to deposit", font=('Helvetica', 24), bg="#003366", fg="white").pack(pady=30)
        self.deposit_entry = tk.Entry(self.root, font=('Helvetica', 20))
        self.deposit_entry.pack(pady=10)
        tk.Button(self.root, text="Deposit", font=('Helvetica', 18), command=self.deposit).pack(pady=10)
        tk.Button(self.root, text="Back", font=('Helvetica', 18), command=self.dashboard).pack()

    def deposit(self):
        try:
            amount = float(self.deposit_entry.get())
            if amount <= 0:
                raise ValueError
            user_data[self.current_user]['balance'] += amount
            messagebox.showinfo("Success", f"₹{amount:.2f} deposited")
            self.dashboard()
        except ValueError:
            messagebox.showerror("Error", "Enter a valid amount")

    def transfer_screen(self):
        self.clear_screen()
        tk.Label(self.root, text="Transfer Money", font=('Helvetica', 24), bg="#003366", fg="white").pack(pady=20)
        tk.Label(self.root, text="Recipient PIN:", bg="#003366", fg="white").pack()
        self.recipient_entry = tk.Entry(self.root)
        self.recipient_entry.pack()
        tk.Label(self.root, text="Amount:", bg="#003366", fg="white").pack()
        self.transfer_amount_entry = tk.Entry(self.root)
        self.transfer_amount_entry.pack()
        tk.Button(self.root, text="Transfer", command=self.transfer).pack(pady=10)
        tk.Button(self.root, text="Back", command=self.dashboard).pack()

    def transfer(self):
        recipient = self.recipient_entry.get()
        try:
            amount = float(self.transfer_amount_entry.get())
            if recipient == self.current_user or recipient not in user_data or amount <= 0:
                raise ValueError
            if amount > user_data[self.current_user]['balance']:
                raise ValueError("Insufficient balance")
            user_data[self.current_user]['balance'] -= amount
            user_data[recipient]['balance'] += amount
            messagebox.showinfo("Success", f"Transferred ₹{amount:.2f} to PIN {recipient}")
            self.dashboard()
        except Exception as e:
            messagebox.showerror("Error", str(e))

    def change_pin_screen(self):
        self.clear_screen()
        tk.Label(self.root, text="Change PIN", font=('Helvetica', 24), bg="#003366", fg="white").pack(pady=30)
        tk.Label(self.root, text="Old PIN:", bg="#003366", fg="white").pack()
        self.old_pin_entry = tk.Entry(self.root, show="*")
        self.old_pin_entry.pack()
        tk.Label(self.root, text="New PIN:", bg="#003366", fg="white").pack()
        self.new_pin_entry = tk.Entry(self.root, show="*")
        self.new_pin_entry.pack()
        tk.Label(self.root, text="Confirm New PIN:", bg="#003366", fg="white").pack()
        self.confirm_pin_entry = tk.Entry(self.root, show="*")
        self.confirm_pin_entry.pack()
        tk.Button(self.root, text="Update", command=self.change_pin).pack(pady=10)
        tk.Button(self.root, text="Back", command=self.dashboard).pack()

    def change_pin(self):
        old_pin = self.old_pin_entry.get()
        new_pin = self.new_pin_entry.get()
        confirm_pin = self.confirm_pin_entry.get()
        if old_pin != self.current_user:
            messagebox.showerror("Error", "Old PIN is incorrect")
        elif new_pin != confirm_pin:
            messagebox.showerror("Error", "PINs do not match")
        elif new_pin in user_data:
            messagebox.showerror("Error", "PIN already in use")
        else:
            user_data[new_pin] = user_data.pop(self.current_user)
            self.current_user = new_pin
            messagebox.showinfo("Success", "PIN changed")
            self.dashboard()

    def payment_screen(self):
        self.clear_screen()
        tk.Label(self.root, text="Select Payment", font=('Helvetica', 24), bg="#003366", fg="white").pack(pady=20)
        tk.Button(self.root, text="Electricity Bill", command=lambda: self.make_payment("Electricity")).pack(pady=5)
        tk.Button(self.root, text="Mobile Recharge", command=lambda: self.make_payment("Mobile")).pack(pady=10)
        tk.Button(self.root, text="Internet Bill", command=lambda: self.make_payment("Internet")).pack(pady=10)
        tk.Button(self.root, text="Back", command=self.dashboard).pack()

    def make_payment(self, service):
        messagebox.showinfo("Payment", f"{service} payment completed")
        self.dashboard()

    def recent_transactions(self):
        messagebox.showinfo("Transactions", "No recent transactions available.")
        self.dashboard()

    def clear_screen(self):
        for widget in self.root.winfo_children():
            widget.destroy()


if __name__ == "__main__":
    root = tk.Tk()
    app = ATMApp(root)
    root.mainloop()
