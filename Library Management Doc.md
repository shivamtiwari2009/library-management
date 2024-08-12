# Library Management

# Core entities

1. User
2. Employee
3. Employee hierarchy
4. Author
5. Publisher
6. Library Location
7. Library Sub Location
8. Auditing
9. Tenant
10. Book
11. Genre
12. Book lending
13. Return book
14. Fines 
15. Member (people who are going to take books)
16. Workflows
17. Workflow trigger
18. Workflow actions

# Services

1. User Service
2. Member Service
3. Search Service
4. Book Management Service
5. Book Lending Service
6. Notification Service
7. Analytics Service
8. Billing Service
9. Config Service
10. Library Card Service
11. Workflows Service
12. Operator Service

# APIs

We will have two sets of api

1. Admin - API prefix `/api/tenant/{tenant_id}/`
2. Member - API prefix `/api/tenant/{tenant_id}/member/`

Most of member APIs will also have admin version too

# User Management Service

### Pre-Defined Roles

1. Admin
2. Employee
3. Member

**Note**

- One can also create custom roles
- A user can have multiple roles.

### JWT token Authorization

**Token Claims**

1. tenant_id: UUID
2. user_id: UUID
3. member_id: UUID
4. roles: array (name of list of roles)

### Table (tenant)

1. id:UUID PRIMARY KEY
2. created_at: timestamp with timezone UTC
3. updated_at: timestamp with timezone UTC
4. deleted_at:  timestamp with timezone UTC
5. name:TEXT (UNIQUE)
6. domain: TEXT (e.g nyclib.lm.com)
7. address: UUID REFERENCES address(id)
8. currency_code: varchar(5)
9. country_code: varchar(5)

### Table (user)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. deleted_at:  timestamp with timezone UTC
6. first_name: TEXT
7. last_name: TEXT
8. full_name: TEXT (auto population, can easily search by full name)
9. email: TEXT
10. photo_ids: array (s3 links)
11. password: TEXT (encrypted)
12. phone_number: jsonb (pair or country code and phone number)
13. address: UUID REFERENCES address(id)

### Table (address)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. first_name: TEXT
6. last_name: TEXT
7. phone_number:  jsonb (pair or country code and phone number)
8. address1: TEXT
9. address2:TEXT
10. city:TEXT
11. state:TEXT
12. country:TEXT
13. zip_code:TEXT
14. type:TEXT (home, office, mailing etc)

### Table (role)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. name:TEXT (reserved roles: admin, employee, member)

### Table (permission)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. name:TEXT (name for permission)

### Table (role_permission)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. service_type: TEXT (BILLING, NOTIFICATION etc)
6. role_id: UUID References role(id)
7. permission_id: UUID References permission(id)

### Table (user_role)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. user_id: UUID REFERENCES user(id)

## APIs

## Tenant APIs

### Register Tenant (Permission: TENANT_CREATE)

```jsx
POST /api/tenants

**Request**
{
    "name": "New York City Library",
    "domain": "nyclib.lm.com",
    "address": "UUID of the address",
    "currency_code": "USD",
    "country_code": "US"
}

**Response**
{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "name": "New York City Library",
    "domain": "nyclib.lm.com",
    "address": "UUID of the address",
    "currency_code": "USD",
    "country_code": "US"
}

```

### Update Tenant (Permission: TENANT_UPDATE)

```jsx
PUT /api/tenants/{id}

**Request**
{
    "name": "Updated New York City Library",
    "domain": "updated.nyclib.lm.com",
    "address": "Updated UUID of the address",
    "currency_code": "EUR",
    "country_code": "FR"
}

**Response**
{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "name": "Updated New York City Library",
    "domain": "updated.nyclib.lm.com",
    "address": "Updated UUID of the address",
    "currency_code": "EUR",
    "country_code": "FR"
}

```

### Get Tenant (Permission: TENANT_READ)

```jsx
GET /api/tenants/{id}

**Response**
{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "deleted_at": null,
    "name": "New York City Library",
    "domain": "nyclib.lm.com",
    "address": "UUID of the address",
    "currency_code": "USD",
    "country_code": "US"
}

```

### Get Tenants (Permission: TENANT_READ_ALL)

```jsx
GET /api/tenants&pageNumber=1&pageSize=10

**Response**

{
  "pageNumber": 1,
  "pageSize": 10,
  "hasNext": true,
  "hasPrev": false,
  "results": [
    {
      "id": "UUID1",
      "created_at": "2024-08-01T12:34:56Z",
      "updated_at": "2024-08-01T13:45:00Z",
      "deleted_at": null,
      "name": "New York City Library",
      "domain": "nyclib.lm.com",
      "address": "UUID of the address",
      "currency_code": "USD",
      "country_code": "US"
    },
    {
      "id": "UUID2",
      "created_at": "2024-08-02T14:34:56Z",
      "updated_at": "2024-08-02T15:45:00Z",
      "deleted_at": null,
      "name": "Los Angeles Public Library",
      "domain": "lapub.lm.com",
      "address": "UUID of the address",
      "currency_code": "USD",
      "country_code": "US"
    }
  ]
}

```

### Delete Tenant (Permission: TENANT_DELETE)

```jsx
DELETE /api/tenants/{id}

Response
{
    "id": "UUID",
    "deleted_at": "2024-08-01T14:00:00Z"
}

```

## User APIs

### Login User (Public URL)

```jsx
POST /api/tenant/{tenantId}/users/login

**Request**
{
    "tenant_id": "UUID for tenant"
    "email": "user@example.com",
    "password": "password123"
}

**Response**

{
    "access_token": "jwt-token-here",
    "refresh_token": "refresh token here"
}

```

### Register User (Permission: USER_CREATE)

```jsx
POST /api/tenant/{tenantId}/users

Request
{
    "first_name": "John",
    "last_name": "Doe",
    "email": "user@example.com",
    "password": "password123",
    "phone_number": {
        "country_code": "+1",
        "number": "1234567890"
    },
    "address": {
		    "address1": "123 Main St",
		    "address2": "Apt 4B",
		    "city": "New York",
		    "state": "NY",
		    "country": "USA",
		    "zip_code": "10001",
		    "type": "home"
		}
}

Response
{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "first_name": "John",
    "last_name": "Doe",
    "full_name": "John Doe",
    "email": "user@example.com",
    "phone_number": {
        "country_code": "+1",
        "number": "1234567890"
    },
    "address": {
		    "address1": "123 Main St",
		    "address2": "Apt 4B",
		    "city": "New York",
		    "state": "NY",
		    "country": "USA",
		    "zip_code": "10001",
		    "type": "home"
		}
}

```

### Update User (Permission: USER_UPDATE)

```jsx
PUT /api/tenant/{tenantId}/users/{id}

Request
{
    "first_name": "Jane",
    "last_name": "Doe",
    "email": "jane.doe@example.com",
    "phone_number": {
        "country_code": "+1",
        "number": "0987654321"
    },
    "address": {
		    "address1": "123 Main St",
		    "address2": "Apt 4B",
		    "city": "New York",
		    "state": "NY",
		    "country": "USA",
		    "zip_code": "10001",
		    "type": "home"
		}
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "first_name": "Jane",
    "last_name": "Doe",
    "full_name": "Jane Doe",
    "email": "jane.doe@example.com",
    "phone_number": {
        "country_code": "+1",
        "number": "0987654321"
    },
    "address": {
		    "address1": "123 Main St",
		    "address2": "Apt 4B",
		    "city": "New York",
		    "state": "NY",
		    "country": "USA",
		    "zip_code": "10001",
		    "type": "home"
		}
}

```

