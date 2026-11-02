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

---

## Key Observations

- Most write operations (POST, PUT, DELETE) require authentication.
- The `/api/posts` endpoint was previously vulnerable to performance issues when returning unbounded results (fixed in INC-002 by adding a `LIMIT`).
- Clear separation exists between posts and comments endpoints.

**Documented by:** Emmanuella Blessing Udeh  
**Date:** November 2026

