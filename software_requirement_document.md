**\# Prompt Engineering Hands on  
  
\# Software requirement**

### Task 1: Software Requirements & User Stories (Least-to-Most Stage 1)
### ChatGPt Input:
Act as a Business Analyst. We want to design a backend REST API named 'Task Tracker

API'. First, outline the system architecture scope and write 3 core B2B user stories (for

User Signup, Task Creation, and Task Updates) with acceptance criteria. Do not write database schemas or backend code yet.

### ChatGPt Output:

## Task Tracker API — Business Analysis

### 1. System Architecture Scope

The **Task Tracker API** will be a backend REST API that allows B2B users to register, create tasks, and manage task updates.

**In scope:**

- **API Layer**

  - RESTful endpoints for user and task operations.

  - Request validation and standardized API responses.

  - Authentication and authorization.

- **User Management**

  - User signup and account creation.

  - User authentication.

  - Role-based access control, if required by the business.

- **Task Management**

  - Create new tasks.

  - View task details and task status.

  - Update task information and status.

  - Associate tasks with their creator/owner.

- **Business Rules**

  - Only authenticated users can create or update tasks.

  - Users can update only tasks they are authorized to manage.

  - Required fields must be validated before processing.

  - Task status changes must follow defined business rules.

- **Security**

  - Secure authentication mechanism such as token-based authentication.

  - Authorization checks on protected operations.

  - Input validation and appropriate error handling.

- **Integration & Operations**

  - REST/JSON communication.

  - Consistent HTTP status codes and error responses.

  - Logging and monitoring considerations.

**Out of scope for this phase:**

- Database schema/design.

- Backend implementation/code.

- UI/frontend application.

- Detailed infrastructure/deployment design.

- Advanced features such as notifications, reporting, file attachments, and third-party integrations.

## 2. Core B2B User Stories

### User Story 1 — User Signup

**As a new business user,**  
I want to create an account using my required business and login information,  
**so that** I can securely access the Task Tracker API.

#### Acceptance Criteria

1.  Given a user provides all required signup information, when they submit the signup request, then the system creates the user account.

2.  The system must validate all required fields before creating the account.

3.  The system must reject signup when the email/account identifier is already registered.

4.  Invalid input must return an appropriate validation error without creating an account.

5.  Sensitive authentication information must not be returned in the API response.

6.  A successful signup must return an appropriate success response and user identifier.

### User Story 2 — Task Creation

**As an authenticated business user,**  
I want to create a task with relevant task details,  
**so that** I can track work that needs to be completed.

#### Acceptance Criteria

1.  Given an authenticated user provides all required task information, when they submit a task creation request, then the system creates the task.

2.  The task must be associated with the authenticated user or authorized business context.

3.  Required task fields must be validated before creation.

4.  A task must receive a unique identifier.

5.  The system must assign the appropriate initial task status according to the business rules.

6.  An unauthenticated user must not be allowed to create a task.

7.  A successful request must return the created task information and an appropriate HTTP success status.

8.  Invalid task data must return a meaningful validation error.

### User Story 3 — Task Updates

**As an authorized business user,**  
I want to update an existing task's information or status,  
**so that** I can keep task progress accurate and up to date.

#### Acceptance Criteria

1.  Given an authenticated and authorized user identifies an existing task, when they submit valid changes, then the system updates the task.

2.  The system must verify that the task exists before attempting the update.

3.  The system must verify that the requesting user has permission to update the task.

4.  Only permitted task fields may be modified.

5.  Invalid status transitions must be rejected when business rules prohibit them.

6.  An unauthenticated or unauthorized request must not modify the task.

7.  Updating a non-existent task must return an appropriate not-found response.

8.  A successful update must return the updated task information.

9.  The system should maintain appropriate update/audit information, such as who performed the update and when.

**\### Task 2: Design SQL Database Schemas (Stage 2)  
**

**\### ChatGPt Input:  
**  
\[CONTEXT\]: We are building the database layer for our Task Tracker API.

\[ACTION\]: Generate relational SQL table schemas matching the database requirements:

\- Table 'users': columns for id (primary key), name, email (unique), password_hash, and

created_at.

\- Table 'tasks': columns for id (primary key), user_id (foreign key referencing users),

title, description, status, due_date, and created_at.

\[REQUIREMENTS\]:

\- Include SQL constraints (NOT NULL, UNIQUE, FOREIGN KEY, PRIMARY KEY).