### Get User (Permission: USER_READ)

```jsx
GET /api/tenant/{tenantId}/users/{id}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "deleted_at": null,
    "first_name": "John",
    "last_name": "Doe",
    "full_name": "John Doe",
    "email": "user@example.com",
    "phone_number": {
        "country_code": "+1",
        "number": "1234567890"
    },
    "address": {
		    "address1": "123 Main St",
		    "address2": "Apt 4B",
		    "city": "New York",
		    "state": "NY",
		    "country": "USA",
		    "zip_code": "10001",
		    "type": "home"
		}
}

```

### Get Users (Permission: USER_READ_ALL)

```jsx
GET /api/tenant/{tenantId}/users

Response

{
  "pageNumber": 1,
  "pageSize": 10,
  "totalPages": 20,
  "hasNext": true,
  "hasPrev": false,
  "results": [
    {
      "id": "UUID1",
      "created_at": "2024-08-01T12:34:56Z",
      "updated_at": "2024-08-01T13:45:00Z",
      "deleted_at": null,
      "first_name": "John",
      "last_name": "Doe",
      "full_name": "John Doe",
      "email": "user@example.com",
      "phone_number": {
        "country_code": "+1",
        "number": "1234567890"
      },
      "address": {
		    "address1": "123 Main St",
		    "address2": "Apt 4B",
		    "city": "New York",
		    "state": "NY",
		    "country": "USA",
		    "zip_code": "10001",
		    "type": "home"
			}
    },
    {
      "id": "UUID2",
      "created_at": "2024-08-02T14:34:56Z",
      "updated_at": "2024-08-02T15:45:00Z",
      "deleted_at": null,
      "first_name": "Jane",
      "last_name": "Smith",
      "full_name": "Jane Smith",
      "email": "jane.smith@example.com",
      "phone_number": {
        "country_code": "+1",
        "number": "0987654321"
      },
      "address": {
		    "address1": "123 Main St",
		    "address2": "Apt 4B",
		    "city": "New York",
		    "state": "NY",
		    "country": "USA",
		    "zip_code": "10001",
		    "type": "home"
			}
    }
  ]
}
```

### Delete User (Permission: USER_DELETE)

```jsx
DELETE /api/tenant/{tenantId}/users/{id}

{
    "id": "UUID",
    "deleted_at": "2024-08-01T14:00:00Z"
}

```

## Role APIs

### Create Role (Permission: ROLE_CREATE)

```jsx
POST /api/tenant/{tenantId}/roles
Request

{
    "name": "admin"
}

Response
{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "name": "admin"
}

```

### Update Role (Permission: ROLE_UPDATE)

```jsx
PUT /api/tenant/{tenantId}/roles/{id}

Request

{
    "name": "employee"
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "name": "employee"
}

```

### Get Role (Permission: ROLE_READ)

```jsx
GET /api/tenant/{tenantId}/roles/{id}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "name": "admin"
}
```

### Get Roles (Permission: ROLE_READ_ALL)

```jsx

GET /api/tenant/{tenantId}/roles?pageNumber=1&pageSize=10

{
  "pageNumber": 1,
  "pageSize": 10,
  "totalPages": 20,
  "hasNext": true,
  "hasPrev": false,
  "results": [
    {
      "id": "UUID1",
      "created_at": "2024-08-01T12:34:56Z",
      "updated_at": "2024-08-01T13:45:00Z",
      "name": "admin"
    },
    {
      "id": "UUID2",
      "created_at": "2024-08-02T14:34:56Z",
      "updated_at": "2024-08-02T15:45:00Z",
      "name": "employee"
    }
  ]
}
```

### Add/Remove Permissions to/from Role (Permission: ROLE_ASSIGN_PERMISSION)

### Delete Role (Permission: ROLE_DELETE)

```jsx
DELETE /api/tenant/{tenantId}/roles/{id}

Response code 204 No Content
```

### Create Permission (Permission: PERMISSION_CREATE)

```jsx
POST /api/tenant/{tenantId}/permissions

Request

{
    "name": "READ"
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "name": "READ"
}

```

### Update Permission (Permission: PERMISSION_UPDATE)

```jsx
PUT /api/tenant/{tenantId}/permissions/{id}

Request

{
    "name": "WRITE"
}

Response

{
    "id": "UUID",
    "tenant_id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "name": "WRITE"
}
```

### Get Permission (Permission: PERMISSION_READ)

```jsx
GET /api/tenant/{tenantId}/permissions/{id}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "name": "READ"
}

```

### Get Permissions (Permission: PERMISSION_READ_ALL)

```jsx
GET /api/tenant/{tenantId}/permissions

Response

{
  "pageNumber": 1,
  "pageSize": 10,
  "totalPages": 20,
  "hasNext": true,
  "hasPrev": false,
  "results": [
    {
      "id": "UUID1",
      "created_at": "2024-08-01T12:34:56Z",
      "updated_at": "2024-08-01T13:45:00Z",
      "name": "READ"
    },
    {
      "id": "UUID2",
      "created_at": "2024-08-02T14:34:56Z",
      "updated_at": "2024-08-02T15:45:00Z",
      "name": "WRITE"
    }
  ]
}
```

### Delete Permission (Permission: PERMISSION_DELETE)

```jsx
DELETE /api/tenant/{tenantId}/permissions/{id}

Response 204 No Content
```

### Create Role Permission (Permission: ROLE_PERMISSION_CREATE)

```jsx
POST /api/tenant/{tenantId}/role_permissions

Request
{
    "service_type": "BILLING",
    "role_id": "UUID",
    "permission_id": "UUID"
}

Response 
{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "service_type": "BILLING",
    "role_id": "UUID",
    "permission_id": "UUID"
}

```

### Get Role Permissions (Permission: ROLE_PERMISSION_READ_ALL)

```jsx
GET /api/tenant/{tenantId}/roles/{id}/permissions?pageNum=1&pageSize=10

{
  "pageNumber": 1,
  "pageSize": 10,
  "totalPages": 20,
  "hasNext": true,
  "hasPrev": false,
  "results": [
    {
      "id": "UUID1",
      "created_at": "2024-08-01T12:34:56Z",
      "updated_at": "2024-08-01T13:45:00Z",
      "service_type": "BILLING",
      "permission": {
        "id": "UUID",
        "name": "USER_CREATE"
      }
    },
    {
      "id": "UUID2",
      "created_at": "2024-08-02T14:34:56Z",
      "updated_at": "2024-08-02T15:45:00Z",
      "service_type": "NOTIFICATION",
      "permission": {
        "id": "UUID",
        "name": "USER_UPDATE"
      }
    }
  ]
}

```

### Delete Role Permission (Permission: ROLE_PERMISSION_DELETE)

```jsx
DELETE /api/tenant/{tenantId}/role_permissions/{id}

Response code 204 No Content
```

# Book Management Service

### Table (book)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. deleted_at:  timestamp with timezone UTC
6. title:TEXT 
7. summary: TEXT — summary of book
8. total_pages: INTEGER
9. group_id: UUID REFERENCES group(id)
10. genre_id: UUID REFERENCES genre(id)
11. author_id: UUID REFERENCES author(id)
12. publisher_id: UUID REFERENCES publisher(id)
13. ISBN: VARCHAR(25)
14. year: Integer

