# 📮 Complete Postman Testing Guide - E-commerce Microservices

## 🎯 Overview

This guide walks you through testing the 3 core microservices (Authentification, Produit, Commande) using Postman with real data, verifying both synchronous communication and JWT security.

---

## 📋 Table of Contents

- [Prerequisites & Setup](#prerequisites--setup)
- [Testing Flow](#testing-flow)
- [Service Endpoints](#service-endpoints)
- [MongoDB Verification](#mongodb-verification)
- [Troubleshooting](#troubleshooting)
- [Tips & Best Practices](#tips--best-practices)
- [Learning Outcomes](#learning-outcomes)

---

## Prerequisites & Setup

### Step 1: Import the Postman Collection

1. Download the JSON file: `Exp Microservices.postman_collection.json`
2. Open Postman ([download here](https://www.postman.com/downloads/) if needed)
3. Click **File** → **Import**
4. Select the JSON file
5. You should see a new collection with all services ✅

### Step 2: Start All Services

Start each service in a separate terminal:

```bash
# Terminal 1 - Auth Service (Port 4002)
cd auth-service && npm start

# Terminal 2 - Produit Service (Port 4000)
cd produit-service && npm start

# Terminal 3 - Commande Service (Port 4001)
cd commande-service && npm start

# Terminal 4 - MongoDB
mongod
```

---

## Testing Flow

⚠️ **Important:** Follow this exact order because the Commande service requires Product IDs and an Auth Token:

### 1️⃣ **SERVICE AUTHENTIFICATION** (Port 4002) - Security

#### Test 1: Register User

```
Method:  POST
URL:     http://localhost:4002/auth/register
Header:  Content-Type: application/json
```

**Request Body:**
```json
{
  "nom": "Sara",
  "email": "sara@example.com",
  "mot_passe": "password123"
}
```

**Expected Response:** ✅ `201 Created` with user details (password will be hashed)

---

#### Test 2: Login User

```
Method:  POST
URL:     http://localhost:4002/auth/login
Header:  Content-Type: application/json
```

**Request Body:**
```json
{
  "email": "sara@example.com",
  "mot_passe": "password123"
}
```

**Expected Response:** ✅ `200 OK` with JWT Token

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

⚠️ **Save this token!** You'll need it for the Commande service tests.

---

### 2️⃣ **SERVICE PRODUIT** (Port 4000) - Catalog

#### Test 1: Add First Product

```
Method:  POST
URL:     http://localhost:4000/produit/ajouter
Header:  Content-Type: application/json
```

**Request Body:**
```json
{
  "nom": "Laptop Pro",
  "description": "High performance laptop",
  "prix": 1200
}
```

**Expected Response:** ✅ `201 Created` with product ID

⚠️ **Save the `_id` from the response!**

---

#### Test 2: Add Second Product

```
Method:  POST
URL:     http://localhost:4000/produit/ajouter
Header:  Content-Type: application/json
```

**Request Body:**
```json
{
  "nom": "Wireless Mouse",
  "description": "Ergonomic mouse",
  "prix": 50
}
```

**Expected Response:** ✅ `201 Created` with product ID

⚠️ **Save this `_id` too!**

---

#### Test 3: Fetch Product Details

```
Method:  POST
URL:     http://localhost:4000/produit/acheter
Header:  Content-Type: application/json
```

**Request Body:**
```json
{
  "ids": [
    "PASTE_PRODUCT_1_ID_HERE",
    "PASTE_PRODUCT_2_ID_HERE"
  ]
}
```

**Expected Response:** ✅ Array containing details of the requested products

---

### 3️⃣ **SERVICE COMMANDE** (Port 4001) - Orders

⚠️ **This service requires the Auth Token and Product IDs from previous steps!**

#### Test 1: Create Order

```
Method:   POST
URL:      http://localhost:4001/commande/ajouter
Headers:  
  - Content-Type: application/json
  - Authorization: Bearer PASTE_YOUR_TOKEN_HERE
```

**Request Body:**
```json
{
  "ids": [
    "PASTE_PRODUCT_1_ID_HERE",
    "PASTE_PRODUCT_2_ID_HERE"
  ]
}
```

**Expected Response:** ✅ `201 Created`

```json
{
  "_id": "65a3456789012cdef1234567",
  "produits": ["id_1", "id_2"],
  "email_utilisateur": "sara@example.com",
  "prix_total": 1250,
  "created_at": "2026-04-25T10:30:00.000Z",
  "__v": 0
}
```

The service automatically:
- Decodes your email from the JWT token
- Fetches product prices from the Produit service via Axios
- Calculates the total price
- Saves the order

---

## Service Endpoints

| Service | Operation | Method | Port | Expected Status |
|---------|-----------|--------|------|-----------------|
| Auth | Register | POST | 4002 | 201 ✅ |
| Auth | Login | POST | 4002 | 200 ✅ (Returns Token) |
| Produit | Add Product 1 | POST | 4000 | 201 ✅ |
| Produit | Add Product 2 | POST | 4000 | 201 ✅ |
| Produit | Fetch Details | POST | 4000 | 201 ✅ |
| Commande | Create Order | POST | 4001 | 201 ✅ |

---

## MongoDB Verification

To verify data was saved in their isolated databases:

```bash
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
```

---

## Troubleshooting

### ❌ "Cannot POST /commande/ajouter" returns 401 Unauthorized

**Solution:** Make sure you included the `Authorization` header with `Bearer <your_token>` and that the token hasn't expired.

### ❌ "MongooseError: Cannot connect to MongoDB"

**Solution:** Start MongoDB with `mongod` in a separate terminal.

### ❌ Commande service returns 500 error on Add Order

**Solution:** Verify that the Produit service is running on port 4000. The Commande service uses Axios to make HTTP requests to `http://localhost:4000/produit/acheter`. If Produit is down, Commande will fail.

### ❌ "EADDRINUSE: address already in use"

**Solution:** Kill the port:
```bash
# Windows
netstat -ano | findstr :4000
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :4000
kill -9 <PID>
```

---

## Tips & Best Practices

### Use Postman Variables

Set up collection variables to avoid manual copy-pasting:

- `{{token}}` - Your JWT authentication token
- `{{produit_id_1}}` - First product ID
- `{{produit_id_2}}` - Second product ID

### Save Example Responses

Click **"Save Response"** in Postman to store example responses for reference.

### Run Collections

Click the arrow next to your collection name → **Run** to execute all tests in sequence.

---

## Learning Outcomes

After completing these tests, you will have verified:

✅ All 3 microservices are working independently on separate ports  
✅ Each service has its own isolated MongoDB database  
✅ JWT Authentication is protecting the Commande routes  
✅ Synchronous inter-service communication (Commande calling Produit via Axios) is fully functional