\- Provide raw DDL SQL code block.

### ChatGPt Output:

CREATE TABLE users (

id BIGINT PRIMARY KEY,

name VARCHAR(100) NOT NULL,

email VARCHAR(255) NOT NULL UNIQUE,

password_hash VARCHAR(255) NOT NULL,

created_at TIMESTAMP NOT NULL

);

CREATE TABLE tasks (

id BIGINT PRIMARY KEY,

user_id BIGINT NOT NULL,

title VARCHAR(255) NOT NULL,

description TEXT,

status VARCHAR(50) NOT NULL,

due_date DATE,

created_at TIMESTAMP NOT NULL,

CONSTRAINT fk_tasks_user

FOREIGN KEY (user_id)

REFERENCES users(id)

);

### Task 3: Generate Route Controllers (Stage 3 - CRISPE)

**\### ChatGPt Input:  
  
**- CAPACITY: Act as a Senior Backend Software Engineer.

\- RECIPIENT: Junior QA and developer team.

\- INSTRUCTION: Write API route controller logic in Node.js (using Express) or Python

(using FastAPI) to handle:

1\. POST /api/v1/tasks (Create a new task, extracting title, description, due_date).

2\. GET /api/v1/tasks/:id (Read a specific task by ID).

\- STYLE: Follow DRY and SOLID design principles. Include clean error handling.

\- PARAMETERS: Use raw SQL queries matching the schemas created in Step 3. Do not use an

ORM.

\- EVALUATION: Write inline comments explaining how parameters are parsed.

**  
\### ChatGPt Output:**

Below is a FastAPI implementation using PostgreSQL-style parameterized SQL. It keeps the controller thin by separating database access and request/response handling, while avoiding an ORM.

from datetime import date, datetime

from typing import Optional

from fastapi import FastAPI, HTTPException, Depends, Path

from pydantic import BaseModel, Field

import asyncpg

app = FastAPI(title="Task Tracker API")

\# -------------------------------------------------------------------

\# Database configuration

\# -------------------------------------------------------------------

DATABASE_URL = "postgresql://user:password@localhost/task_tracker"

async def get_db():

"""

Creates a database connection for the request.

The connection is closed after the request completes.

"""

connection = await asyncpg.connect(DATABASE_URL)

try:

yield connection

finally:

await connection.close()

\# -------------------------------------------------------------------

\# Request model

\# -------------------------------------------------------------------

class TaskCreateRequest(BaseModel):

title: str = Field(..., min_length=1, max_length=255)

description: Optional\[str\] = None

due_date: Optional\[date\] = None

\# -------------------------------------------------------------------

\# POST /api/v1/tasks

\# Create a new task

\# -------------------------------------------------------------------

@app.post("/api/v1/tasks", status_code=201)

async def create_task(

task: TaskCreateRequest,

db=Depends(get_db)

):

"""

Creates a new task for the authenticated user.

In a real application, user_id should come from the

authentication context rather than the request body.

"""

\# FastAPI/Pydantic automatically parses the incoming JSON body

\# into the TaskCreateRequest object. For example:

\#

\# {

\# "title": "Prepare report",

\# "description": "Prepare the monthly report",

\# "due_date": "2026-10-05"

\# }

\#

\# task.title, task.description, and task.due_date contain the

\# parsed values after validation.

user_id = 1 \# Replace with authenticated user's ID

sql = """

INSERT INTO tasks (

user_id,

title,

description,

status,

due_date,

created_at

)

VALUES (\$1, \$2, \$3, \$4, \$5, \$6)

RETURNING

id,

user_id,

title,

description,

status,

due_date,

created_at

"""

try:

row = await db.fetchrow(

sql,

user_id,

task.title,

task.description,

"OPEN",

task.due_date,

datetime.utcnow()

)

return dict(row)

except asyncpg.PostgresError:

\# Do not expose database-specific details to the API client.

raise HTTPException(

status_code=500,

detail="Unable to create task"

)

\# -------------------------------------------------------------------

\# GET /api/v1/tasks/:id

\# Read a specific task

\# -------------------------------------------------------------------

@app.get("/api/v1/tasks/{id}")

async def get_task(

id: int = Path(..., gt=0),

db=Depends(get_db)

):

\# FastAPI parses the {id} route parameter and converts it to an

\# integer because the parameter is declared as \`id: int\`.

\#

\# For example:

\# GET /api/v1/tasks/25

\#

\# results in:

\# id = 25

