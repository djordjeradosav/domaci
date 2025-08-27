# Postman API Testiranje Vodič - Laravel Client Invoice API

Ovaj vodič objašnjava korak po korak kako da testiraš Laravel API pomoću Postman kolekcije.

## 📋 Preduslovi

- Postman aplikacija instalirana
- Laravel aplikacija pokrenuta
- Database sa potrebnim tabelama i podacima

## 📥 KORAK 1: Import kolekcije u Postman

1. **Otvori Postman aplikaciju**
2. **Klikni na "Import"** dugme (gore levo)
3. **Izaberi "Raw text" tab**
4. **Kopiraj JSON kod** iz Postman kolekcije
5. **Paste-uj u text polje**
6. **Klikni "Continue"** → **"Import"**

## 🌍 KORAK 2: Kreiranje Environment-a

1. **Klikni na "Environments"** (leva strana)
2. **Klikni "+" za novo environment**
3. **Nazovi ga "Laravel API Test"**
4. **Dodaj sledeće varijable:**

| Variable | Initial Value | Current Value |
|----------|---------------|---------------|
| baseUrl | `http://localhost:8000/api` | `http://localhost:8000/api` |
| token | (prazno) | (prazno) |
| admin_token | (prazno) | (prazno) |
| client_id | (prazno) | (prazno) |
| invoice_id | (prazno) | (prazno) |
| contact_id | (prazno) | (prazno) |
| user_id | (prazno) | (prazno) |

5. **Klikni "Save"**
6. **Selektuj ovaj environment** u dropdown-u gore desno

## 🚀 KORAK 3: Pokretanje Laravel servera

**U terminalu:**
```bash
cd domaci1
php artisan serve
```

Proveri da li server radi na `http://localhost:8000`

## 🔐 KORAK 4: Testiranje Autentifikacije

### 4.1 Registracija Admin korisnika

**Request Body za registraciju admin-a:**
```json
{
    "name": "Admin User",
    "email": "admin@test.com",
    "password": "password123",
    "password_confirmation": "password123",
    "role": "admin",
    "company": "Admin Company",
    "phone": "+381123456789"
}
```

**Koraci:**
1. **Otvori folder "01 - Autentifikacija"**
2. **Klikni "Registracija Admin"**
3. **Klikni "Send"**
4. **Očekivani rezultat:** Status 201, odgovor sa token-om
5. **Proveri:** Environment varijabla `admin_token` je automatski popunjena

### 4.2 Login Admin

**Request Body za login:**
```json
{
    "email": "admin@test.com",
    "password": "password123"
}
```

**Koraci:**
1. **Klikni "Login Admin"**
2. **Klikni "Send"**
3. **Očekivani rezultat:** Status 200, token se čuva

### 4.3 Proveri current user
1. **Klikni "Get Current User"**
2. **Klikni "Send"**
3. **Očekivani rezultat:** Podaci o korisniku sa role: "Admin"

### 4.4 Reset password

**Request Body:**
```json
{
    "current_password": "password123",
    "new_password": "newpassword123",
    "new_password_confirmation": "newpassword123"
}
```

### 4.5 Kreiranje dodatnih korisnika

**User role:**
```json
{
    "name": "Regular User",
    "email": "user@test.com",
    "password": "password123",
    "password_confirmation": "password123",
    "role": "user"
}
```

**Client role:**
```json
{
    "name": "Client User",
    "email": "client@test.com",
    "password": "password123",
    "password_confirmation": "password123",
    "role": "client"
}
```

## 👥 KORAK 5: Testiranje Client ruta

### 5.1 Kreiranje klijenta

**Request Body:**
```json
{
    "name": "Test Client",
    "email": "testclient@example.com",
    "phone": "+381123456789",
    "company": "Test Company Ltd",
    "address": "Test Address 123, Belgrade, Serbia",
    "status": "Active"
}
```

**Koraci:**
1. **Otvori folder "02 - Clients"**
2. **Klikni "Create Client (Admin)"**
3. **Proveri da li je admin_token selektovan**
4. **Klikni "Send"**
5. **Očekivani rezultat:** Status 201, `client_id` se čuva automatski

### 5.2 Lista klijenata
1. **Klikni "Get All Clients"**
2. **Klikni "Send"**
3. **Očekivani rezultat:** Lista sa kreiranim klijentom

### 5.3 Pojedinačni klijent
1. **Klikni "Get Single Client"**
2. **Klikni "Send"**
3. **Primetiti:** URL koristi `{{client_id}}` varijablu

### 5.4 Ažuriranje klijenta

**Request Body:**
```json
{
    "name": "Updated Test Client",
    "email": "testclient@example.com",
    "phone": "+381123456789",
    "company": "Updated Company Ltd",
    "address": "Updated Address 456, Belgrade, Serbia",
    "status": "Active"
}
```

