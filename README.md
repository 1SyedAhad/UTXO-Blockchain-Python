💰 UTXO-Based Blockchain Simulation (Python)
This project provides a simplified simulation of a blockchain system using the Unspent Transaction Output (UTXO) model, similar to Bitcoin. It demonstrates how transactions consume existing UTXOs and create new ones, manage transaction fees, and track balances within a wallet. The simulation includes a detailed scenario of a user managing their cryptocurrency.

✨ Features
UTXO Management: Tracks and manages unspent transaction outputs as the fundamental unit of value.

Transaction Creation: Allows users to create transactions by selecting appropriate UTXOs as inputs and generating new UTXOs as outputs (including change and recipient outputs).

Transaction Fees: Supports the inclusion of transaction fees.

Balance Tracking: Calculates the current balance for any given owner based on their UTXOs.

Wallet Representation: Simulates a basic wallet by listing UTXOs belonging to a specific owner.

Simulation Steps: Provides a clear demonstration of how UTXOs are consumed and created across multiple transactions.

🚀 Installation
This project is a single Python file and does not require any special installation beyond a standard Python environment.

Clone the repository (or copy the code):

git clone https://github.com/1SyedAhad/UTXO-Blockchain-Python.git
cd UTXO-Blockchain-Python

(https://github.com/1SyedAhad/UTXO-Blockchain-Python)

Ensure you have Python installed:
This code is compatible with Python 3.x. If you don't have Python, download it from python.org.

💻 Usage
To run the simulation, simply execute the Python file. The if __name__ == "__main__": block contains a complete scenario demonstrating the UTXO model in action.

Example Scenario
This simulation walks through a series of transactions for "Jawad," showcasing how his UTXO wallet evolves.

import uuid

class UTXO:
    def __init__(self, txid, amount, owner):
        self.id = txid  # Unique UTXO ID (can be a transaction output ID + index)
        self.amount = amount
        self.owner = owner

class Transaction:
    def __init__(self, inputs, outputs, fee):
        self.id = str(uuid.uuid4()) # Unique Transaction ID
        self.inputs = inputs       # List of UTXO IDs consumed by this transaction
        self.outputs = outputs     # List of new UTXO objects created by this transaction
        self.fee = fee             # Transaction fee

class Blockchain:
    def __init__(self):
        self.utxos = {}   # Dictionary to store all active UTXOs (utxo_id: UTXO object)
        self.chain = []   # List of confirmed transactions (representing blocks in a simplified way)

    def add_utxo(self, utxo):
        """Adds a new UTXO to the global set of unspent transaction outputs."""
        self.utxos[utxo.id] = utxo

    def get_balance(self, owner):
        """Calculates the total balance for a given owner by summing their UTXO amounts."""
        return sum(utxo.amount for utxo in self.utxos.values() if utxo.owner == owner)

    def get_utxos(self, owner):
        """Retrieves all UTXOs currently owned by a specific owner."""
        return [utxo for utxo in self.utxos.values() if utxo.owner == owner]

    def make_transaction(self, sender, recipients, fee):
        """
        Creates and processes a new transaction.

        Args:
            sender (str): The owner initiating the transaction.
            recipients (list): A list of tuples, each containing (receiver_address, amount).
            fee (float): The transaction fee.

        Returns:
            Transaction: The newly created transaction object.

        Raises:
            ValueError: If the sender has insufficient balance.
        """
        print(f"\n--- Initiating transaction for {sender} ---")
        print(f"Current UTXOs for {sender}:")
        for utxo in self.get_utxos(sender):
            print(f"  {utxo.id}: {utxo.amount} BTC")
        print(f"Current balance for {sender}: {self.get_balance(sender)} BTC")

        # Get all UTXOs belonging to the sender and sort them to prioritize larger ones first (greedy selection)
        utxos = self.get_utxos(sender)
        utxos.sort(key=lambda x: x.amount, reverse=True)

        # Calculate the total amount needed for recipients and fee
        total_needed = sum(amount for _, amount in recipients) + fee
        selected_inputs = [] # UTXOs chosen to fund the transaction
        total_input_amount = 0

        # Select UTXOs until the total input amount is sufficient
        for utxo in utxos:
            selected_inputs.append(utxo)
            total_input_amount += utxo.amount
            if total_input_amount >= total_needed:
                break

        # Check for insufficient balance
        if total_input_amount < total_needed:
            raise ValueError(f"Insufficient balance for {sender}: needed {total_needed} BTC, found {total_input_amount} BTC")

        # Build inputs (IDs of consumed UTXOs)
        input_ids = [u.id for u in selected_inputs]

        # Build outputs (new UTXOs for recipients)
        outputs = [UTXO(str(uuid.uuid4()), amt, recv) for recv, amt in recipients]

        # Calculate and add change UTXO if any
        change = total_input_amount - total_needed
        if change > 0:
            change_utxo_id = str(uuid.uuid4())
            outputs.append(UTXO(change_utxo_id, change, sender))
            print(f"  Change UTXO created for {sender}: {change} BTC (ID: {change_utxo_id})")

        # Create the new transaction object
        tx = Transaction(input_ids, outputs, fee)
        self.chain.append(tx) # Add transaction to the blockchain (simplified to a list)

        # Update UTXO set: Remove consumed inputs and add new outputs
        for txid in input_ids:
            if txid in self.utxos:
                del self.utxos[txid]
            print(f"  Consumed UTXO: {txid}")

        for utxo in outputs:
            self.utxos[utxo.id] = utxo
            print(f"  Created new UTXO: {utxo.id} ({utxo.amount} BTC for {utxo.owner})")

        print(f"Transaction {tx.id} confirmed.")
        return tx

    def print_chain(self):
        """Prints a simplified representation of the transactions in the blockchain."""
        if not self.chain:
            print("Blockchain is empty (no transactions yet).")
            return
        for i, tx in enumerate(self.chain):
            print(f"\n--- Transaction {i+1} (ID: {tx.id}) ---")
            print(f"  Inputs: {', '.join(tx.inputs) if tx.inputs else 'None'}")
            print(f"  Fee: {tx.fee} BTC")
            for out in tx.outputs:
                print(f"  Output: {out.amount} BTC to {out.owner} (ID: {out.id})")

# ----------------------
# Simulation Execution
# ----------------------

bc = Blockchain()

print("--- Initializing Jawad's Wallet with UTXOs ---")
# Initial UTXOs for Jawad's crypto account wallet
utxo1 = UTXO("imran1_utxo", 2.0, "Jawad")
utxo2 = UTXO("farooq1_utxo", 0.4, "Jawad")
utxo3 = UTXO("mohsin1_utxo", 0.6, "Jawad")
utxo4 = UTXO("mujtaba1_utxo", 0.2, "Jawad")

bc.add_utxo(utxo1)
bc.add_utxo(utxo2)
bc.add_utxo(utxo3)
bc.add_utxo(utxo4)

print(f"Initial Balance for Jawad: {bc.get_balance('Jawad')} BTC")
print("Jawad's Initial UTXOs:")
for utxo in bc.get_utxos("Jawad"):
    print(f"  {utxo.id}: {utxo.amount} BTC")


# --- Amazon Transactions ---

print("\n\n===== Amazon Store Transactions =====")

# i) Jawad buys one Laptop (1.3 BTC + 0.1 BTC fee)
print("\n--- Transaction 1: Jawad buys Laptop (1.3 BTC) + 0.1 BTC fee ---")
# Total needed = 1.3 + 0.1 = 1.4 BTC
# Jawad's UTXOs available: 2.0, 0.6, 0.4, 0.2 (sorted by amount for selection)
# Selected inputs: "imran1_utxo" (2.0 BTC)
# Change: 2.0 - 1.4 = 0.6 BTC
try:
    bc.make_transaction("Jawad", [("Amazon", 1.3)], fee=0.1)
except ValueError as e:
    print(f"Error: {e}")

print(f"\nJawad's Balance After Laptop Purchase: {bc.get_balance('Jawad')} BTC")
print("Jawad's UTXOs After Laptop Purchase:")
for utxo in bc.get_utxos("Jawad"):
    print(f"  {utxo.id}: {utxo.amount} BTC")


# ii) Jawad buys one HardDisk (0.2 BTC + 0.1 BTC fee)
print("\n--- Transaction 2: Jawad buys HardDisk (0.2 BTC) + 0.1 BTC fee ---")
# Total needed = 0.2 + 0.1 = 0.3 BTC
# Jawad's UTXOs available: (new change UTXO from previous tx), 0.6, 0.4, 0.2
# Let's assume the change UTXO from above is "jawad_change_tx1" (0.6 BTC)
# Selected inputs: "jawad_change_tx1" (0.6 BTC)
# Change: 0.6 - 0.3 = 0.3 BTC
try:
    bc.make_transaction("Jawad", [("Amazon", 0.2)], fee=0.1)
except ValueError as e:
    print(f"Error: {e}")

print(f"\nJawad's Balance After HardDisk Purchase: {bc.get_balance('Jawad')} BTC")
print("Jawad's UTXOs After HardDisk Purchase:")
for utxo in bc.get_utxos("Jawad"):
    print(f"  {utxo.id}: {utxo.amount} BTC")

print("\n\n===== Yasir pays Jawad & Alibaba Store Transactions =====")

# Yasir pays 3.4 BTC to Jawad
print("\n--- Transaction 3 (Simulated external payment): Yasir pays 3.4 BTC to Jawad ---")
# This is simulated as a direct UTXO addition to Jawad's wallet
utxo_yasir = UTXO("yasir1_utxo", 3.4, "Jawad")
bc.add_utxo(utxo_yasir)
print(f"New UTXO added for Jawad from Yasir: {utxo_yasir.id}: {utxo_yasir.amount} BTC")
print(f"\nJawad's Balance After Yasir's Payment: {bc.get_balance('Jawad')} BTC")
print("Jawad's UTXOs After Yasir's Payment:")
for utxo in bc.get_utxos("Jawad"):
    print(f"  {utxo.id}: {utxo.amount} BTC")


# Alibaba: Bicycle
print("\n--- Transaction 4: Jawad buys Bicycle (2.1 BTC) + 0.2 BTC fee ---")
# Total needed = 2.1 + 0.2 = 2.3 BTC
# Jawad's current UTXOs (example IDs, actual will vary):
#   - From Laptop change: approx 0.3 BTC (from previous transaction)
#   - From HardDisk change: approx 0.3 BTC (from previous transaction)
#   - From Yasir: 3.4 BTC
# Selected inputs: "yasir1_utxo" (3.4 BTC) (assuming it's picked due to being largest)
# Change: 3.4 - 2.3 = 1.1 BTC
try:
    bc.make_transaction("Jawad", [("Alibaba", 2.1)], fee=0.2)
except ValueError as e:
    print(f"Error: {e}")

print(f"\nJawad's Balance After Bicycle Purchase: {bc.get_balance('Jawad')} BTC")
print("Jawad's UTXOs After Bicycle Purchase:")
for utxo in bc.get_utxos("Jawad"):
    print(f"  {utxo.id}: {utxo.amount} BTC")


# Alibaba: Printer
print("\n--- Transaction 5: Jawad buys Printer (0.3 BTC) + 0.1 BTC fee ---")
# Total needed = 0.3 + 0.1 = 0.4 BTC
# Jawad's current UTXOs (example IDs, actual will vary):
#   - From Bicycle change: approx 1.1 BTC (from previous transaction)
#   - Other remaining change UTXOs if not consumed: 0.3 BTC, 0.4 BTC (original), 0.2 BTC (original)
# Selected inputs: The smaller remaining UTXOs, or the change from Bicycle if needed.
# Let's assume a change UTXO (e.g., from HardDisk purchase) is 0.3 BTC and the original 0.4 BTC is available.
# The code will pick based on its internal sorting (largest first, or whatever is necessary).
# If a 0.6 BTC change UTXO exists, it will likely be picked.
try:
    bc.make_transaction("Jawad", [("Alibaba", 0.3)], fee=0.1)
except ValueError as e:
    print(f"Error: {e}")

print(f"\nJawad's Balance After Printer Purchase: {bc.get_balance('Jawad')} BTC")
print("Jawad's UTXOs After Printer Purchase:")
for utxo in bc.get_utxos("Jawad"):
    print(f"  {utxo.id}: {utxo.amount} BTC")


# Final blockchain state and Jawad's balance
print("\n\n========== Final Blockchain State (Confirmed Transactions) ==========")
bc.print_chain()

print("\n========== Jawad's Final Balance ==========")
print(f"Jawad's Total Balance: {bc.get_balance('Jawad')} BTC")

print("\n========== Jawad's Remaining UTXOs ==========")
jawad_utxos = bc.get_utxos("Jawad")
if jawad_utxos:
    for utxo in jawad_utxos:
        print(f"  {utxo.id}: {utxo.amount} BTC")
else:
    print("No UTXOs remaining for Jawad.")

To run this example:

python your_utxo_blockchain_script_name.py

(Replace your_utxo_blockchain_script_name.py with the actual name of your Python file.)

Intermediate Steps and Final State Explanation
The make_transaction method now includes print statements that show the selected inputs, created outputs, and change for each transaction, giving a clear view of the intermediate steps.

Initial State:

Jawad's initial UTXOs: Imran (2.0 BTC), Farooq (0.4 BTC), Mohsin (0.6 BTC), Mujtaba (0.2 BTC).

Total Balance: 2.0+0.4+0.6+0.2=3.2 BTC.

Amazon: Laptop Purchase (1.3 BTC + 0.1 BTC fee)

Total needed: 1.3+0.1=1.4 BTC.

Input(s) Used: The script will likely pick imran1_utxo (2.0 BTC) as it's the largest and sufficient.

Outputs Created:

New UTXO for "Amazon" with 1.3 BTC.

Change UTXO for "Jawad" with 2.0−1.4=0.6 BTC.

Remaining UTXOs for Jawad: Farooq (0.4 BTC), Mohsin (0.6 BTC), Mujtaba (0.2 BTC), and the new change UTXO (0.6 BTC).

Balance for Jawad: 0.4+0.6+0.2+0.6=1.8 BTC.

Amazon: HardDisk Purchase (0.2 BTC + 0.1 BTC fee)

Total needed: 0.2+0.1=0.3 BTC.

Input(s) Used: The script will likely pick the smallest available UTXO that is sufficient, or a combination. If the 0.6 BTC change UTXO is available, it might use that. Let's assume it uses the 0.6 BTC change UTXO from the laptop purchase.

Outputs Created:

New UTXO for "Amazon" with 0.2 BTC.

Change UTXO for "Jawad" with 0.6−0.3=0.3 BTC.

Remaining UTXOs for Jawad: Farooq (0.4 BTC), Mohsin (0.6 BTC), Mujtaba (0.2 BTC), and the new change UTXO (0.3 BTC).

Balance for Jawad: 0.4+0.6+0.2+0.3=1.5 BTC.

Yasir Pays Jawad (3.4 BTC)

Input(s) Used: None (this is a new incoming UTXO).

Outputs Created: New UTXO for "Jawad" from "Yasir" with 3.4 BTC.

Remaining UTXOs for Jawad: Farooq (0.4 BTC), Mohsin (0.6 BTC), Mujtaba (0.2 BTC), change UTXO (0.3 BTC from HardDisk), and the new UTXO from Yasir (3.4 BTC).

Balance for Jawad: 0.4+0.6+0.2+0.3+3.4=4.9 BTC.

Alibaba: Bicycle Purchase (2.1 BTC + 0.2 BTC fee)

Total needed: 2.1+0.2=2.3 BTC.

Input(s) Used: The script will likely pick the largest available UTXO, which is the 3.4 BTC UTXO from Yasir.

Outputs Created:

New UTXO for "Alibaba" with 2.1 BTC.

Change UTXO for "Jawad" with 3.4−2.3=1.1 BTC.

Remaining UTXOs for Jawad: Farooq (0.4 BTC), Mohsin (0.6 BTC), Mujtaba (0.2 BTC), change UTXO (0.3 BTC from HardDisk), and the new change UTXO (1.1 BTC from Bicycle).

Balance for Jawad: 0.4+0.6+0.2+0.3+1.1=2.6 BTC.

Alibaba: Printer Purchase (0.3 BTC + 0.1 BTC fee)

Total needed: 0.3+0.1=0.4 BTC.

Input(s) Used: The script will try to fulfill this with the smallest combination of available UTXOs. It might use the 0.4 BTC (Farooq's original) or the 0.6 BTC (Mohsin's original) or a combination of smaller ones (e.g., 0.3 BTC change + 0.2 BTC Mujtaba). Let's assume it picks the original farooq1_utxo (0.4 BTC).

Outputs Created:

New UTXO for "Alibaba" with 0.3 BTC.

No change if exactly 0.4 BTC UTXO is used. If a larger UTXO is used, there will be change.

Final Remaining UTXOs for Jawad: The UTXOs remaining after this transaction will be shown in the "Jawad's Remaining UTXOs" section of the output.

Final Balance for Jawad: This will be the sum of all Jawad's remaining UTXOs. The simulation's output will provide the exact final balance.

The output of the Python script will clearly show the exact UTXO IDs consumed and created, providing a precise step-by-step breakdown as requested.

🏗️ Code Structure
The project is structured around three main classes:

UTXO: Represents an Unspent Transaction Output. Each UTXO has a unique ID, an amount, and an owner. These are the fundamental units of value in the system.

Transaction: Represents a transaction. It includes a unique ID, a list of input UTXO IDs that are being consumed, a list of new UTXO objects being created as outputs, and the transaction fee.

Blockchain: Manages the overall state of the UTXO set and records transactions.

__init__(self): Initializes the global utxos dictionary and the chain (list of transactions).

add_utxo(self, utxo): Adds a new UTXO to the global pool of unspent outputs.

get_balance(self, owner): Calculates the current aggregate balance for a specific owner.

get_utxos(self, owner): Retrieves all currently unspent UTXOs belonging to a given owner.

make_transaction(self, sender, recipients, fee): The core function for creating and processing transactions. It selects necessary input UTXOs, calculates change, creates new output UTXOs, and updates the global UTXO set and the transaction chain. Includes print statements for step-by-step visibility.

print_chain(self): Displays a simplified view of all confirmed transactions in the blockchain.

⚠️ Error Handling
The make_transaction method includes a ValueError check to ensure that the sender has sufficient funds (total UTXO value) to cover the transaction amount plus the fee.

👋 Contributing
Contributions are welcome! If you have suggestions for improvements or find any issues, feel free to:

Fork the repository.

Create a new branch (git checkout -b feature/your-feature).

Make your changes.

Commit your changes (git commit -m 'Add your feature').

Push to the branch (git push origin feature/your-feature).

Open a Pull Request.

📄 License
This project is open-source and available under the MIT License.