### Table (book_inventory)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. deleted_at:  timestamp with timezone UTC
6. book_id: UUID REFERENCES book(id)
7. sku: TEXT (internal unique identifier)
8. reserve_price: bigint (store price in lowest money type)
9. cost_price: bigint (store price in lowest money type)
10. barcode: TEXT
11. language_id: UUID REFERENCES language(id)
12. ISBN: VARCHAR(25)
13. year: Integer
14. photo_ids: array (s3 links)
15. book_sub_location_id: UUID REFERENCES book_sub_location(id)
    1. This will be null if book is in BORROWED status

### Table (genre)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. deleted_at:  timestamp with timezone UTC
6. name: TEXT

### Table (group) e.g book series

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. deleted_at:  timestamp with timezone UTC
6. name: TEXT

Note: Series of books can be part of one group (e.g Harry Porter, Lord of the rings)

### Table (language)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. deleted_at:  timestamp with timezone UTC
6. name: TEXT
7. code: varchar(5) e.g en, fr etc

### Table (author)

1. id: UUID PRIMARY KEY 
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. deleted_at:  timestamp with timezone UTC
6. first_name:TEXT 
7. last_name: TEXT
8. full_name: TEXT (auto populated by concatenating first and last name)
9. dob: DATE 

### Table (publisher)

1. id: UUID PRIMARY KEY 
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. deleted_at:  timestamp with timezone UTC
6. name:TEXT 
7. last_name: TEXT
8. full_name: TEXT (auto populated by concatenating first and last name)
9. dob: DATE 

### Table (book_location)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. name: TEXT
6. address: jsonb

### Table (book_sub_location)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID REFERENCES tenant(id)
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. name: TEXT
6. book_location_id: UUID REFERENCES book_location(id)

## APIs

## Book APIs

### Create Book (Permission: BOOK_CREATE)

```jsx
POST /api/tenant/{tenantId}/books

Request
{
    "title": "The Great Gatsby",
    "summary": "A novel written by American author F. Scott Fitzgerald.",
    "total_pages": 218,
    "group_id": "UUID",
    "genre_id": "UUID",
    "author_id": "UUID",
    "publisher_id": "UUID",
    "ISBN": "9780743273565",
    "year": 1925
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "title": "The Great Gatsby",
    "summary": "A novel written by American author F. Scott Fitzgerald.",
    "total_pages": 218,
    "group_id": "UUID",
    "genre_id": "UUID",
    "author_id": "UUID",
    "publisher_id": "UUID",
    "ISBN": "9780743273565",
    "year": 1925
}

```

### Update Book (Permission: BOOK_UPDATE)

```jsx
PUT /api/tenant/{tenantId}/books/{id}

Request

{
    "title": "The Great Gatsby (Updated)",
    "summary": "A novel written by American author F. Scott Fitzgerald. (Updated)",
    "total_pages": 220,
    "group_id": "New UUID",
    "genre_id": "New UUID",
    "author_id": "New UUID",
    "publisher_id": "New UUID",
    "ISBN": "9780743273566",
    "year": 1926
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "title": "The Great Gatsby (Updated)",
    "summary": "A novel written by American author F. Scott Fitzgerald. (Updated)",
    "total_pages": 220,
    "group_id": "New UUID",
    "genre_id": "New UUID",
    "author_id": "New UUID",
    "publisher_id": "New UUID",
    "ISBN": "9780743273566",
    "year": 1926
}

```

### Get Book (Permission: BOOK_READ)

```jsx
GET /api/tenant/{tenantId}/books/{id}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "deleted_at": null,
    "title": "The Great Gatsby",
    "summary": "A novel written by American author F. Scott Fitzgerald.",
    "total_pages": 218,
    "group_id": "UUID",
    "genre_id": "UUID",
    "author_id": "UUID",
    "publisher_id": "UUID",
    "ISBN": "9780743273565",
    "year": 1925
}

```

### Get Books (Permission: BOOK_READ_ALL)

1. Various filters (e.g filter by status etc)

```jsx
GET /api/tenant/{tenantId}/books?pageNumber=1&pageSize=10&status="AVAILABLE"&genre_id="UUID"
Available filters
1. status
2. genre_id
3. author_id
4. publisher_id
5. title
6. isbn
7. year

{
  "pageNumber": 1,
  "pageSize": 10,
  "totalPages": 20,
  "hasNext": true,
  "hasPrev": false,
  "results": [
    {
      "id": "UUID1",
      "created_at": "2024-08-01T12:34:56Z",
      "updated_at": "2024-08-01T13:45:00Z",
      "deleted_at": null,
      "title": "The Great Gatsby",
      "summary": "A novel written by American author F. Scott Fitzgerald.",
      "total_pages": 218,
      "group_id": "UUID",
      "genre_id": "UUID",
      "author_id": "UUID",
      "publisher_id": "UUID",
      "ISBN": "9780743273565",
      "year": 1925
    },
    {
      "id": "UUID2",
      "created_at": "2024-08-02T14:34:56Z",
      "updated_at": "2024-08-02T15:45:00Z",
      "deleted_at": null,
      "title": "To Kill a Mockingbird",
      "summary": "A novel by Harper Lee published in 1960.",
      "total_pages": 281,
      "group_id": "UUID",
      "genre_id": "UUID",
      "author_id": "UUID",
      "publisher_id": "UUID",
      "ISBN": "9780060935467",
      "year": 1960
    }
  ]
}
```

### Delete Book (Permission: BOOK_DELETE)

```jsx
DELETE /api/tenant/{tenantId}/books/{id}

Response 

{
    "id": "UUID",
    "deleted_at": "2024-08-01T14:00:00Z"
}

```

## Genre APIs

### Create Genre (Permission: GENRE_CREATE)

```jsx
POST /api/tenant/{tenantId}/genres

Request

{
    "name": "Fiction"
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "name": "Fiction"
}

```

### Update Genre (Permission: GENRE_UPDATE)

```jsx
PUT /api/tenant/{tenantId}/genres/{id}

Request
{
    "name": "Historical Fiction"
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "name": "Historical Fiction"
}

```

### Get Genre (Permission: GENRE_READ)

```jsx
GET /api/tenant/{tenantId}/genres/{id}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "deleted_at": null,
    "name": "Fiction"
}

```

### Get Genres (Permission: GENRE_READ_ALL)

```jsx
GET /api/tenant/{tenantId}/genres&name='Fiction'
Available filters
1. name

Response

{
  "pageNumber": 1,
  "pageSize": 10,
  "totalPages": 20,
  "hasNext": true,
  "hasPrev": false,
  "results": [
    {
      "id": "UUID1",
      "created_at": "2024-08-01T12:34:56Z",
      "updated_at": "2024-08-01T13:45:00Z",
      "deleted_at": null,
      "name": "Fiction"
    },
    {
      "id": "UUID2",
      "created_at": "2024-08-02T14:34:56Z",
      "updated_at": "2024-08-02T15:45:00Z",
      "deleted_at": null,
      "name": "Non-Fiction"
    }
  ]
}
```

### Delete Genre (Permission: GENRE_DELETE)

```jsx
DELETE /api/tenant/{tenantId}/genres/{id}

{
    "id": "UUID",
    "deleted_at": "2024-08-01T14:00:00Z"
}

```

## Group APIs

### Create Group (Permission: GROUP_CREATE)

```jsx
POST /api/tenant/{tenantId}/groups

Request
{
    "name": "Harry Potter Series"
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "name": "Harry Potter Series"
}

```

### Update Group (Permission: GROUP_UPDATE)

```jsx
PUT /api/tenant/{tenantId}/groups/{id}

Request
{
    "name": "Harry Potter Series (Updated)"
}

Response
{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "name": "Harry Potter Series (Updated)"
}

```

