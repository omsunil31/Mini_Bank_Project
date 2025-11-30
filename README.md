# ------------------ MINI BANK ------------------

class Bank_Account:
    # Class variables (shared by all accounts)
    accounts_list = []   # Stores all account objects created
    used_account_numbers = set()  # Keeps track of account numbers to avoid duplicates

    def __init__(self, name, city, balance=0):
        """
        Constructor method: runs automatically when a new Bank_Account object is created.
        """
        self.Customer_Name = name           # Save customer's name
        self.Customer_City = city           # Save customer's city
        self.Customer_Balance = balance     # Save customer's balance
        self.Customer_Account = self.generate_account_number()  # Generate unique account number
        self.transactions = []              # Empty list to store transaction history
        self.log_transaction("Account Opened", balance)  # Record the opening transaction

    @classmethod
    def generate_account_number(cls):
        """
        Generate a unique 10-digit account number.
        Uses random numbers and ensures no duplicates.
        """
        while True:  # Keep trying until a unique number is found
            acc_num = random.randint(10**9, 10**10 - 1)  # Random 10-digit number
            if acc_num not in cls.used_account_numbers:  # Check uniqueness
                cls.used_account_numbers.add(acc_num)    # Reserve this number
                return acc_num                           # Return it

    @classmethod
    def open_account(cls, name, city, balance=1000):
        """
        Open a new account with minimum deposit of 1000 INR.
        Prevents duplicate accounts (same name + city).
        """
        if balance < 1000:
            print("⚠️ Minimum opening balance is INR 1000. Please deposit at least 1000.")
            return None

        # Check if account already exists for same name + city
        for acc in cls.accounts_list:
            if acc.Customer_Name == name and acc.Customer_City == city:
                print("❌ Account already exists for this Name and City.")
                return None

        # Create new account object
        new_acc = Bank_Account(name, city, balance)
        cls.accounts_list.append(new_acc)  # Add to shared list

        # Print confirmation summary
        print(f"\n✅ Account opened successfully!")
        print("----- Account Information -----")
        print(f"Name       : {new_acc.Customer_Name}")
        print(f"City       : {new_acc.Customer_City}")
        print(f"Account No : {new_acc.Customer_Account}")
        print(f"Balance    : INR {new_acc.Customer_Balance}")
        print("-------------------------------")

        return new_acc

    def log_transaction(self, action, amount=0):
        """
        Record every transaction with timestamp, action, amount, and balance.
        """
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")  # Current time
        self.transactions.append({
            "time": timestamp,
            "action": action,
            "amount": amount,
            "balance": self.Customer_Balance
        })

    def deposit(self, amount):
        """
        Deposit money into account.
        """
        self.Customer_Balance += amount
        print(f"💰 Deposited INR {amount}. Updated Balance: INR {self.Customer_Balance}")
        self.log_transaction("Deposit", amount)

    def withdraw(self, amount):
        """
        Withdraw money if balance is sufficient.
        """
        if amount <= self.Customer_Balance:
            self.Customer_Balance -= amount
            print(f"💸 Withdrawn INR {amount}. Updated Balance: INR {self.Customer_Balance}")
            self.log_transaction("Withdrawal", amount)
        else:
            print("⚠️ Withdrawal failed: Insufficient Balance.")
            self.log_transaction("Failed Withdrawal", amount)

    def balance_enquiry(self):
        """
        Show current balance.
        """
        print(f"📊 Account No: {self.Customer_Account}, Balance: INR {self.Customer_Balance}")
        self.log_transaction("Balance Enquiry")

    def close_account(self):
        """
        Close account and remove from system.
        """
        print(f"🔒 Account {self.Customer_Account} closed. Balance INR {self.Customer_Balance} paid by cheque.")
        self.log_transaction("Account Closed", self.Customer_Balance)
        Bank_Account.accounts_list.remove(self)          # Remove from list
        Bank_Account.used_account_numbers.remove(self.Customer_Account)  # Free account number

    def show_transactions(self):
        """
        Display all transactions for this account.
        """
        print(f"\n📜 Transaction History for Account {self.Customer_Account}:")
        for txn in self.transactions:
            print(f"{txn['time']} | {txn['action']} | Amount: {txn['amount']} | Balance: {txn['balance']}")
        print("============================================")

    @classmethod
    def display_all_accounts(cls):
        """
        Show all accounts currently open in the system.
        """
        if not cls.accounts_list:
            print("📂 No active accounts found.")
        else:
            print("\n===== Existing Accounts =====")
            for acc in cls.accounts_list:
                print(f"Name: {acc.Customer_Name}, City: {acc.Customer_City}, "
                      f"Account No: {acc.Customer_Account}, Balance: INR {acc.Customer_Balance}")
            print("=============================")