\#

\# Path(..., gt=0) also validates that the ID is greater than zero.

sql = """

SELECT

id,

user_id,

title,

description,

status,

due_date,

created_at

FROM tasks

WHERE id = \$1

"""

try:

row = await db.fetchrow(sql, id)

if row is None:

raise HTTPException(

status_code=404,

detail="Task not found"

)

return dict(row)

except HTTPException:

\# Preserve the intended 404 response.

raise

except asyncpg.PostgresError:

raise HTTPException(

status_code=500,

detail="Unable to retrieve task"

)

**Key QA points**

- **POST /api/v1/tasks** accepts title, description, and due_date from the JSON request body.

- **GET /api/v1/tasks/{id}** extracts and validates the task ID from the URL.

- SQL uses **parameterized queries** (\$1, \$2, etc.), avoiding SQL injection.

- No ORM is used.

- 404 is returned when the requested task does not exist.

- Database errors are not exposed directly to API consumers.

- The user_id = 1 placeholder should be replaced by the authenticated user's ID once authentication middleware is implemented.

- The SQL matches the users/tasks schema from the previous step.

**\### Task 4: Run Secure Code Reflection & Debugging (Stage 4)  
  
\### ChatGPt Input**

**  
**// Vulnerable Express Route

app.post('/api/v1/login', (req, res) =\> {

let query = "SELECT \* FROM users WHERE email = '" + req.body.email + "' AND

password = '" + req.body.password + "'";

db.query(query, (err, result) =\> {

if (err) throw err;

res.send(result);

});

});

### ChatGPt Output:

This Express login route has two major security issues:

1.  **SQL injection** — user input is concatenated directly into the SQL query.

2.  **Plaintext password comparison** — the schema uses password_hash, so passwords should be verified against the stored hash rather than queried directly.