### Get Group (Permission: GROUP_READ)

```jsx
GET /api/tenant/{tenantId}/groups/{id}

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:45:00Z",
    "deleted_at": null,
    "name": "Harry Potter Series"
}

```

### Get Groups (Permission: GROUP_READ_ALL)

```jsx
GET /api/tenant/{tenantId}/groups?pageNumber=1&pageSize=10&name="Harry"

Response

{
  "pageNumber": 1,
  "pageSize": 10,
  "totalPages": 20,
  "hasNext": true,
  "hasPrev": false,
  "results": [
    {
      "id": "UUID1",
      "created_at": "2024-08-01T12:34:56Z",
      "updated_at": "2024-08-01T13:45:00Z",
      "deleted_at": null,
      "name": "Harry Potter Series"
    },
    {
      "id": "UUID2",
      "created_at": "2024-08-02T14:34:56Z",
      "updated_at": "2024-08-02T15:45:00Z",
      "deleted_at": null,
      "name": "Lord of the Rings Series"
    }
  ]
}
```

### Delete Group (Permission: GROUP_DELETE)

```jsx
DELETE /api/tenant/{tenantId}/groups/{id}

{
    "id": "UUID",
    "deleted_at": "2024-08-01T14:00:00Z"
}

```

## Author APIs

### Create Author (Permission: AUTHOR_CREATE)

### Update Author (Permission: AUTHOR_UPDATE)

### Get Author (Permission: AUTHOR_READ)

### Get Authors (Permission: AUTHOR_READ_ALL)

### Delete Author (Permission: AUTHOR_DELETE)

## Publisher APIs

### Create Publisher (Permission: PUBLISHER_CREATE)

### Update Publisher (Permission: PUBLISHER_UPDATE)

### Get Publisher (Permission: PUBLISHER_READ)

### Get Publishers (Permission: PUBLISHER_READ_ALL)

### Delete Publisher (Permission: PUBLISHER_DELETE)

## Book Location APIs

### Create Book Location (Permission: LOCATION_CREATE)

### Update Book Location (Permission: LOCATION_UPDATE)

### Get Book Location (Permission: LOCATION_READ)

### Get Book Locations (Permission: LOCATION_READ_ALL)

### Delete Book Location (Permission: LOCATION_DELETE)

## Book Sub Location APIs

### Create Book Sub Location (Permission: SUB_LOCATION_CREATE)

### Update Book Sub Location (Permission: SUB_LOCATION_UPDATE)

### Get Book Sub Location (Permission: SUB_LOCATION_READ)

### Get Book Sub Locations (Permission: SUB_LOCATION_READ_ALL)

### Delete Book Sub Location (Permission: SUB_LOCATION_DELETE)

# Member Service

1. Register member

### Table (member)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. user_id: UUID
4. created_at: timestamp with timezone UTC
5. updated_at: timestamp with timezone UTC
6. is_active: boolean
7. membership_start_time: timestamp with timezone UTC
8. membership_end_time: timestamp with timezone UTC

## APIs

## Member APIs

### Create Member (Permission: MEMBER_CREATE)

### Update Member (Permission: MEMBER_UPDATE)

### Get Member (Permission: MEMBER_READ)

### Get Members (Permission: MEMBER_READ_ALL)

### Activate/Deactivate Member (Permission: MEMBER_ACTIVATE_DEACTIVATE)

### Renew Membership (Permission: MEMBER_RENEW_MEMBERSHIP)

## Member Reservation and Borrowing APIs

### Get Member Book Reservations (Permission: MEMBER_BOOK_RESERVATION)

### Get Member Book Reservation by reservation ID (Permission: MEMBER_BOOK_RESERVATION)

### Get Member Book borrowings (Permission: MEMBER_BOOK_BORROWING)

### Get Member Book borrowing by borrowing ID (Permission: MEMBER_BOOK_BORROWING)

# Book Lending Service

1. Book inventory Cache
2. Book Reservation
3. Book Return
4. Book Borrowing

### Table (book_inventory_cache) (will be populate async using cdc)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. book_inventory_id: UUID
4. status: TEXT (e.g AVAILABLE, BORROWED, MISSING, OVERDUE, LOST .These will be enums in coding language)
5. book_ticket_item_id: UUID (ticket item id corresponding to which this book is borrowed)
6. extended_due_time: timestamp with timezone UTC
7. deleted_at: timestamp with timezone UTC

### Table (book_reservation)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. book_inventory_id: UUID REFERENCES book_inventory(id)
6. member_id: UUID
7. reserve_price: bigint (store price in lowest money type)
    1. since book_inventory price can change for future reservation. This is price at the time of reservation
    2. We will only charge this price after borrow
8. status: TEXT (ACTIVE, CANCELED, FULFILLED)
9. reserve_from_time: timestamp with timezone UTC
10. reservation_to_time: timestamp with timezone UTC

Note: 

1. Priority will be given based on reserve_from_time sorted ascending
2. One can borrow book without reservation if its not already reserved for that timeframe
3. Reservation will be automatically canceled if not borrowed in certain period
4. We will only allow reserving book 1 month in advance (configurable)

### Table (borrow_book_ticket)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. member_id: UUID
6. book_borrow_id: UUID REFERENCES borrow_book(id)
7. total_price: bigint (store price in lowest money type)
8. status: TEXT (IN_PROGRESS, PAID, VOID, PAYMENT_FAILED)
9. payment_id: UUID (will come from billing service)

### Table (borrow_book_ticket_item)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. book_inventory_id: UUID 
6. member_id: UUID
7. borrow_price: bigint (store price in lowest money type)
8. borrow_time: timestamp with timezone UTC
9. return_time: timestamp with timezone UTC
10. original_due_time: timestamp with timezone UTC
11. extended_due_time: timestamp with timezone UTC (initially same as original_due_time)
12. status: TEXT (BORROWED, RETURN, OVERDUE)
13. invoice_id: UUID (this will come from billing service)

### Table (task)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. status: TEXT (FAILED, IN_PROGRESS, SUCCESS)
6. name: TEXT

## APIs

## Book Reservation APIs

### Reserve Book (Permission: LENDING_BOOK_RESERVATION)

**Validation**

- **Reserve Book:**
    - Check if the reservation is within the allowed advance booking period (1 month or configurable).
    - Ensure that the book is not already reserved for the specified timeframe.
    - Implement logic to automatically cancel reservations if not borrowed within a specified period.
    - Same day book reservation not allowed.

```jsx
POST /api/tenant/{tenant_id}/lending/member/reservations (member api)

Request

{
    "book_inventory_id": "UUID",
    "reserve_from_time": "2024-09-01T10:00:00Z",
    "reservation_to_time": "2024-09-15T10:00:00Z"
}

Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "book_inventory_id": "UUID",
    "member_id": "UUID",
    "reserve_price": 500,
    "status": "ACTIVE",
    "reserve_from_time": "2024-09-01T10:00:00Z",
    "reservation_to_time": "2024-09-15T10:00:00Z"
}

POST /api/tenant/{tenant_id}/lending/reservations (admin version)

Request
{
    "book_inventory_id": "UUID",
    "member_id": "UUID",
    "reserve_from_time": "2024-09-01T10:00:00Z",
    "reservation_to_time": "2024-09-15T10:00:00Z"
}
Response

{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "book_inventory_id": "UUID",
    "member_id": "UUID",
    "reserve_price": 500,
    "status": "ACTIVE",
    "reserve_from_time": "2024-09-01T10:00:00Z",
    "reservation_to_time": "2024-09-15T10:00:00Z"
}

```