# ------------------ HELPER FUNCTION ------------------

def find_account(acc_no):
    """
    Search for an account by account number.
    Returns account object if found, else None.
    """
    for acc in Bank_Account.accounts_list:
        if acc.Customer_Account == acc_no:
            return acc
    return None


# ------------------ MENU PROGRAM ------------------

def menu():
    """
    Interactive menu-driven program for banking operations.
    Keeps running until user chooses Exit.
    """
    while True:
        print("\n===== BANK MENU =====")
        print("1. Open Account")
        print("2. Deposit")
        print("3. Withdraw")
        print("4. Balance Enquiry")
        print("5. Close Account")
        print("6. Display All Accounts")
        print("7. Show Transaction History")
        print("8. Exit")

        choice = input("Enter your choice (1-8): ").strip()

        if choice == "1":
            # Account opening workflow
            print("\n--- Account Opening ---")
            name = input("Customer Name: ").strip()
            city = input("Customer City: ").strip()

            # Keep prompting until minimum deposit of 1000 is entered
            while True:
                try:
                    balance = float(input("Opening Balance (minimum 1000): ").strip())
                    if balance >= 1000:
                        break
                    else:
                        print("⚠️ Minimum opening balance is INR 1000. Please try again.")
                except ValueError:
                    print("⚠️ Invalid amount. Please enter a numeric value.")

            # Confirmation summary
            print("\n--- Confirm Account Details ---")
            print(f"Name: {name}")
            print(f"City: {city}")
            print(f"Initial Deposit: INR {balance}")
            confirm = input("Confirm account opening? (Y/N): ").strip().lower()

            if confirm == "y":
                Bank_Account.open_account(name, city, balance)
            else:
                print("❌ Account opening cancelled.")

        elif choice == "2":
            # Deposit workflow
            acc_no = int(input("Enter Account Number: ").strip())
            acc = find_account(acc_no)
            if acc:
                amount = float(input("Enter Deposit Amount: ").strip())
                acc.deposit(amount)
            else:
                print("❌ Account not found.")

        elif choice == "3":
            # Withdrawal workflow
            acc_no = int(input("Enter Account Number: ").strip())
            acc = find_account(acc_no)
            if acc:
                amount = float(input("Enter Withdrawal Amount: ").strip())
                acc.withdraw(amount)
            else:
                print("❌ Account not found.")

        elif choice == "4":
            # Balance enquiry workflow
            acc_no = int(input("Enter Account Number: ").strip())
            acc = find_account(acc_no)
            if acc:
                acc.balance_enquiry()
            else:
                print("❌ Account not found.")

        elif choice == "5":
            # Close account workflow
            acc_no = int(input("Enter Account Number: ").strip())
            acc = find_account(acc_no)
            if acc:
                acc.close_account()
            else:
                print("❌ Account not found.")

        elif choice == "6":
            # Display all accounts
            Bank_Account.display_all_accounts()

        elif choice == "7":
            # Show transaction history
            acc_no = int(input("Enter Account Number: ").strip())
            acc = find_account(acc_no)
            if acc:
                acc.show_transactions()
            else:
                print("❌ Account not found.")

        elif choice == "8":
            # Exit program
            print("👋 Thank you for banking with us!")
            break

        else:
            print("⚠️ Invalid choice. Please try again.")


# ------------------ RUN THE PROGRAM ------------------
menu()