### 5.5 Brisanje klijenta
- **Samo admin može da briše klijente**
- **Klikni "Delete Client (Admin Only)"**

## 📞 KORAK 6: Testiranje Contact ruta

### 6.1 Kreiranje kontakta

**Request Body:**
```json
{
    "client_id": {{client_id}},
    "contact_name": "John Doe",
    "contact_email": "john.doe@testclient.com",
    "contact_phone": "+381987654321"
}
```

**Koraci:**
1. **Otvori folder "03 - Contacts"**
2. **Klikni "Create Contact"**
3. **Proveri Body:** `client_id` koristi `{{client_id}}` varijablu
4. **Klikni "Send"**
5. **Očekivani rezultat:** Status 201, `contact_id` se čuva

### 6.2 Ostale contact operacije
- **Get All Contacts** - Lista svih kontakata
- **Get Single Contact** - Koristi `{{contact_id}}`
- **Update Contact** - Ažuriraj podatke kontakta
- **Delete Contact** - Obriši kontakt

**Update Contact Body:**
```json
{
    "contact_name": "John Updated Doe",
    "contact_email": "john.updated@testclient.com",
    "contact_phone": "+381987654322"
}
```

## 💰 KORAK 7: Testiranje Invoice ruta

### 7.1 Kreiranje fakture

**Request Body:**
```json
{
    "client_id": {{client_id}},
    "amount": 1000.00,
    "tax_amount": 200.00,
    "issue_date": "2024-12-01",
    "due_date": "2024-12-31",
    "notes": "Test invoice for services",
    "status": "Draft",
    "items": [
        {
            "description": "Web Development",
            "quantity": 10,
            "rate": 100.00,
            "amount": 1000.00
        }
    ]
}
```

**Koraci:**
1. **Otvori folder "04 - Invoices"**
2. **Klikni "Create Invoice (Admin)"**
3. **Proveri:** Koristi admin_token i client_id
4. **Klikni "Send"**
5. **Očekivani rezultat:** Status 201, `invoice_id` se čuva

### 7.2 Ostale invoice operacije
- **Get All Invoices** - Paginirana lista
- **Get Single Invoice** - Koristi `{{invoice_id}}`
- **Update Invoice** - Ažuriraj fakturu
- **Delete Invoice** - Obriši fakturu (samo admin)

### 7.3 Invoice statusi
Dostupni statusi: `Draft`, `Sent`, `Paid`, `Overdue`, `Cancelled`

## 👑 KORAK 8: Admin funkcije

### 8.1 User management (Admin only)
1. **Otvori folder "05 - Admin Only Routes"**
2. **Get All Users** - Lista svih korisnika
3. **Get Single User** - Koristi `{{user_id}}`
4. **Update User** - Ažuriraj korisnika

**Update User Body:**
```json
{
    "name": "Updated Admin User",
    "email": "admin@test.com"
}
```

**Napomena:** Samo admin može pristupiti ovim rutama!

## ❌ KORAK 9: Error Testing

### 9.1 Test bez tokena
1. **Otvori folder "06 - Error Testing"**
2. **"Unauthorized Access (No Token)"** 
3. **Očekivani rezultat:** Status 401 Unauthorized

### 9.2 Test pogrešnih privilegija

**Koraci:**
1. **Prvo se uloguj kao obični user:**
   - Idi na "01 - Autentifikacija"
   - "Login User" → token se čuva u `token` varijabli
2. **"User Try to Create Client"** 
3. **Očekivani rezultat:** Status 403 Forbidden

### 9.3 Test pogrešnih credentials
1. **"Invalid Login Credentials"**
2. **Očekivani rezultat:** Status 422 Validation Error

## 🔧 KORAK 10: Debugging

### Proverava Console
**View → Show Postman Console**

Očekivane poruke:
```
Token saved: 1|abc123...
Client ID saved: 1
Invoice ID saved: 1
Contact ID saved: 1
```

### Proveri Environment varijable
- **Klikni na "eye" ikonu** pored environment dropdown-a
- **Vidi da li su varijable popunjene**

### Česti problemi i rešenja

| Status | Problem | Rešenje |
|--------|---------|---------|
| 401 | Unauthorized | Token nije poslat ili je nevažeći |
| 403 | Forbidden | Korisnik nema dozvolu za operaciju |
| 404 | Not Found | Pogrešan URL ili ID ne postoji |
| 422 | Validation Error | Pogrešni ili nepotpuni podaci u Body |
| 500 | Server Error | Proveri Laravel logs |

### Laravel logs
```bash
tail -f storage/logs/laravel.log
```

## 📊 KORAK 11: Pokretanje cele kolekcije

