# E-Wallet-App

# Curl to Onboard User:
curl --location 'localhost:8083/user-onboarding/create/user' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "Som",
    "email": "somnath.try@gmail.com",
    "phoneNo": "123456677",
    "password": "12345",
    "userIdentifier": "AADHAAR",
    "userIdentifierValue": "56587452256",
    "dob": "14/02/2003",
    "address": "BTM Layout, Bangalore"
}'

# Curl to Make Transaction
curl --location --request POST 'localhost:8082/txn-service/initiate/transaction?amount=4&receiver=5635878963&purpose=Sending%20Mney%20to%20robin%204rs' \
--header 'Authorization: Basic NTY5NzQ1NTI1MzoxMjM0NQ=='

# Curl to see transaction history
curl --location 'localhost:8082/txn-service/transaction/history?mobile=5697455253' \
--header 'mobile: 5697455253' \
--header 'Authorization: Basic NTY5NzQ1NTI1MzoxMjM0NQ=='

# Remember these Points
1. Kafka Should be up and running
2. Mysql Should be up and database should be created, take database name from application.properties file
3. Following topics needs to be created: user-details, txn-details, txn-update





# 💳 E-Wallet Application

A distributed Spring Boot microservices system enabling core digital wallet functionalities, including user onboarding, wallet creation, fund transfers, transaction tracking, and email notifications.

---

## 📌 Features
- ✅ User registration with KYC verification
- 💼 Wallet creation & balance management
- 🔁 Money transfer between users
- 📜 Transaction history
- ✉️ Email notifications on onboarding

---

## ⚙️ Architecture

```
[User Service] → Kafka → [Notification Service] 
               → Kafka → [Wallet Service]
[Transaction Service] → Kafka → [Wallet Service]
[Wallet Service] → Kafka → [Transaction Service]
```

---

## 🧰 Tech Stack
- **Java 17**, Spring Boot 3.2.5
- **MySQL**, Apache Kafka
- **Spring Security (Basic Auth)**, BCrypt
- **Maven** build
- **Mailtrap** for email testing

---

## 🧩 Microservices Overview

### 1. 👤 User Service (Port: 8083)
- **Endpoints:**
  - `POST /user-onboarding/create/user` — Create user
  - `GET /user-onboarding/get/user` — Fetch user
- **Highlights:**
  - KYC via PAN/AADHAAR/VOTER_ID
  - Password encryption
  - Publishes to Kafka topic: `user-details`

---

### 2. 💼 Wallet Service (Port: 8084)
- Creates wallet with ₹50 initial balance
- Transfers and updates balances
- Kafka:
  - **Listen**: `user-details`, `txn-details`
  - **Publish**: `txn-update`

---

### 3. 🔄 Transaction Service (Port: 8082)
- **Endpoints:**
  - `POST /txn-service/initiate/transaction`
  - `GET /txn-service/transaction/history`
- Handles money transfers and logs transactions
- Basic Auth via `phone:password`
- Kafka:
  - **Listen**: `txn-update`

---

### 4. 📩 Notification Service (Port: 8081)
- Sends welcome emails using Mailtrap
- Kafka:
  - **Listen**: `user-details`

---

## 🛠️ Setup Instructions

### ✅ Prerequisites
- Java 17
- Apache Kafka (local)
- MySQL

### 🗃️ Databases
- `walletuserdb` (User)
- `walletdb` (Wallet)
- `transactiondb` (Transactions)

### 🧵 Kafka Topics
```bash
kafka-topics --create --topic user-details --bootstrap-server localhost:9092
kafka-topics --create --topic txn-details --bootstrap-server localhost:9092
kafka-topics --create --topic txn-update --bootstrap-server localhost:9092
```

---

## 🚀 Running the App

```bash
# Common module
cd common
mvn install

# Services
cd ../UserService && mvn spring-boot:run
cd ../WalletService && mvn spring-boot:run
cd ../TxnService && mvn spring-boot:run
cd ../NotificationService && mvn spring-boot:run
```

---

## 📲 API Usage

