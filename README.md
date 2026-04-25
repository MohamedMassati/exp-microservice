# 📮 Complete Postman Testing Guide - Exp Microservice

## 🎯 Overview
This guide walks you through testing all microservices using Postman with real data and all HTTP methods (GET, POST, PUT, DELETE).

## 📥 STEP 1 — Import the Postman Collection
1. Download the JSON file: `exp-microservices.postman_collection.json`
2. Open Postman (download from https://www.postman.com/downloads/ if needed)
3. Click **File → Import**
4. Select the JSON file
5. You should see a new collection with all services ✅

## 🚀 STEP 2 — Make sure all services are running

Before testing, verify all services are running:

```bash
# Terminal 1 - Service 1
cd service-1 && node index.js

# Terminal 2 - Service 2
cd service-2 && node index.js

# Terminal 3 - Service 3
cd service-3 && node index.js

# Terminal 4 - API Gateway
cd api-gateway && node index.js
```

Also make sure MongoDB is running:

```bash
mongod
```

## 📚 COMPLETE TESTING FLOW

Follow this order to test everything correctly:

### 1️⃣ SERVICE 1 (Port 3001) - Primary Operations

**Test 1: POST create resource**
- **Method:** POST
- **URL:** `http://localhost:3001/api/v1/resource`
- **Header:** `Content-Type: application/json`

Body:
```json
{
  "name": "Resource 1",
  "description": "Test resource",
  "status": "active"
}
```

Expected Response: ✅ 200 OK with resource ID

**Test 2: POST create another resource**
- **Method:** POST
- **URL:** `http://localhost:3001/api/v1/resource`

Body:
```json
{
  "name": "Resource 2",
  "description": "Another test resource",
  "status": "active"
}
```

**Test 3: POST create third resource**
- **Method:** POST
- **URL:** `http://localhost:3001/api/v1/resource`

Body:
```json
{
  "name": "Resource 3",
  "description": "Third resource",
  "status": "inactive"
}
```

**Test 4: GET all resources**
- **Method:** GET
- **URL:** `http://localhost:3001/api/v1/resource`
- Expected Response: Array of 3 resources

**Test 5: GET resource by ID**
- **Method:** GET
- **URL:** `http://localhost:3001/api/v1/resource/65a1234567890abcdef12345`
- Expected Response: Single resource object

**Test 6: PUT update resource**
- **Method:** PUT
- **URL:** `http://localhost:3001/api/v1/resource/65a1234567890abcdef12345`

Body:
```json
{
  "status": "inactive"
}
```

Expected Response: Updated resource

**Test 7: DELETE resource (optional)**
- **Method:** DELETE
- **URL:** `http://localhost:3001/api/v1/resource/65a1234567890abcdef12345`

### 2️⃣ SERVICE 2 (Port 3002) - Secondary Operations

**Test 1: POST create entity**
- **Method:** POST
- **URL:** `http://localhost:3002/api/v1/entity`

Body:
```json
{
  "title": "Entity 1",
  "content": "Test content",
  "type": "type_a"
}
```

Expected Response: ✅ 200 OK with entity ID

**Test 2: GET all entities**
- **Method:** GET
- **URL:** `http://localhost:3002/api/v1/entity`

**Test 3: GET entity by ID**
- **Method:** GET
- **URL:** `http://localhost:3002/api/v1/entity/{entity_id}`

**Test 4: PUT update entity**
- **Method:** PUT
- **URL:** `http://localhost:3002/api/v1/entity/{entity_id}`

Body:
```json
{
  "type": "type_b"
}
```

**Test 5: DELETE entity**
- **Method:** DELETE
- **URL:** `http://localhost:3002/api/v1/entity/{entity_id}`

### 3️⃣ SERVICE 3 (Port 3003) - Data Management

**Test 1: POST create data**
- **Method:** POST
- **URL:** `http://localhost:3003/api/v1/data`

Body:
```json
{
  "key": "data_key_1",
  "value": "data_value_1"
}
```

**Test 2: GET all data**
- **Method:** GET
- **URL:** `http://localhost:3003/api/v1/data`

**Test 3: GET data by ID**
- **Method:** GET
- **URL:** `http://localhost:3003/api/v1/data/{data_id}`

**Test 4: PUT update data**
- **Method:** PUT
- **URL:** `http://localhost:3003/api/v1/data/{data_id}`

Body:
```json
{
  "value": "updated_value"
}
```

**Test 5: DELETE data**
- **Method:** DELETE
- **URL:** `http://localhost:3003/api/v1/data/{data_id}`

### 4️⃣ API GATEWAY (Port 3000) - Routing Tests

**Test 1: Route to Service 1**
- **Method:** GET
- **URL:** `http://localhost:3000/service1/api/v1/resource`

**Test 2: Route to Service 2**
- **Method:** GET
- **URL:** `http://localhost:3000/service2/api/v1/entity`

**Test 3: Route to Service 3**
- **Method:** GET
- **URL:** `http://localhost:3000/service3/api/v1/data`

## 📊 Complete Test Scenario Summary

| Service | Operation | Method | Expected Status |
|---------|-----------|--------|-----------------|
| Service 1 | Create resource | POST | 200 ✅ |
| Service 1 | Get all | GET | 200 ✅ |
| Service 1 | Get by ID | GET | 200 ✅ |
| Service 1 | Update | PUT | 200 ✅ |
| Service 1 | Delete | DELETE | 200 ✅ |
| Service 2 | Create entity | POST | 200 ✅ |
| Service 2 | Get all | GET | 200 ✅ |
| Service 2 | Get by ID | GET | 200 ✅ |
| Service 2 | Update | PUT | 200 ✅ |
| Service 2 | Delete | DELETE | 200 ✅ |
| Service 3 | Create data | POST | 200 ✅ |
| Service 3 | Get all | GET | 200 ✅ |
| Service 3 | Get by ID | GET | 200 ✅ |
| Service 3 | Update | PUT | 200 ✅ |
| Service 3 | Delete | DELETE | 200 ✅ |
| Gateway | Route Service 1 | GET | 200 ✅ |
| Gateway | Route Service 2 | GET | 200 ✅ |
| Gateway | Route Service 3 | GET | 200 ✅ |

## 🔍 Verifying Data in MongoDB

To verify data was actually saved:

```bash
# Open MongoDB shell
mongosh

# List all databases
show databases

# Use microservices databases
use exp_service_1
use exp_service_2
use exp_service_3

# View all documents
db.resources.find()
db.entities.find()
db.data.find()

# Count documents
db.resources.countDocuments()
```

## 🐛 Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| ❌ "Cannot POST /" | Make sure API Gateway is running on port 3000 |
| ❌ "MongooseError: Cannot connect to MongoDB" | Start MongoDB with `mongod` |
| ❌ "Cannot connect to localhost:3001" | Make sure all services are running in separate terminals |
| ❌ Returns empty array | Add test data first using POST requests |
| ❌ "EADDRINUSE: address already in use" | Kill the port: `netstat -ano \| findstr :3000` (Windows) then `taskkill /PID <PID> /F` |
| ❌ "CORS error" | Verify CORS configuration in API Gateway |
| ❌ Service not responding | Check if service is properly initialized and MongoDB connection is active |

## ✨ Tips for Postman

- **Save responses:** Click "Save Response" to save example responses
- **Use variables:** Create environment variables for base URLs and IDs
- **Run collections:** Click the arrow next to collection name → Run
- **View assertions:** Look at Tests tab to verify responses
- **Check timeline:** Click "Timeline" to see network timing
- **Generate code:** Select any request → Code to see equivalent code in multiple languages
- **Test scripts:** Use the Tests tab to automate response validation

## 🎓 Learning Outcomes

After completing these tests, you will have verified:

✅ All microservices are working correctly
✅ API Gateway routing all requests properly
✅ Each service has its own MongoDB database
✅ CRUD operations (Create, Read, Update, Delete)
✅ Synchronous (REST) communication
✅ Service-to-service communication
✅ Service independence (one failure doesn't break others)
✅ Data persistence and retrieval

## 📦 Project Structure

```
exp-microservice/
├── service-1/
│   ├── index.js
│   ├── routes/
│   ├── models/
│   └── package.json
├── service-2/
│   ├── index.js
│   ├── routes/
│   ├── models/
│   └── package.json
├── service-3/
│   ├── index.js
│   ├── routes/
│   ├── models/
│   └── package.json
├── api-gateway/
│   ├── index.js
│   ├── middleware/
│   └── package.json
├── exp-microservices.postman_collection.json
└── README.md
```

## 🚀 Quick Start

1. Clone the repository
2. Install dependencies for each service: `npm install`
3. Ensure MongoDB is running
4. Start all services in separate terminals
5. Import Postman collection
6. Follow the testing flow above

## 📝 Notes

- Each service runs on its own port (3001, 3002, 3003)
- API Gateway runs on port 3000
- Each service has its own MongoDB database
- All requests should have `Content-Type: application/json` header for POST/PUT requests
- Keep the real IDs from previous requests to use in subsequent tests

---

Happy Testing! 🚀

**Created:** 2026-04-25 19:50:07
**Version:** 1.0
**Status:** Active