### Collection Runner
1. **Klikni na "Laravel Client Invoice API" kolekciju**
2. **Klikni "Run collection"**
3. **Selektuj folder koji želiš da testiraš**
4. **Postavke:**
   - Iterations: 1
   - Delay: 0ms
   - Data file: None
5. **Klikni "Run Laravel Client..."**

### Redosled pokretanja
1. **01 - Autentifikacija** (obavezno prvo)
2. **02 - Clients**
3. **03 - Contacts**
4. **04 - Invoices**
5. **05 - Admin Only Routes**
6. **06 - Error Testing**

## 💡 Pro Tips

### 1. Custom Pre-request Script
```javascript
// Dodaj timestamp za unique email-ove
pm.environment.set("timestamp", Date.now());

// Debug environment varijable
console.log("Current token:", pm.environment.get("token"));
console.log("Base URL:", pm.environment.get("baseUrl"));
```

### 2. Custom Tests
```javascript
// Test response time
pm.test("Response time is less than 200ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(200);
});

// Test JSON structure
pm.test("Response has required fields", function () {
    const response = pm.response.json();
    pm.expect(response).to.have.property('success');
    pm.expect(response).to.have.property('data');
});

// Test status codes
pm.test("Status code is 201 for creation", function () {
    pm.response.to.have.status(201);
});
```

### 3. Environment switching
Kreiraj različite environment-e:
- **Development**: `http://localhost:8000/api`
- **Testing**: `http://test.example.com/api`
- **Production**: `https://api.example.com/api`

### 4. Export rezultata
- **Nakon pokretanja kolekcije → Export Results**
- **Format:** JSON ili HTML
- **Čuvaj za dokumentaciju**

## 🔒 Security Testing

### JWT Token validation
```javascript
// Test da token postoji i ima pravilan format
pm.test("Token is valid JWT format", function () {
    const token = pm.environment.get("token");
    pm.expect(token).to.match(/^\d+\|[A-Za-z0-9_-]+$/);
});
```

### Role-based access
```javascript
// Test admin permissions
pm.test("Admin can access user management", function () {
    if (pm.request.headers.get("Authorization").includes("admin_token")) {
        pm.response.to.have.status(200);
    }
});
```

## 📈 Performance Testing

### Load Testing setup
```javascript
// Pre-request script za load testing
const delay = Math.floor(Math.random() * 1000);
setTimeout(() => {}, delay);
```

### Monitoring
```javascript
// Track response times
pm.test("API performance check", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});
```

## 🎯 API Endpoints Summary

### Public Routes
- `POST /api/register` - Registracija
- `POST /api/login` - Login

### Authenticated Routes
- `GET /api/user` - Current user info
- `POST /api/logout` - Logout
- `POST /api/reset-password` - Reset password

### Client Routes
- `GET /api/clients` - Lista klijenata
- `POST /api/clients` - Kreiraj klijenta (Admin only)
- `GET /api/clients/{id}` - Pojedinačni klijent
- `PUT /api/clients/{id}` - Ažuriraj klijenta (Admin only)
- `DELETE /api/clients/{id}` - Obriši klijenta (Admin only)

### Invoice Routes
- `GET /api/invoices` - Lista faktura
- `POST /api/invoices` - Kreiraj fakturu (Admin only)
- `GET /api/invoices/{id}` - Pojedinačna faktura
- `PUT /api/invoices/{id}` - Ažuriraj fakturu (Admin only)
- `DELETE /api/invoices/{id}` - Obriši fakturu (Admin only)

### Contact Routes
- `GET /api/contacts` - Lista kontakata
- `POST /api/contacts` - Kreiraj kontakt
- `GET /api/contacts/{id}` - Pojedinačni kontakt
- `PUT /api/contacts/{id}` - Ažuriraj kontakt
- `DELETE /api/contacts/{id}` - Obriši kontakt

### Admin Routes
- `GET /api/users` - Lista korisnika
- `GET /api/users/{id}` - Pojedinačni korisnik
- `PUT /api/users/{id}` - Ažuriraj korisnika
- `DELETE /api/users/{id}` - Obriši korisnika

---

## ✅ Checklist za kompleto testiranje

- [ ] Import Postman kolekcije
- [ ] Setup Environment varijabli
- [ ] Pokretanje Laravel servera
- [ ] Registracija admin korisnika
- [ ] Login admin korisnika
- [ ] Kreiranje test klijenta
- [ ] Kreiranje test kontakta
- [ ] Kreiranje test fakture
- [ ] Testiranje admin funkcija
- [ ] Testiranje error scenarija
- [ ] Validacija svih response-a
- [ ] Export test rezultata

Ovaj vodič pokriva sve aspekte testiranja Laravel API-ja kroz Postman kolekciju. Koristi ga kao referentni materijal tokom razvoja i testiranja! 🚀
