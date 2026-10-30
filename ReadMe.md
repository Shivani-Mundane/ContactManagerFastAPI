# Contact Manager API

A complete FastAPI REST API for managing contacts with JWT authentication, built following clean layered architecture.

---

## Folder Structure

```
contact_manager/
├── requirements.txt
├── README.md
└── app/
    ├── __init__.py
    ├── main.py
    ├── core/         (config.py, database.py, security.py, exceptions.py, dependencies.py)
    ├── routers/      (auth_router.py, contact_router.py)
    ├── controller/   (auth_controller.py, contact_controller.py)
    ├── service/      (auth_service.py, contact_service.py)
    ├── repository/   (user_repository.py, contact_repository.py)
    ├── models/       (user.py, contact.py, __init__.py importing both)
    └── schema/       (user.py, token.py, contact.py)
```

### Architecture Rule & Request Flow
```
router -> controller -> service -> repository -> database
```
- **routers**: Declare endpoints, route paths, HTTP status codes, response schemas, and forward calls to controllers.
- **controller**: Orchestrates request processing and translates domain exceptions into standard HTTP exceptions (`400`, `401` with `WWW-Authenticate: Bearer`, `404`).
- **service**: Contains business logic (password hashing, JWT creation, existence validation). Raises domain exceptions (`ConflictError`, `AuthenticationError`, `NotFoundError`).
- **repository**: Handles all database interactions using SQLAlchemy 2.x ORM sessions.
- **core/dependencies.py**: Factory functions assembling repository $\to$ service $\to$ controller per request using FastAPI's dependency injection system, along with the `get_current_user` token validator.

---

## Setup & Running with MySQL

### 1. Create MySQL Database
Run in your MySQL client:
```sql
CREATE DATABASE contact_manager CHARACTER SET utf8mb4;
```

### 2. Environment Setup
Create and activate a virtual environment:
```bash
cd contact_manager
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 3. Configure Environment Variables
Set your database credentials and secret key:
```bash
export DATABASE_URL="mysql+pymysql://<user>:<password>@<host>:3306/contact_manager"
export SECRET_KEY="your-strong-production-secret-key"
```

### 4. Run the Server
```bash
uvicorn app.main:app --reload
```
Interactive Swagger docs are available at `http://localhost:8000/docs`.

---

## Endpoints

| Method | Endpoint | Auth Required | Description |
|---|---|---|---|
| `POST` | `/auth/register` | No | Register a new user |
| `POST` | `/auth/login` | No | Login (OAuth2 form) and receive JWT |
| `POST` | `/contacts` | Yes | Create a new contact |
| `GET` | `/contacts` | Yes | List contacts (supports `skip`, `limit`, and `search`) |
| `GET` | `/contacts/{id}` | Yes | Get a single contact |
| `PUT` | `/contacts/{id}` | Yes | Partially update a contact |
| `DELETE` | `/contacts/{id}` | Yes | Delete a contact |

---

## cURL Examples

### 1. Register
```bash
curl -X POST "http://localhost:8000/auth/register" \
  -H "Content-Type: application/json" \
  -d '{"username": "johndoe", "password": "securepassword123"}'
```

### 2. Login
```bash
curl -X POST "http://localhost:8000/auth/login" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=johndoe&password=securepassword123"
```
*Response:*
```json
{
  "access_token": "<YOUR_ACCESS_TOKEN>",
  "token_type": "bearer"
}
```

### 3. Create Contact
```bash
curl -X POST "http://localhost:8000/contacts" \
  -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Jane Doe",
    "email": "jane@example.com",
    "phone": "+1234567890",
    "address": "123 Elm Street"
  }'
```

### 4. List Contacts (with optional query parameters)
```bash
curl -X GET "http://localhost:8000/contacts?skip=0&limit=10&search=Jane" \
  -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>"
```

### 5. Get Contact by ID
```bash
curl -X GET "http://localhost:8000/contacts/1" \
  -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>"
```

### 6. Update Contact
```bash
curl -X PUT "http://localhost:8000/contacts/1" \
  -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"phone": "+9876543210"}'
```

### 7. Delete Contact
```bash
curl -X DELETE "http://localhost:8000/contacts/1" \
  -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>"
```

