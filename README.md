📮 Complete Postman Testing Guide - E-commerce Microservices

🎯 Overview
This guide walks you through testing the 3 core microservices (Authentification, Produit, Commande) using Postman with real data, verifying both synchronous communication and JWT security.

📥 STEP 1 — Import the Postman Collection
Download the JSON file: Exp Microservices.postman_collection.json
Open Postman (download from [https://www.postman.com/downloads/](https://www.postman.com/downloads/) if needed)
Click File → Import
Select the JSON file
You should see a new collection with all services ✅

🚀 STEP 2 — Make sure all services are running
Before testing, verify the 3 microservices are running:

# Terminal 1
cd auth-service && npm start

# Terminal 2
cd produit-service && npm start

# Terminal 3
cd commande-service && npm start

Also make sure MongoDB is running:
mongod

📚 COMPLETE TESTING FLOW
Follow this exact order because the Commande service requires Product IDs and an Auth Token to work correctly:

1️⃣ SERVICE AUTHENTIFICATION (Port 4002) - Security
Test 1: POST Register User
Method: POST
URL: http://localhost:4002/auth/register
Header: Content-Type: application/json

Body:
{
  "nom": "Sara",
  "email": "sara@example.com",
  "mot_passe": "password123"
}
Expected Response: ✅ 201 Created with user details (password will be hashed).

Test 2: POST Login User
Method: POST
URL: http://localhost:4002/auth/login
Header: Content-Type: application/json

Body:
{
  "email": "sara@example.com",
  "mot_passe": "password123"
}
Expected Response: ✅ 200 OK with the JWT Token.
⚠️ COPY THIS TOKEN! You will need it for the Commande service.

{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

2️⃣ SERVICE PRODUIT (Port 4000) - Catalog
Test 1: POST Add Product
Method: POST
URL: http://localhost:4000/produit/ajouter
Header: Content-Type: application/json

Body:
{
  "nom": "Laptop Pro",
  "description": "High performance laptop",
  "prix": 1200
}
Expected Response: ✅ 201 Created with product ID.
⚠️ COPY the `_id` from the response!

Test 2: POST Add Another Product
Method: POST
URL: http://localhost:4000/produit/ajouter

Body:
{
  "nom": "Wireless Mouse",
  "description": "Ergonomic mouse",
  "prix": 50
}
Expected Response: ✅ 201 Created with the second product ID.
⚠️ COPY this `_id` too!

Test 3: POST Buy Products (Check Details)
Method: POST
URL: http://localhost:4000/produit/acheter
Header: Content-Type: application/json

Body:
{
  "ids": [
    "PASTE_PRODUCT_1_ID_HERE",
    "PASTE_PRODUCT_2_ID_HERE"
  ]
}
Expected Response: Array containing the details of the requested products.

3️⃣ SERVICE COMMANDE (Port 4001) - Orders
⚠️ This service requires the Auth Token and the Product IDs!

Test 1: POST Add Order
Method: POST
URL: http://localhost:4001/commande/ajouter
Headers: 
Content-Type: application/json
Authorization: Bearer PASTE_YOUR_TOKEN_HERE

Body:
{
  "ids": [
    "PASTE_PRODUCT_1_ID_HERE",
    "PASTE_PRODUCT_2_ID_HERE"
  ]
}
Expected Response: ✅ 201 Created. The service will automatically decode your email from the token, fetch the prices from the Produit service via Axios, calculate the total, and save the order.

{
  "_id": "65a3456789012cdef1234567",
  "produits": ["id_1", "id_2"],
  "email_utilisateur": "sara@example.com",
  "prix_total": 1250,
  "created_at": "2026-04-25T10:30:00.000Z",
  "__v": 0
}

📊 Complete Test Scenario Summary
Service         Operation         Method   Expected Status
Auth            Register          POST     201 ✅
Auth            Login             POST     200 ✅ (Returns Token)
Produit         Add product 1     POST     201 ✅
Produit         Add product 2     POST     201 ✅
Produit         Buy (Fetch details)POST    201 ✅
Commande        Add order         POST     201 ✅ (Requires Token & IDs)

🔍 Verifying Data in MongoDB
To verify data was actually saved in their isolated databases:

# Open MongoDB shell
mongosh

# List all databases
show databases

# Use the specific service databases
use auth-service
use produit-service
use commande-service

# View all documents
db.utilisateurs.find()
db.produits.find()
db.commandes.find()

🐛 Common Issues & Solutions
❌ "Cannot POST /commande/ajouter" returns 401 Unauthorized
Solution: Make sure you included the `Authorization` header with `Bearer <your_token>` and that the token hasn't expired.

❌ "MongooseError: Cannot connect to MongoDB"
Solution: Start MongoDB with `mongod` in a separate terminal.

❌ Commande service returns 500 error on Add Order
Solution: Verify that the Produit service is running on port 4000. The Commande service uses Axios to make an HTTP request to `http://localhost:4000/produit/acheter`. If Produit is down, Commande fails.

❌ "EADDRINUSE: address already in use"
Solution: Kill the port: `netstat -ano | findstr :4000` (Windows) then `taskkill /PID <PID> /F`

✨ Tips for Postman
Save responses: Click "Save Response" to save example responses
Use variables: Set up collection variables for `{{token}}`, `{{produit_id_1}}`, and `{{produit_id_2}}` so you don't have to copy/paste them manually.
Run collections: Click the arrow next to collection name → Run

🎓 Learning Outcomes
After completing these tests, you will have verified:
✅ All 3 microservices are working independently on separate ports.
✅ Each service has its own isolated MongoDB database.
✅ JWT Authentication is protecting the Commande routes.
✅ Synchronous inter-service communication (Commande calling Produit via Axios) is fully functional.
