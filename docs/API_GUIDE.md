# 🌐 Guide API

Documentation complète des endpoints de l'API Budget Manager.

## 📍 Base URL

```
http://localhost:8080/api/v1
```

## 🔐 Authentification

Tous les endpoints (sauf `/auth/*`) nécessitent un token JWT :

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## 📊 Format des réponses

### Succès
```json
{
  "id": 1,
  "field1": "value1"
}
```

### Erreur
```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "field1": "Error message"
  },
  "timestamp": "2024-02-20T10:30:00"
}
```

## 📝 Codes HTTP

| Code | Description |
|------|-------------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 500 | Internal Server Error |

---

## 🔑 Authentification

### Inscription

**Endpoint** : `POST /auth/register`

**Request** :
```json
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "SecurePass123!"
}
```

**Response** : `201 Created`
```json
{
  "token": "eyJhbGc...",
  "user": {
    "id": 1,
    "username": "johndoe",
    "email": "john@example.com"
  }
}
```

---

### Connexion

**Endpoint** : `POST /auth/login`

**Request** :
```json
{
  "username": "johndoe",
  "password": "SecurePass123!"
}
```

**Response** : `200 OK`
```json
{
  "token": "eyJhbGc...",
  "user": {
    "id": 1,
    "username": "johndoe",
    "email": "john@example.com"
  }
}
```

---

## 👤 Utilisateurs

### Obtenir le profil

**Endpoint** : `GET /users/me`

**Headers** : `Authorization: Bearer {token}`

**Response** : `200 OK`
```json
{
  "id": 1,
  "username": "johndoe",
  "email": "john@example.com",
  "createdAt": "2024-02-20T10:30:00"
}
```

---

## 🏦 Comptes

### Créer un compte

**Endpoint** : `POST /accounts`

**Request** :
```json
{
  "name": "Compte Courant",
  "type": "CHECKING",
  "balance": 1000.00,
  "currency": "EUR"
}
```

**Types** : `CHECKING`, `SAVINGS`, `CREDIT_CARD`

**Response** : `201 Created`
```json
{
  "id": 1,
  "name": "Compte Courant",
  "type": "CHECKING",
  "balance": 1000.00,
  "currency": "EUR"
}
```

---

### Lister les comptes

**Endpoint** : `GET /accounts`

**Response** : `200 OK`
```json
[
  {
    "id": 1,
    "name": "Compte Courant",
    "type": "CHECKING",
    "balance": 1000.00
  }
]
```

---

## 📊 Transactions

### Créer une transaction

**Endpoint** : `POST /transactions`

**Request** :
```json
{
  "description": "Achat supermarché",
  "amount": 45.80,
  "type": "EXPENSE",
  "date": "2024-02-20",
  "accountId": 1,
  "categoryId": 3
}
```

**Types** : `INCOME`, `EXPENSE`

**Response** : `201 Created`
```json
{
  "id": 1,
  "description": "Achat supermarché",
  "amount": 45.80,
  "type": "EXPENSE",
  "date": "2024-02-20",
  "account": {
    "id": 1,
    "name": "Compte Courant"
  },
  "category": {
    "id": 3,
    "name": "Alimentation",
    "icon": "🛒"
  }
}
```

---

### Lister les transactions

**Endpoint** : `GET /transactions`

**Query Parameters** :
- `accountId` : Filtrer par compte
- `categoryId` : Filtrer par catégorie
- `type` : `INCOME` ou `EXPENSE`
- `startDate` : Date début (YYYY-MM-DD)
- `endDate` : Date fin
- `page` : Numéro de page (défaut: 0)
- `size` : Taille de page (défaut: 20)
- `sort` : Tri (défaut: date,desc)

**Exemple** :
```
GET /transactions?type=EXPENSE&startDate=2024-01-01&page=0&size=10
```

**Response** : `200 OK` (paginé)

---

## 🏷️ Catégories

### Lister les catégories

**Endpoint** : `GET /categories`

**Query Parameters** :
- `type` : `INCOME` ou `EXPENSE`

**Response** : `200 OK`
```json
[
  {
    "id": 1,
    "name": "Salaire",
    "type": "INCOME",
    "icon": "💰",
    "isSystem": true
  },
  {
    "id": 3,
    "name": "Alimentation",
    "type": "EXPENSE",
    "icon": "🛒",
    "isSystem": true
  }
]
```

---

### Créer une catégorie

**Endpoint** : `POST /categories`

**Request** :
```json
{
  "name": "Hobbies",
  "type": "EXPENSE",
  "icon": "🎨",
  "color": "#9C27B0"
}
```

---

## 💰 Budgets

### Créer un budget

**Endpoint** : `POST /budgets`

**Request** :
```json
{
  "categoryId": 3,
  "amount": 300.00,
  "month": 2,
  "year": 2024
}
```

**Response** : `201 Created`
```json
{
  "id": 1,
  "category": {
    "id": 3,
    "name": "Alimentation"
  },
  "amount": 300.00,
  "spent": 0.00,
  "remaining": 300.00,
  "percentage": 0,
  "status": "ON_TRACK"
}
```

**Statuts** : `ON_TRACK`, `WARNING`, `EXCEEDED`

---

### Lister les budgets

**Endpoint** : `GET /budgets`

**Query Parameters** :
- `month` : Mois (1-12)
- `year` : Année

---

## 📈 Statistiques
[O
### Résumé général

**Endpoint** : `GET /statistics/summary`

**Query Parameters** :
- `startDate`
- `endDate`

**Response** : `200 OK`
```json
{
  "period": {
    "startDate": "2024-02-01",
    "endDate": "2024-02-29"
  },
  "totalIncome": 2500.00,
  "totalExpenses": 1856.30,
  "balance": 643.70,
  "savingsRate": 25.7,
  "transactionCount": 47
}
```

---

### Dépenses par catégorie

**Endpoint** : `GET /statistics/by-category`

**Response** : `200 OK`
```json
{
  "totalAmount": 1856.30,
  "categories": [
    {
      "category": {
        "name": "Alimentation",
        "icon": "🛒"
      },
      "amount": 456.80,
      "percentage": 24.6,
      "transactionCount": 15
    }
  ]
}
```

---

## 🔍 Recherche

### Recherche de transactions

**Endpoint** : `GET /transactions/search`

**Query Parameters** :
- `q` : Texte de recherche
- `minAmount` : Montant minimum
- `maxAmount` : Montant maximum
- `startDate`
- `endDate`
- `accountIds` : IDs séparés par virgule
- `categoryIds`
- `type`

**Exemple** :
```
GET /transactions/search?q=restaurant&minAmount=20&maxAmount=100
```

---

## 🧪 Tester l'API

### Avec cURL

**Inscription** :
```bash
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@example.com",
    "password": "SecurePass123!"
  }'
```

**Créer une transaction** :
```bash
curl -X POST http://localhost:8080/api/v1/transactions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "description": "Test",
    "amount": 50.00,
    "type": "EXPENSE",
    "date": "2024-02-20",
    "accountId": 1
  }'
```

---

## 📖 Documentation Swagger

Une fois l'application démarrée :

```
http://localhost:8080/swagger-ui.html
```

---

**Retour à** [Installation](INSTALLATION.md) | [Roadmap](ROADMAP.md)