### Cancel Reservation (Permission: LENDING_BOOK_CANCEL_RESERVATION)

- **Cancel Reservation:**
    - Only allow cancellation of reservations that are currently active.
    - Update the status to "CANCELED" and record the updated timestamp.

```jsx
DELETE /api/tenant/{tenant_id}/lending/reservations/{id}

Response
{
    "id": "UUID",
    "tenant_id": "UUID",
    "status": "CANCELED",
    "updated_at": "2024-08-01T14:00:00Z"
}

```

## Book Borrowing APIs

### Create Borrow Book Ticket (Permission: LENDING_BOOK_BORROW)

Validation

- Check if there is existing open ticket, if so then return that

```jsx
POST /api/tenant/{tenant_id}/lending/borrow_book_tickets

Request
{
    "member_id": "UUID",
}

Response
{
    "id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:34:56Z",
    "member_id": "UUID",
    "total_price": 0,
    "currency_code": "USD"
    "status": "IN_PROGRESS",
    "payment_id": null
}
```

### Add Books to Borrow Book Ticket (Permission: LENDING_BOOK_BORROW)

Validation

- Check if book is not reserved during this time
- Book can be borrowed without reservation
- validate book has available status

```jsx
POST /api/tenant/{tenant_id}/lending/borrow_book_tickets/{ticket_id}/items

{
    "book_inventory_id": "UUID",
    "due_time": "2024-08-15T10:00:00Z",
}

Response
{
    "id": "UUID",
    "created_at": "2024-08-01T12:35:00Z",
    "updated_at": "2024-08-01T12:35:00Z",
    "book_inventory_id": "UUID",
    "member_id": "UUID",
    "borrow_price": 500,
    "borrow_time": "2024-08-01T10:00:00Z",
    "return_time": null,
    "due_time": "2024-08-15T10:00:00Z",
    "status": "BORROWED",
    "invoice_id": null,
    "ticket_id": "UUID"
}
```

### Get Borrow Book Ticket (Permission: LENDING_BOOK_BORROW)

```jsx
GET /api/tenant/{tenant_id}/lending/borrow_book_tickets/{ticket_id}

Response
{
    "id": "UUID",
    "tenant_id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:40:00Z",
    "member_id": "UUID",
    "total_price": 1000,
    "status": "PAID",
    "payment_id": "payment_id_from_billing_service",
    "items": [
        {
            "id": "UUID1",
            "tenant_id": "UUID",
            "created_at": "2024-08-01T12:35:00Z",
            "updated_at": "2024-08-01T12:35:00Z",
            "book_inventory_id": "UUID1",
            "member_id": "UUID",
            "borrow_price": 500,
            "borrow_time": "2024-08-01T10:00:00Z",
            "return_time": null,
            "due_time": "2024-08-15T10:00:00Z",
            "status": "BORROWED",
            "invoice_id": "UUID1"
        },
        {
            "id": "UUID2",
            "tenant_id": "UUID",
            "created_at": "2024-08-01T12:36:00Z",
            "updated_at": "2024-08-01T12:36:00Z",
            "book_inventory_id": "UUID2",
            "member_id": "UUID",
            "borrow_price": 500,
            "borrow_time": "2024-08-01T10:00:00Z",
            "return_time": null,
            "due_time": "2024-08-15T10:00:00Z",
            "status": "BORROWED",
            "invoice_id": "UUID2"
        }
    ]
}

```

### Make payment for Borrow book ticket (Permission: LENDING_BOOK_RETURN)

```jsx
POST /api/tenant/{tenant_id}/lending/borrow_book_tickets/{ticket_id}/make_payment
We will poll on status to verify if payment is done

Response 
{
    "id": "UUID",
    "tenant_id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T12:40:00Z",
    "member_id": "UUID",
    "total_price": 1000,
    "status": "IN_PROGRESS",
    "payment_id": "payment_id_from_billing_service",
    "items": [
        {
            "id": "UUID1",
            "tenant_id": "UUID",
            "created_at": "2024-08-01T12:35:00Z",
            "updated_at": "2024-08-01T12:35:00Z",
            "book_inventory_id": "UUID1",
            "member_id": "UUID",
            "borrow_price": 500,
            "borrow_time": "2024-08-01T10:00:00Z",
            "return_time": null,
            "due_time": "2024-08-15T10:00:00Z",
            "status": "BORROWED",
            "invoice_id": "UUID1"
        },
        {
            "id": "UUID2",
            "tenant_id": "UUID",
            "created_at": "2024-08-01T12:36:00Z",
            "updated_at": "2024-08-01T12:36:00Z",
            "book_inventory_id": "UUID2",
            "member_id": "UUID",
            "borrow_price": 500,
            "borrow_time": "2024-08-01T10:00:00Z",
            "return_time": null,
            "due_time": "2024-08-15T10:00:00Z",
            "status": "BORROWED",
            "invoice_id": "UUID2"
        }
    ]
}
```

### Return Book (Permission: LENDING_BOOK_RETURN)

```jsx
POST /api/tenant/{tenant_id}/lending/books/{book_inventory_id}/return

Response 204 No content

```

### Extend Book Borrowing period (Permission: LENDING_BOOK_BORROW)

- Extension will not be allowed if there upcoming reservation

```jsx
POST /api/tenant/{tenant_id}/lending/books/{book_inventory_id}/extend

Request
{
   "to_time": "2024-08-01T12:35:00Z"
}

Response 204 No content

```

### Get Task (Permission: LENDING_TASK)

```jsx
GET /api/tenant/{tenant_id}/lending/tasks/{id}

Response
{
    "id": "UUID",
    "tenant_id": "UUID",
    "created_at": "2024-08-01T12:34:56Z",
    "updated_at": "2024-08-01T13:00:00Z",
    "status": "IN_PROGRESS",
    "name": "Sample Task"
}

```

### Background Jobs

1. Mark books overdue if they are outside lending period and send notification
2.  Send notifications for which due dates are nearby
3. Auto extend book by one day if overdue 
    1. Create invoice
    2. Charge payment
    3. send notification
4. Send notification for failed payments

# Search  Service

API for these will hit elastic search

## APIs

1. Search book by name
2. Filter book by availability status for given time range
3. API to view when book is available (i.e not reserved)
4. API to perform CRUD in elastic search (will be used by workflows)

```jsx
POST /api/tenant/{tenant_id}/books/search

{
  "name": {
    "operator": "ilike",
    "value": "Harry porter"
  },
  "status": {
    "operator": "eq",
    "value": "available"
  },
  "author": {
    "operator": "ilike",
    "value": "Jon Doe"
  }
}
```

# Billing Service

1. Charge borrowing fees
2. Charge late fee
3. Get all payment history for member
4. Get fee corresponding to borrow id

### Table (invoice)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. name: TEXT (autogenerated unique invoice number)
6. user_id: UUID
7. total_price: bigint (store price in lowest money type)
8. status: TEXT (DRAFT, PAID, PAYMENT_FAILED)
9. payment_id: UUID REFERENCES payment(id)

### Table (invoice_item)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. invoice_id: UUID REFERENCES invoice(id)
6. entity_id: UUID (Generic id, in this case this will be book_inventory_id)
7. member_id: UUID **(This should not be here)**
8. price: bigint (store price in lowest money type)
9. type: TEXT (e.g REGULAR_FEE, LATE_FEE, OTHER_FEE)
10. notes: TEXT (Notes corresponding to type of fee if any)
11. status: TEXT (PAID or UNPAID)

