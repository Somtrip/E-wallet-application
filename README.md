# E-Wallet-App

# Curl to Onboard User:
curl --location 'localhost:8083/user-onboarding/create/user' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "Sagar",
    "email": "sagar@gmail.com",
    "phoneNo": "5697455253",
    "password": "12345",
    "userIdentifier": "AADHAAR",
    "userIdentifierValue": "56587452256",
    "dob": "14/02/1997",
    "address": "Vasundhra Ph 1 Delhi"
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