A safer version using **parameterized SQL** and \`bcrypt\*\* would be:

const express = require('express');

const bcrypt = require('bcrypt');

const router = express.Router();

router.post('/api/v1/login', async (req, res) =\> {

try {

// Parse login parameters from the JSON request body.

// Expected body:

// {

// "email": "user@example.com",

// "password": "userPassword"

// }

const { email, password } = req.body;

// Validate required parameters before querying the database.

if (!email \|\| !password) {

return res.status(400).json({

error: 'Email and password are required'

});

}

// Use a parameterized query instead of concatenating user input.

// \$1 is safely replaced with \`email\` by the database driver.

const query = \`

SELECT id, name, email, password_hash, created_at

FROM users

WHERE email = \$1

\`;

const result = await db.query(query, \[email\]);

// Avoid revealing whether the email exists.

if (result.rows.length === 0) {

return res.status(401).json({

error: 'Invalid email or password'

});

}

const user = result.rows\[0\];

// Compare the supplied plaintext password against the stored

// bcrypt password hash. The plaintext password is never stored.

const passwordMatches = await bcrypt.compare(

password,

user.password_hash

);

if (!passwordMatches) {

return res.status(401).json({

error: 'Invalid email or password'

});

}

// Do not return password_hash to the client.

return res.status(200).json({

id: user.id,

name: user.name,

email: user.email,

created_at: user.created_at

});

} catch (err) {

// Log the detailed error internally, but don't expose database

// or implementation details to the API consumer.

console.error('Login error:', err);

return res.status(500).json({

error: 'Internal server error'

});

}

});

module.exports = router;

**What changed**

| **Vulnerability/Issue** | **Original**            | **Secure approach**              |
|-------------------------|-------------------------|----------------------------------|
| SQL injection           | String concatenation    | Parameterized query              |
| Password storage        | Queries password        | Uses password_hash               |
| Password verification   | SQL comparison          | bcrypt.compare()                 |
| Missing parameters      | Not validated           | 400 Bad Request                  |
| Invalid credentials     | Returns database result | Generic 401 response             |
| Sensitive data          | SELECT \* + res.send()  | Explicit columns, excluding hash |
| DB errors               | throw err               | Controlled 500 response          |

**Important:** For a production API, successful authentication should also establish a session or issue a securely signed access token (such as a short-lived JWT), rather than simply returning the user record.

**\### Task 5: Run Secure Code Reflection & Debugging (Stage 4)  
  
\### ChatGPt Input**

Act as a Security Auditor. Scan the code block below for security vulnerabilities (e.g.

SQL Injection, unhandled crashes).

\[CODE TO AUDIT\]:

\[Insert the Vulnerable Express Route code above\]

Identify the issue, explain the risk, and write a refactored, secure version of this

endpoint using parameterized SQL queries and try/catch blocks.

### ChatGPt Output:

### Security Audit

The vulnerable login endpoint has several security issues:

| **Issue**                     | **Risk**                                                                                                                                       | **Severity** |
|-------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|--------------|
| SQL Injection                 | An attacker can manipulate the SQL query through email or password input, potentially bypassing authentication or accessing unauthorized data. | **Critical** |
| Plaintext password comparison | Passwords are compared directly in SQL rather than against a secure password hash.                                                             | **High**     |
| Unhandled database errors     | throw err can crash the request/process and expose implementation details.                                                                     | **High**     |
| SELECT \*                     | Returns unnecessary fields and increases the chance of exposing sensitive information.                                                         | **Medium**   |
| Returning raw DB results      | Could expose password hashes or other internal fields to the client.                                                                           | **High**     |
| No input validation           | Malformed or missing credentials are passed directly to the database.                                                                          | **Medium**   |
| User enumeration              | Different responses could reveal whether an email exists if implemented inconsistently.                                                        | **Medium**   |

### Refactored Secure Endpoint

The following version uses **parameterized SQL**, bcrypt password verification, input validation, and try/catch error handling.

const express = require('express');

const bcrypt = require('bcrypt');

const router = express.Router();

router.post('/api/v1/login', async (req, res) =\> {

try {

// Parse email and password from the JSON request body.

// Example:

// {

// "email": "user@example.com",

// "password": "MyPassword123"

// }

const { email, password } = req.body;

// Validate that both parameters were supplied.

if (

typeof email !== 'string' \|\|

typeof password !== 'string' \|\|

!email.trim() \|\|

!password

) {

return res.status(400).json({

error: 'Email and password are required'

});

}

// Parameterized SQL prevents user input from becoming

// executable SQL. \$1 is a parameter placeholder.

const query = \`

SELECT

id,

name,

email,

password_hash,

created_at

FROM users

WHERE email = \$1

\`;

// The email value is supplied separately from the SQL statement.

const result = await db.query(query, \[email.trim()\]);

// Use the same generic response for an unknown email or

// incorrect password to reduce account enumeration risk.

if (result.rows.length === 0) {

return res.status(401).json({

error: 'Invalid email or password'

});

}

const user = result.rows\[0\];

// Compare the supplied password against the stored bcrypt hash.

// The plaintext password is never stored or returned.

const passwordMatches = await bcrypt.compare(

password,

user.password_hash

);

if (!passwordMatches) {

return res.status(401).json({

error: 'Invalid email or password'

});

}

// Return only non-sensitive user information.

return res.status(200).json({

id: user.id,

name: user.name,

email: user.email,

created_at: user.created_at

});

} catch (error) {

// Log detailed information internally for troubleshooting.

// Do not expose database errors to the API consumer.

console.error('Login error:', error);

return res.status(500).json({

error: 'Internal server error'

});

}

});

module.exports = router;

### Key Security Improvements

**1. SQL Injection → Fixed**

Instead of:

"WHERE email = '" + req.body.email + "'"

the secure implementation uses:

WHERE email = \$1

with:

db.query(query, \[email\])

The database driver treats the supplied email as **data**, not executable SQL.

**2. Password handling → Fixed**

The endpoint retrieves password_hash and verifies it with:

bcrypt.compare(password, user.password_hash)

The plaintext password is never stored or returned.

**3. Database crashes → Fixed**

Database operations are wrapped in:

try {

// database operation

} catch (error) {

// controlled response

}

The client receives a generic 500 response rather than internal database details.

**4. Sensitive response data → Fixed**

The endpoint no longer sends the complete database row. In particular, password_hash is excluded from the response.

**5. Authentication enumeration → Reduced**

Both an unknown email and an incorrect password return:

401 Invalid email or password

For production, the next security layer should include **rate limiting/lockout protection and a secure session or short-lived access token** after successful authentication.

**\### Task 5: API Documentation Prompt  
  
\### ChatGPt Input:**

Generate Swagger/OpenAPI yaml documentation for the two endpoints:

\- POST /api/v1/tasks

\- GET /api/v1/tasks/:id

Include request schema, 200 success response block, and 400 error response blocks.

### ChatGPt Output:

Below is an OpenAPI 3.0 YAML definition for the two Task Tracker API endpoints. I’ve included request schemas and the requested 200/400 response blocks, while also documenting the 404 case for the GET endpoint.

openapi: 3.0.3

info:

title: Task Tracker API

version: 1.0.0

description: REST API for creating and retrieving tasks.

paths:

/api/v1/tasks:

post:

summary: Create a new task

operationId: createTask

requestBody:

required: true

content:

application/json:

schema:

\$ref: '#/components/schemas/CreateTaskRequest'

responses:

'200':

description: Task created successfully

content:

application/json:

schema:

\$ref: '#/components/schemas/Task'

'400':

description: Invalid task data

content:

application/json:

schema:

\$ref: '#/components/schemas/ErrorResponse'

/api/v1/tasks/{id}:

get:

summary: Get a task by ID

operationId: getTaskById

parameters:

\- name: id

in: path

required: true

description: Unique identifier of the task

schema:

type: integer

format: int64

minimum: 1

responses:

'200':

description: Task retrieved successfully

content:

application/json:

schema:

\$ref: '#/components/schemas/Task'

'400':

description: Invalid task ID

content:

application/json:

schema:

\$ref: '#/components/schemas/ErrorResponse'

'404':

description: Task not found

content:

application/json:

schema:

\$ref: '#/components/schemas/ErrorResponse'

components:

schemas:

CreateTaskRequest:

type: object

required:

\- title

properties:

title:

type: string

maxLength: 255

description: Title of the task

example: Prepare monthly report

description:

type: string

description: Detailed description of the task

example: Prepare and submit the September monthly report.

due_date:

type: string

format: date

nullable: true

description: Task due date

example: '2026-10-05'

Task:

type: object

required:

\- id

\- user_id

\- title

\- status

\- created_at

properties:

id:

type: integer

format: int64

example: 101

user_id:

type: integer

format: int64

example: 25

title:

type: string

example: Prepare monthly report

description:

type: string

nullable: true

example: Prepare and submit the September monthly report.

status:

type: string

example: OPEN

due_date:

type: string

format: date

nullable: true

example: '2026-10-05'

created_at:

type: string

format: date-time

example: '2026-09-28T10:30:00Z'

ErrorResponse:

type: object

required:

\- error

properties:

error:

type: string

description: Human-readable error message

example: Invalid task data

**Note:** In OpenAPI, the Express-style route /api/v1/tasks/:id is documented as /api/v1/tasks/{id}. The {id} path parameter is then explicitly defined under parameters.

**\### Task 6: Unit Test Prompt  
  
\### ChatGPt Input:  
  
**Act as a QA Automation Engineer. Write 2 unit tests in Jest/Supertest (or Python

PyTest) to validate:

\- Test Case 1: Successful task creation (asserts status 201).

\- Test Case 2: Getting task with invalid ID (asserts status 404).

### ChatGPt Output:

Below are two Jest/Supertest tests assuming the Task Tracker API is implemented with **Node.js + Express** and exposes the routes from the previous steps.

const request = require('supertest');

const app = require('../app');

describe('Task Tracker API', () =\> {

// Test Case 1: Successful task creation

test('should create a new task successfully', async () =\> {

const response = await request(app)

.post('/api/v1/tasks')

.send({

title: 'Prepare monthly report',

description: 'Prepare the September report',

due_date: '2026-10-05'

});

// Verify that the API returns HTTP 201 Created.

expect(response.statusCode).toBe(201);

// Verify that the response contains the created task.

expect(response.body).toHaveProperty('id');

expect(response.body.title).toBe('Prepare monthly report');

});

// Test Case 2: Getting task with invalid ID

test('should return 404 when task does not exist', async () =\> {

const response = await request(app)

.get('/api/v1/tasks/999999');

// Verify that the API returns HTTP 404 Not Found.

expect(response.statusCode).toBe(404);

// Verify that an appropriate error is returned.

expect(response.body).toHaveProperty('error');

expect(response.body.error).toBe('Task not found');

});

});

### Test Coverage

| **Test Case**                | **Request**                             | **Expected Result** |
|------------------------------|-----------------------------------------|---------------------|
| Successful task creation     | POST /api/v1/tasks with valid task data | **201 Created**     |
| Invalid/non-existent task ID | GET /api/v1/tasks/999999                | **404 Not Found**   |

**QA note:** For reliable automated tests, the test database should be isolated from production data, and the first test should clean up the created task or use a transaction/fixture that resets the test data after execution.