### Table (payment_methods)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. user_id: UUID
6. external_payment_provider_customer_id: TEXT
7. external_payment_method_id: TEXT

### Table (payment)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. type: TEXT (CREDIT_CARD, CASH, DEBIT_CARD)
6. price: bigint (store price in lowest money type)
7. tax: bigint (store price in lowest money type)
8. total_price: bigint (store price in lowest money type)
9. external_payment_id: TEXT (ID corresponding to external payment gateway e.g stripe)

## APIs

- This contains normal crud APIs

### Create Invoice (Permission: BILLING_INVOICE_CREATE)

### Update Invoice (Permission: BILLING_INVOICE_UPDATE)

### Get Invoice (Permission: BILLING_INVOICE_READ)

### Get Invoices (Permission: BILLING_INVOICE_READ_ALL)

1. This will have various filters

### Delete Invoice (Permission: BILLING_INVOICE_DELETE)

# Config Service

Used for various configuration across microservices

### Table (config)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. service_type: TEXT (e.g BILLING, BOOK, BOOK_RESERVATION, LENDING, ANALYTICS etc)
6. key: TEXT
7. argument: jsonb
8. enabled: boolean

### APIs

- This contains normal crud APIs

# Notification Service

### Table (subscription)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. type: array(string) (EMAIL, TEXT, WEB, MOBILE)
6. user_id: UUID

### Table (notification_history)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. user_id: UUID
6. type: TEXT (EMAIL, TEXT, WEB, MOBILE)
7. message: TEXT

### APIs

- This contains normal crud APIs

# Workflow Service

1. List all workflows
2. Workflow contains list of actions in some order
3. We need to register actions
4. action registration should know when that action can be triggered
    1. can action start on their own with any other input e.g get all books (this can be triggered anytime irrespective of where it is in flow)
    2. validating action input
    3. keeping track of action output
5. Custom output per action based on condition e.g if value > 0 then return APPROVE

### Table (workflow)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. definition: JSONB NOT NULL,
6. input_output_schema: jsonb
7. name: TEXT
8. description: TEXT
9. UNIQUE(tenant_id, name)

### Table (permission)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. name: TEXT 
6. UNIQUE(tenant_id, name)

### Table (workflow_permission)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. workflow_id: UUID REFERENCES workflow(id)
6. permission_id: UUID REFERENCES permission(id)

### Table (workflow_user_permission)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. user_id: UUID 
6. permission_id: UUID REFERENCES permission(id)

### Table (action)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. name: TEXT 
6. description: TEXT
7. function_unique_id: UUID — This will be defined in function schema json
8. function_name TEXT NOT NULL
9. schema: jsonb
    1. We will have schema files for each function which will generate code and on upgrade this will get updated.
    2. Function unique id will be defined in file.
10. UNIQUE(tenant_id, function_unique_id)

### Table (workflow_trigger)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. event_id: UUID REFENCES event(id)
6. workflow_id: UUID

### Table (event)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. name: TEXT

### Table (workflow_execution_history)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. workflow_id: UUID REFERENCES workflow(id)
6. user_id: UUID
7. input_parameters: jsonb — input values will be persisted for each workflow 
8. output_result: jsonb — result will be persisted for each workflow

### Table (action_execution_history)

1. id: UUID PRIMARY KEY
2. tenant_id: UUID
3. created_at: timestamp with timezone UTC
4. updated_at: timestamp with timezone UTC
5. workflow_id: UUID REFERENCES workflow(id)
6. action_id: UUID REFERENCES action(id)
7. user_id: UUID
8. input_parameters: jsonb — input values will be persisted for each action 
9. output_result: jsonb — result will be persisted for each action

### **Action Schema files**

- parametersSchema: json schema , this will help us validate parameter easily
- returnTypeSchema: json schema , this will help us validate parameter easily

```jsx
File getAllUsersAction.json

{
      "id": "1",
      "name": "Get All Users",
      "description": "Fetch all users",
      "functionName": "getAllUsers",
      "parametersSchema": [],
      "returnType": "List<User>"
      "returnTypeSchema": {
				  "type": "array",
				  "items": {
				    "type": "object",
				    "properties": {
				      "id": {
				        "type": "string"
				      },
				      "first_name": {
				        "type": "string"
				      },
				      "last_name": {
				        "type": "string"
				      }
				    },
				    "required": [
				      "id",
				      "first_name"
				    ]
				  }
			}

}

File getOverdueBooksForUserAction.json

{
      "id": "2",
      "name": "Get Overdue Books for User",
      "description": "Fetch overdue books for a user",
      "functionName": "getOverdueBooksForUser",
      "parametersSchema": [
        {"name": "userId", "type": "UUID"}
      ],
      "returnType": "List<Book>"
      "returnTypeSchema": {
				  "type": "array",
				  "items": {
				    "type": "object",
				    "properties": {
				      "id": {
				        "type": "string"
				      },
				      "name": {
				        "type": "string"
				      },
				      "author_id": {
				        "type": "string"
				      }
				    },
				    "required": [
				      "id",
				      "first_name"
				    ]
				  }
			}
    }
```

Using above schema files we will auto generate java code e.g 

**ActionFactory.java**

```jsx
public interface ActionFactory {

    @Action(name = "Get All Users", description = "Fetch all users", parameterTypes = {}, returnType = List.class)
    public List<User> getAllUsers();
    
    @Action(name = "Get Overdue Books for User", description = "Fetch overdue books for a user", parameterTypes = {String.class}, returnType = List.class)
    public List<Book> getOverdueBooksForUser(String userId);
```

- We can now implement each action in `ActionFactoryImpl`
- This way we can maintain consistency in db and code regarding actions
- 

## Workflow definition

For workflow definition we can use modified aws step function state machine schema