### ➕ Create User
```bash
curl --location 'localhost:8083/user-onboarding/create/user' --header 'Content-Type: application/json' --data '{
  "name": "John Doe",
  "email": "john@example.com",
  "phoneNo": "1234567890",
  "password": "secure123",
  "userIdentifier": "AADHAAR",
  "userIdentifierValue": "56587452256",
  "dob": "01/01/1990",
  "address": "City, Country"
}'
```

### 💸 Initiate Transaction
```bash
curl --location --request POST 'localhost:8082/txn-service/initiate/transaction?amount=100&receiver=9876543210&purpose=Payment' --header 'Authorization: Basic <base64(phone:password)>'
```

### 📜 Transaction History
```bash
curl 'localhost:8082/txn-service/transaction/history?mobile=1234567890' --header 'Authorization: Basic <base64(phone:password)>'
```

---

## 🔒 Security
- Basic Auth with phone number
- Password hashing using BCrypt
- Role-based access (`USER`, `ADMIN`)

---

## 📁 Config Files
Each service has its `application.properties` under:
- `/UserService/src/main/resources/`
- `/WalletService/src/main/resources/`
- `/TxnService/src/main/resources/`
- `/NotificationService/src/main/resources/`

---

## 🧱 Key Classes & Components

### Common Module
- `CommonConstants`
- `UserIdentifier` enum

### User Service
- `UserController`, `UserService`, `User`

### Wallet Service
- `TransactionWorker`, `Wallet`, `OnboardedUserConsumer`

### Transaction Service
- `TransactionController`, `TransactionService`, `Transaction`

### Notification Service
- `EmailWorker`, `UserCreatedConsumer`

---

## 🧩 Database Schema

### User Service: `walletuserdb`
```sql
CREATE TABLE user (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255), email VARCHAR(255) UNIQUE,
  phone VARCHAR(255) UNIQUE, address TEXT,
  dob VARCHAR(20), password VARCHAR(255),
  user_identifier ENUM('PAN','AADHAAR','VOTER_ID'),
  user_identifier_value VARCHAR(255),
  role ENUM('USER','ADMIN'),
  user_status ENUM('ACTIVE','INACTIVE','BLOCKED'),
  created_on DATETIME, updated_on DATETIME
);
```

### Wallet Service: `walletdb`
```sql
CREATE TABLE wallet (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id VARCHAR(255), balance DOUBLE,
  phone_no VARCHAR(255) UNIQUE,
  user_identifier VARCHAR(50),
  user_identifier_value VARCHAR(255),
  created_on DATETIME, updated_on DATETIME
);
```

### Transaction Service: `transactiondb`
```sql
CREATE TABLE transaction (
  id INT AUTO_INCREMENT PRIMARY KEY,
  txn_id VARCHAR(255), sender_id VARCHAR(255),
  receiver_id VARCHAR(255), amount DOUBLE,
  purpose VARCHAR(255),
  txn_status ENUM('INITIATED','FAILED','SUCCESS','PENDING'),
  txn_message TEXT,
  created_on DATETIME, updated_on DATETIME
);
```

---

## ❗ Error Handling
- Global `@ControllerAdvice` handling
- DB & Kafka exception coverage
- Transaction status resolution via Kafka

---

## ✅ Testing
```bash
mvn test
```

---

## 📡 Monitoring
- Spring Actuator endpoints
- Kafka CLI tools
- DB consistency checks

---

## 🧠 Troubleshooting

| Issue                       | Solution                                 |
|----------------------------|------------------------------------------|
| Kafka not reachable        | Ensure Kafka runs on `localhost:9092`    |
| DB connection errors       | Check credentials in `application.properties` |
| Auth failures              | Ensure correct `Base64(phone:password)`  |
| Txn not processed          | Check wallet balance & receiver user     |

---

## 🌱 Future Enhancements
- Swagger API docs
- Circuit Breakers (Resilience4j)
- JWT Authentication
- Wallet balance check endpoint
- Admin dashboard
- Transaction reversal system

---

## 📂 Repository Structure
```
.
├── common/
├── UserService/
├── WalletService/
├── TxnService/
├── NotificationService/
└── README.md
```

---

## 📬 Contact
For any queries or contributions, feel free to open issues or raise pull requests.

---
