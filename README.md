# DSO-103-Exercise-the-API-Surface
This document maps and explains the main API endpoints of the PostureSec application.  

## Overview

This document maps and explains the main API endpoints of the PostureSec application.  
Exercising the API surface means understanding what each endpoint does and how the frontend communicates with the backend.

---

## API Endpoints

### Health Check
- **Method:** GET  
- **Endpoint:** `/api/health`  
- **Description:** Checks if the backend service is running and healthy.

### Posts

| Method | Endpoint            | Description                              |
|--------|---------------------|------------------------------------------|
| GET    | `/api/posts`        | Retrieves the list of all posts          |
| GET    | `/api/posts/:id`    | Retrieves a single post with its comments|
| POST   | `/api/posts`        | Creates a new post                       |
| PUT    | `/api/posts/:id`    | Updates an existing post                 |
| DELETE | `/api/posts/:id`    | Deletes a post                           |

### Comments

| Method | Endpoint                      | Description                        |
|--------|-------------------------------|------------------------------------|
| GET    | `/api/comments/post/:postId`  | Gets all comments for a specific post |
| POST   | `/api/comments`               | Creates a new comment              |
| DELETE | `/api/comments/:id`           | Deletes a comment                  |

---

## How the API Surface Works

1. The **Frontend** (React) sends HTTP requests to these endpoints.
2. The **Backend** (Express) receives the requests, processes the logic, and talks to the database.
3. The **Database** (PostgreSQL) stores and returns the data.
4. The Backend sends a response back to the Frontend.

This is the core communication pattern of the 3-tier architecture.

## Now in summary to each end point

The API surface is the set of “doors” that the PostureSec application opens to the outside world.  
Exercising the API surface means understanding:

- What each endpoint does
- What a normal request and response look like
- What happens when invalid input is sent
- Where authentication (auth middleware) is applied

This is a core skill for an AI DevSecOps Engineer because most real-world security issues and outages happen at the API level.

---

## 1. `/api/health` – Health Check

### What it is
This is the “Are you alive?” endpoint.  
It simply tells the caller that the backend is running.

### Why it matters
Monitoring tools, load balancers, and Kubernetes use this endpoint to know whether the service is healthy.  
Without it, outages can go unnoticed.

### Request

GET /api/health

### Successful Response
```json
{
  "status": "ok"
}
---
Status code: 200 OK
Authentication
No authentication required.
This route is normally placed outside the auth middleware so monitoring systems can reach it easily.
Failure Modes
Backend is down → Connection refused or timeout
Backend is frozen → Request hangs
Health check is too heavy → False “unhealthy” signals

## 2. `/api/posts` – Posts Endpoints

### What it is
These endpoints manage the main content of the platform (security posts).

### Why it matters
This is the most used part of the API.  
Problems here (like the unbounded query we fixed in INC-002) can make the whole application slow or unavailable.

### Endpoints

| Method | Endpoint         | Description                        | Auth Required |
|--------|------------------|------------------------------------|---------------|
| GET    | `/api/posts`     | Get list of posts                  | No            |
| GET    | `/api/posts/:id` | Get one post with comments         | No            |
| POST   | `/api/posts`     | Create a new post                  | Yes           |
| PUT    | `/api/posts/:id` | Update a post                      | Yes           |
| DELETE | `/api/posts/:id` | Delete a post                      | Yes           |

### Example – Create a Post (Valid Request)

POST /api/posts
Content-Type: application/json

{
  "title": "Understanding Shift-Left Security",
  "content": "Security should start from the beginning of the development process.",
  "author": "Emmanuella",
  "emoji": "🛡️"
}

Example – Invalid Input Rejection
If required fields are missing:
POST /api/posts
{
  "title": ""
}
Expected response:
{
  "error": "Title and content are required"
}

Status code: 400 Bad Request
This shows that the API correctly rejects invalid input.
Authentication Middleware Position
Public routes (GET) → No auth required
Write routes (POST, PUT, DELETE) → Protected by requireAuth middleware
The middleware runs before the route handler.
If the user is not logged in, the request is rejected with 401 Unauthorized.

3. /api/comments – Comments Endpoints
What it is
These endpoints allow users to add and manage comments on posts.
Why it matters
Comments increase interaction but also increase risk (spam, abuse, injection).
Proper validation and authentication are very important here.

### Endpoints

| Method | Endpoint                     | Description                     | Auth Required |
|--------|------------------------------|---------------------------------|---------------|
| GET    | `/api/comments/post/:postId` | Get comments for a post         | No            |
| POST   | `/api/comments`              | Create a new comment            | Yes           |
| DELETE | `/api/comments/:id`          | Delete a comment                | Yes           |

Example – Invalid Input
Sending a comment without required fields should return a 400 Bad Request error.
Authentication Middleware
Same pattern as posts:
Reading comments → Public
Creating or deleting comments → Requires authentication

Summary – Auth Middleware Position
Request comes in
       ↓
[Authentication Middleware]  ← checks if user is logged in
       ↓
Route Handler (only runs if auth passed)

Health check → No auth
Reading posts/comments → No auth
Creating, updating, deleting → Auth required

## Key Lessons for an AI DevSecOps Engineer

Always understand every public door (endpoint) your application opens.
Validate all input — never trust data from the client.
Protect write operations with authentication.
Keep health checks simple and reliable.
Unbounded responses (like the old /api/posts) can become serious availability issues.
Understanding the API surface helps you find weaknesses early and build more secure, reliable systems.


## Key Observations

- Most write operations (POST, PUT, DELETE) require authentication.
- The `/api/posts` endpoint was previously vulnerable to performance issues when returning unbounded results (fixed in INC-002 by adding a `LIMIT`).
- Clear separation exists between posts and comments endpoints.

**Documented by:** Emmanuella Blessing Udeh  
**Date:** November 2026