- [https://states-language.net/spec.html](https://states-language.net/spec.html#pass-state)
- [https://aws-step-functions-data-science-sdk.readthedocs.io/en/stable/choicerules.html](https://aws-step-functions-data-science-sdk.readthedocs.io/en/stable/choicerules.html)

This will allow us to not re-invent the wheel

### **Definition using internal actions directly**

```jsx
{
  "name": "Notify Users of Overdue Books",
  "description": "A workflow to notify users of overdue books",
  "startAt": "GetAllUsers",
  "states": {
    "GetAllUsers": {
      "type": "Task",
      "method": "getAllUsers",
      "next": "ForEachUser"
    },
    "ForEachUser": {
      "type": "Map",
      "itemsPath": "$.users",
      "iterator": {
        "startAt": "GetOverdueBooksForUser",
        "states": {
          "GetOverdueBooksForUser": {
            "type": "Task",
            "method": "getOverdueBooksForUser",
            "next": "CheckEmail"
          },
          "CheckEmail": {
            "type": "Choice",
            "choices": [
              {
                "variable": "$.user.email",
                "stringEquals": "",
                "next": "SendTextNotification"
              }
            ],
            "default": "SendEmailNotification"
          },
          "SendEmailNotification": {
            "type": "Task",
            "method": "sendEmailNotification",
            "end": true
          },
          "SendTextNotification": {
            "type": "Task",
            "method": "sendTextNotification",
            "end": true
          }
        }
      }
    }
  }
}

```

### **Definition using api**

```jsx
{
  "Comment": "A workflow to notify users of overdue books",
  "StartAt": "GetAllUsers",
  "States": {
    "GetAllUsers": {
      "Type": "Task",
      "Resource": "arn:aws:states:::http:invoke",
      "Parameters": {
        "ApiEndpoint": "https://library-management.com/getAllUsers",
        "Method": "GET",
        "Headers": {
          "Content-Type": "application/json"
        }
      },
      "Next": "ForEachUser"
    },
    "ForEachUser": {
      "Type": "Map",
      "ItemsPath": "$.users",
      "Iterator": {
        "StartAt": "GetOverdueBooksForUser",
        "States": {
          "GetOverdueBooksForUser": {
            "Type": "Task",
            "Resource": "arn:aws:states:::http:invoke",
            "Parameters": {
              "ApiEndpoint": "https://library-management.com/getOverdueBooksForUser",
              "Method": "POST",
              "Headers": {
                "Content-Type": "application/json"
              },
              "Body": {
                "userId.$": "$.id"
              }
            },
            "Next": "CheckEmail"
          },
          "CheckEmail": {
            "Type": "Choice",
            "Choices": [
              {
                "Variable": "$.user.email",
                "StringEquals": "",
                "Next": "SendTextNotification"
              }
            ],
            "Default": "SendEmailNotification"
          },
          "SendEmailNotification": {
            "Type": "Task",
            "Resource": "arn:aws:states:::http:invoke",
            "Parameters": {
              "ApiEndpoint": "https://library-management.com/sendEmailNotification",
              "Method": "POST",
              "Headers": {
                "Content-Type": "application/json"
              },
              "Body": {
                "email.$": "$.user.email",
                "overdueBooks.$": "$.overdueBooks"
              }
            },
            "End": true
          },
          "SendTextNotification": {
            "Type": "Task",
            "Resource": "arn:aws:states:::http:invoke",
            "Parameters": {
              "ApiEndpoint": "https://library-management.com/sendTextNotification",
              "Method": "POST",
              "Headers": {
                "Content-Type": "application/json"
              },
              "Body": {
                "phoneNumber.$": "$.user.phone_number",
                "overdueBooks.$": "$.overdueBooks"
              }
            },
            "End": true
          }
        }
      },
      "End": true
    }
  }
}

```

### **Definition with error and custom condition**

```jsx
{
  "Comment": "A workflow to notify users of overdue books",
  "StartAt": "GetAllUsers",
  "States": {
    "GetAllUsers": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:dynamodb:scan",
      "Parameters": {
        "TableName": "UsersTable"
      },
      "TimeoutSeconds": 60,
      "Retry": [
        {
          "ErrorEquals": ["States.Timeout"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2.0
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["States.ALL"],
          "Next": "HandleGetAllUsersError"
        }
      ],
      "Next": "ForEachUser"
    },
    "HandleGetAllUsersError": {
      "Type": "Fail",
      "Error": "GetAllUsersFailed",
      "Cause": "Failed to get all users from the database."
    },
    "ForEachUser": {
      "Type": "Map",
      "ItemsPath": "$.Items",
      "Iterator": {
        "StartAt": "GetOverdueBooksForUser",
        "States": {
          "GetOverdueBooksForUser": {
            "Type": "Task",
            "Resource": "arn:aws:states:::aws-sdk:dynamodb:query",
            "Parameters": {
              "TableName": "BooksTable",
              "KeyConditionExpression": "UserId = :userId",
              "ExpressionAttributeValues": {
                ":userId": {
                  "S.$": "$.id"
                }
              }
            },
            "TimeoutSeconds": 30,
            "Retry": [
              {
                "ErrorEquals": ["States.Timeout"],
                "IntervalSeconds": 2,
                "MaxAttempts": 3,
                "BackoffRate": 2.0
              }
            ],
            "Catch": [
              {
                "ErrorEquals": ["States.ALL"],
                "Next": "HandleGetOverdueBooksError"
              }
            ],
            "Next": "CheckOverdueDuration"
          },
          "HandleGetOverdueBooksError": {
            "Type": "Fail",
            "Error": "GetOverdueBooksFailed",
            "Cause": "Failed to get overdue books for the user."
          },
          "CheckOverdueDuration": {
            "Type": "Choice",
            "Choices": [
              {
                "Variable": "$.dueDate",
                "TimestampGreaterThan": 34543534545,
                "Next": "CheckEmail"
              }
            ],
            "Default": "SkipNotification"
          },
          "CheckEmail": {
            "Type": "Choice",
            "Choices": [
              {
                "Variable": "$.email",
                "IsPresent": false,
                "Next": "SendTextNotification"
              }
            ],
            "Default": "SendEmailNotification"
          },
          "SendEmailNotification": {
            "Type": "Task",
            "Resource": "arn:aws:states:::sns:publish",
            "Parameters": {
              "Message.$": "Hello, you have overdue books: $.overdueBooks",
              "TopicArn": "arn:aws:sns:us-east-1:123456789012:NotifyUser"
            },
            "End": true
          },
          "SendTextNotification": {
            "Type": "Task",
            "Resource": "arn:aws:states:::sns:publish",
            "Parameters": {
              "Message.$": "Hello, you have overdue books: $.overdueBooks",
              "PhoneNumber.$": "$.phoneNumber"
            },
            "End": true
          },
          "SkipNotification": {
            "Type": "Pass",
            "End": true
          }
        }
      },
      "End": true
    }
  }
}

```

### **Definition for connecting Multiple workflows**

- There will be time when user wants to create atomic workflows for future reuse.
- Here we are connecting multiple workflow and creating combined workflow definition

Sub-Workflow 1: GetAllUsers

```jsx
{
  "Comment": "Sub-Workflow to get all users",
  "StartAt": "GetAllUsersTask",
  "States": {
    "GetAllUsersTask": {
      "Type": "Task",
      "method": "getAllUsers",
      "End": true
    }
  }
}

```

Sub-Workflow 2: GetOverdueBooksForUser

```jsx
{
  "Comment": "Sub-Workflow to get overdue books for a user",
  "StartAt": "GetOverdueBooksForUserTask",
  "States": {
    "GetOverdueBooksForUserTask": {
      "Type": "Task",
      "method": "getOverdueBooksForUser",
      "End": true
    }
  }
}

```

Sub-Workflow 3: SendNotification

```jsx
{
  "Comment": "Sub-Workflow to send a notification",
  "StartAt": "CheckOverdueDuration",
  "States": {
    "CheckOverdueDuration": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.overdueDuration",
          "NumericGreaterThan": 24,
          "Next": "CheckEmail"
        }
      ],
      "Default": "SkipNotification"
    },
    "CheckEmail": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.email",
          "IsPresent": false,
          "Next": "SendTextNotification"
        }
      ],
      "Default": "SendEmailNotification"
    },
    "SendEmailNotification": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "Message.$": "Hello, you have overdue books: $.overdueBooks",
        "TopicArn": "arn:aws:sns:us-east-1:123456789012:NotifyUser"
      },
      "End": true
    },
    "SendTextNotification": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "Message.$": "Hello, you have overdue books: $.overdueBooks",
        "PhoneNumber.$": "$.phoneNumber"
      },
      "End": true
    },
    "SkipNotification": {
      "Type": "Pass",
      "End": true
    }
  }
}

```

Main Workflow

```jsx
{
  "Comment": "Main Workflow to notify users of overdue books",
  "StartAt": "ExecuteGetAllUsersWorkflow",
  "States": {
    "ExecuteGetAllUsersWorkflow": {
      "Type": "Task",
      "Resource": "arn:aws:states:::states:startExecution.sync",
      "Parameters": {
        "StateMachineArn": "GetAllUsersWorkflowID",
        "Input": {
          "ExecutionId.$": "$$.Execution.Id"
        }
      },
      "Next": "ForEachUser"
    },
    "ForEachUser": {
      "Type": "Map",
      "ItemsPath": "$.Users",
      "Iterator": {
        "StartAt": "ExecuteGetOverdueBooksForUserWorkflow",
        "States": {
          "ExecuteGetOverdueBooksForUserWorkflow": {
            "Type": "Task",
            "Resource": "arn:aws:states:::states:startExecution.sync",
            "Parameters": {
              "StateMachineArn": "GetOverdueBooksForUserWorkflowId",
              "Input": {
                "UserId.$": "$.id"
              }
            },
            "Next": "ExecuteSendNotificationWorkflow"
          },
          "ExecuteSendNotificationWorkflow": {
            "Type": "Task",
            "Resource": "arn:aws:states:::states:startExecution.sync",
            "Parameters": {
              "StateMachineArn": "SendNotificationWorkflowId",
              "Input": {
                "User.$": "$"
              }
            },
            "End": true
          }
        }
      },
      "End": true
    }
  }
}

```

**List of various types**

- Task
- Parallel
- Map
- Pass
- Wait
- Choice
- Succeed
- Fail
- External - In case workflow wants to call external API directly e.g some aws lambda

**Implementation**

- validate workflow definition
- We have to create framework to execute each steps based on above workflow definition
- For each type we can have switch clause and execute either internal methods or apis
- Each action can be temporal workflow internally

## APIs

## Workflow Create (Permission: WORKFLOW_CREATE)

```jsx
POST /api/tenant/{tenant_id}/workflows

Request

{
  "name": "Notify Users of Overdue Books",
  "description": "A workflow to notify users of overdue books",
  "definition": {
    "StartAt": "GetAllUsers",
    "States": {
      "GetAllUsers": {
        "Type": "Task",
        "Resource": "arn:aws:states:::http:invoke",
        "Parameters": {
          "ApiEndpoint": "https://your-api.example.com/getAllUsers",
          "Method": "GET",
          "Headers": {
            "Content-Type": "application/json"
          }
        },
        "Next": "ForEachUser"
      },
      "ForEachUser": {
        "Type": "Map",
        "ItemsPath": "$.users",
        "Iterator": {
          "StartAt": "GetOverdueBooksForUser",
          "States": {
            "GetOverdueBooksForUser": {
              "Type": "Task",
              "Resource": "arn:aws:states:::http:invoke",
              "Parameters": {
                "ApiEndpoint": "https://your-api.example.com/getOverdueBooksForUser",
                "Method": "POST",
                "Headers": {
                  "Content-Type": "application/json"
                },
                "Body": {
                  "userId.$": "$.id"
                }
              },
              "Next": "CheckEmail"
            },
            "CheckEmail": {
              "Type": "Choice",
              "Choices": [
                {
                  "Variable": "$.user.email",
                  "StringEquals": "",
                  "Next": "SendTextNotification"
                }
              ],
              "Default": "SendEmailNotification"
            },
            "SendEmailNotification": {
              "Type": "Task",
              "Resource": "arn:aws:states:::http:invoke",
              "Parameters": {
                "ApiEndpoint": "https://your-api.example.com/sendEmailNotification",
                "Method": "POST",
                "Headers": {
                  "Content-Type": "application/json"
                },
                "Body": {
                  "email.$": "$.user.email",
                  "overdueBooks.$": "$.overdueBooks"
                }
              },
              "End": true
            },
            "SendTextNotification": {
              "Type": "Task",
              "Resource": "arn:aws:states:::http:invoke",
              "Parameters": {
                "ApiEndpoint": "https://your-api.example.com/sendTextNotification",
                "Method": "POST",
                "Headers": {
                  "Content-Type": "application/json"
                },
                "Body": {
                  "phoneNumber.$": "$.user.phone_number",
                  "overdueBooks.$": "$.overdueBooks"
                }
              },
              "End": true
            }
          }
        }
      }
    }
  }
}

Response
Response will return same json as above with just workflow id to it
```

## Workflow execute

```jsx
POST /api/tenant/{tenant_id}/workflows/{workflow_id}/execute

Response
Task
{
	"id": "UUID"
	"status": "IN_PROGRESS"
	"created_at": "some time",
	"updated_at": "some time",
	"message": "Sending notification"
	"steps": "3/8"
}
```

## Workflow trigger

- Workflow service will subscribe to events from kafka
- Consumer will consume these events and check for if has any workflow asssociated
- e.g of message `tenant/{tenant_id}/lending/BOOK_AVAILALBE`

## Other APIs

These are simple CRUD APIs

1. CRUD for permission for workflow
2. Assign permission to user for workflow
3. CRUD get Lest of actions
4. READ APIs for workflow and action history

# Analytics Service

- Top 100 most borrowed books
- top users by most borrowed books
- Users ploted on map as per location (to view which region has more borrowers)
- etc.

# Misc

### Upload (Request Pre-Sgined s3 URL)

```json
POST /api/tenant/{tenant_id}upload/upload_url

Response code: 200 OK
{
   "sha256": "da96c12a41e5334ee287b13428692b701569b823944467db712f371114da7bde"
   "content_length": 5151511,
   "file_name": "photo.png"
}
Response Body
{
  "url": "https://demo.s3.eu-west-2.amazonaws.com/image.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAJJWZ7B6WCRGMKFGQ%2F20180210%2Feu-west-2%2Fs3%2Faws4_request&X-Amz-Date=20180210T171315Z&X-Amz-Expires=1800&X-Amz-Signature=12b74b0788aa036bc7c3d03b3f20c61f1f91cc9ad8873e3314255dc479a25351&X-Amz-SignedHeaders=host",
}

== Error ==
If file is already uploaded then this will throw an error saying file already found.
If name is different for file but sha is same, we can update name in file metadata.
```

1. This API will generate Pre-Signed corresponding to SHA256 of file contents and size of file in bytes
2. Request Body
    1. sha256: SHA-256 of file contents
    2. content_length: File size in bytes
    3. ObjectKey: `sha256`  — This is random UUID will make sure we are not limited by number of write requests. S3 only allows 3500 writes per sec per partitioned prefix. Since prefix is randomized we will not have this limit.
3. Using above properties File Service will generate pre-signed url (
    1. `x-amz-checksum-algorithm: SHA256` 
    2. `x-amz-checksum-sha256:da96c12a41e5334ee287b13428692b701569b823944467db712f371114da7bde`  , 
    3. `contentLength:5151511` 
    4. `x-amz-meta-user_id:<userid>:` User metadata (will be used to map with key)
4. Use multi part s3 upload
5. This will make sure generated URL will not be used for any other file
6. On upload complete s3 will send notification and User service or book service will subscribe to this. Once notification is received User service or book service will fetch object metadata and upsert it in DB 

# Multi Tenant (isolated)

- If tenant does not want to share data with other tenants then we can deploy isolated library management service.
- Each isolated service will be deployed using operator and terraform
- All services are deployed on kubernetes (EKS)
- 

# Notes

- [ ]  Globally defined genre
- [ ]  Soft delete, all queries should make sure deleted_at is null
- [ ]  member should have their own UI to view which books they have rented out and keep history

# Extension

- [ ]  Library Batch for member for access
- [ ]  Monthly subscriptions
- [ ]  Global library - This will be connecting all library management
    - [ ]  Lending across libraries
    - [ ]  Purchasing across libraries