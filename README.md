# GCOMC Wallet API Documentation

## Base URL

```plaintext
https://neoncorp.eu.org:8853
```

## Endpoints

### 1. Create a New Wallet

**Endpoint:** `/new_wallet`

**Method:** `POST`

**Description:** Creates a new wallet with a given passphrase.

**Request Body:**

```json
{
    "passphrase": "string"
}
```

**Response:**

```json
{
    "wallet_hash": "string"
}
```

### 2. Restore Wallet

**Endpoint:** `/restore_wallet`

**Method:** `POST`

**Description:** Restores an existing wallet using its hash and passphrase.

**Request Body:**

```json
{
    "wallet_hash": "string",
    "passphrase": "string"
}
```

**Response (success):**

```json
{
    "balance": "number"
}
```

**Response (error):**

```json
{
    "error": "string"
}
```

### 3. Send Funds

**Endpoint:** `/send`

**Method:** `POST`

**Description:** Sends funds from one wallet to another.

**Request Body:**

```json
{
    "sender": "string",
    "receiver": "string",
    "amount": "number"
}
```

**Response (success):**

```json
{
    "transaction_id": "string",
    "status": "string"
}
```

**Response (error):**

```json
{
    "error": "string"
}
```

### 4. Show Transactions

**Endpoint:** `/transactions`

**Method:** `POST`

**Description:** Retrieves a list of transactions for a given wallet.

**Request Body:**

```json
{
    "wallet_hash": "string"
}
```

**Response:**

```json
{
    "transactions": [
        {
            "transaction_id": "string",
            "sender": "string",
            "receiver": "string",
            "amount": "number",
            "timestamp": "string"
        }
        // ... more transactions
    ]
}
```

## Python Implementation

### 1. Save Wallet Hash

Saves the wallet hash to a local file.

```python
import os

LOGIN_PATH = os.path.expanduser('~/.neon/login')

def save_wallet_hash(wallet_hash):
    os.makedirs(os.path.dirname(LOGIN_PATH), exist_ok=True)
    with open(LOGIN_PATH, 'w') as f:
        f.write(wallet_hash)
```

### 2. Load Wallet Hash

Loads the wallet hash from a local file.

```python
def load_wallet_hash():
    if os.path.exists(LOGIN_PATH):
        with open(LOGIN_PATH, 'r') as f:
            return f.read().strip()
    return None
```

### 3. Create Wallet

Prompts the user for a passphrase to create a new wallet.

```python
import requests

def create_wallet():
    passphrase = input('Enter a passphrase to secure your wallet: ')
    response = requests.post(SERVER_URL + '/new_wallet', json={'passphrase': passphrase})
    wallet_hash = response.json()['wallet_hash']
    save_wallet_hash(wallet_hash)
    print(f'New wallet created: {wallet_hash}')
    print_wallet_balance(wallet_hash, passphrase)
```

### 4. Restore Wallet

Restores a wallet using its hash and passphrase.

```python
def restore_wallet(wallet_hash):
    passphrase = input('Enter your wallet passphrase: ')
    response = requests.post(SERVER_URL + '/restore_wallet', json={'wallet_hash': wallet_hash, 'passphrase': passphrase})
    if response.status_code == 200:
        save_wallet_hash(wallet_hash)
        print(f'Wallet restored: {wallet_hash}')
        print_wallet_balance(wallet_hash, passphrase)
    else:
        print('Wallet not found or incorrect passphrase')
```

### 5. Send Funds

Sends funds from the sender's wallet to a receiver's wallet.

```python
def send_funds(sender, passphrase):
    receiver = input('Enter receiver address: ')
    amount = int(input('Enter amount: '))
    response = requests.post(SERVER_URL + '/send', json={'sender': sender, 'receiver': receiver, 'amount': amount})
    if response.status_code == 200:
        print(f'Transaction successful: {response.json()}')
        print_wallet_balance(sender, passphrase)
    else:
        print(f'Error: {response.json()["error"]}')
```

### 6. Show Transactions

Displays the transactions for a given wallet.

```python
def show_transactions(wallet_hash):
    response = requests.post(SERVER_URL + '/transactions', json={'wallet_hash': wallet_hash})
    print('Transactions:')
    for tx in response.json()['transactions']:
        print(tx)
```

### 7. Print Wallet Balance

Prints the balance of a wallet.

```python
def print_wallet_balance(wallet_hash, passphrase):
    response = requests.post(SERVER_URL + '/restore_wallet', json={'wallet_hash': wallet_hash, 'passphrase': passphrase})
    if response.status_code == 200:
        balance = response.json()['balance']
        print(f'Wallet balance: {balance} GCOMC')
```

### 8. Main Function

Handles the main logic of the program, including creating, restoring wallets, sending funds, and showing transactions.

```python
def main():
    wallet_hash = load_wallet_hash()
    if wallet_hash:
        passphrase = input('Enter your wallet passphrase: ')
        print_wallet_balance(wallet_hash, passphrase)
    else:
        print('1. Make new wallet')
        print('2. Restore wallet')
        choice = input('Select an option: ')
        if choice == '1':
            create_wallet()
            wallet_hash = load_wallet_hash()
        elif choice == '2':
            wallet_hash = input('Enter your wallet hash: ')
            restore_wallet(wallet_hash)
            wallet_hash = load_wallet_hash()
        else:
            print('Invalid choice')
            return

    while True:
        print('1. Send funds')
        print('2. Show transactions')
        print('3. Exit')
        print('4. Show wallet hash (not recommended)')
        choice = input('Select an option: ')
        if choice == '1':
            send_funds(wallet_hash, passphrase)
        elif choice == '2':
            show_transactions(wallet_hash)
        elif choice == '3':
            break
        elif choice == '4':
            print(f'Wallet hash: {wallet_hash}')
        else:
            print('Invalid choice')

if __name__ == '__main__':
    print('GCOMC Wallet')
    main()
```

