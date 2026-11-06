# DSO-103: Exercise the API Surface

**Documented by:** Emmanuella Blessing Udeh  
**Date:** November 2026

## Overview

The API surface is the set of “doors” that the PostureSec application opens to the outside world.  
Exercising the API surface means understanding what each endpoint does, what a normal request and response look like, what happens when invalid input is sent, and where authentication is applied.

This is a core skill for an AI DevSecOps Engineer because most real-world security issues and outages happen at the API level.

---

## 1. /api/health – Health Check

**What it is**  
This is the “Are you alive?” endpoint. It simply tells the caller that the backend is running.

**Why it matters**  
Monitoring tools, load balancers, and Kubernetes use this endpoint to know whether the service is healthy. Without it, outages can go unnoticed.

**Request**  
GET /api/health

**Successful Response**  
{ "status": "ok" }  
Status code: 200 OK

**Authentication**  
No authentication required. This route is normally placed outside the auth middleware so monitoring systems can reach it easily.

**Failure Modes**  
- Backend is down → Connection refused or timeout  
- Backend is frozen → Request hangs  
- Health check is too heavy → False “unhealthy” signals

---

## 2. /api/posts – Posts Endpoints

**What it is**  
These endpoints manage the main content of the platform (security posts).

**Why it matters**  
This is the most used part of the API. Problems here (like the unbounded query we fixed in INC-002) can make the whole application slow or unavailable.

**Endpoints**  
- GET /api/posts → Get list of posts (No Auth)  
- GET /api/posts/:id → Get one post with comments (No Auth)  
- POST /api/posts → Create a new post (Auth Required)  
- PUT /api/posts/:id → Update a post (Auth Required)  
- DELETE /api/posts/:id → Delete a post (Auth Required)

**Example – Create a Post (Valid Request)**  
POST /api/posts  
Content-Type: application/json  

{  
  "title": "Understanding Shift-Left Security",  
  "content": "Security should start from the beginning of the development process.",  
  "author": "Emmanuella",  
  "emoji": "🛡️"  
}

**Example – Invalid Input Rejection**  
If required fields are missing, the API should return:  
{ "error": "Title and content are required" }  
Status code: 400 Bad Request  

This shows that the API correctly rejects invalid input.

**Authentication Middleware Position**  
- Public routes (GET) → No auth required  
- Write routes (POST, PUT, DELETE) → Protected by requireAuth middleware  

The middleware runs before the route handler. If the user is not logged in, the request is rejected with 401 Unauthorized.

---

## 3. /api/comments – Comments Endpoints

**What it is**  
These endpoints allow users to add and manage comments on posts.

**Why it matters**  
Comments increase interaction but also increase risk (spam, abuse, injection). Proper validation and authentication are very important here.

**Endpoints**  
- GET /api/comments/post/:postId → Get comments for a post (No Auth)  
- POST /api/comments → Create a new comment (Auth Required)  
- DELETE /api/comments/:id → Delete a comment (Auth Required)

**Example – Invalid Input**  
Sending a comment without required fields should return a 400 Bad Request error.

**Authentication Middleware**  
Same pattern as posts:  
- Reading comments → Public  
- Creating or deleting comments → Requires authentication

---

## Summary – Auth Middleware Position

Request comes in  
↓  
[Authentication Middleware] ← checks if user is logged in  
↓  
Route Handler (only runs if auth passed)

- Health check → No auth  
- Reading posts/comments → No auth  
- Creating, updating, deleting → Auth required

---

## Key Lessons for an AI DevSecOps Engineer

1. Always understand every public door (endpoint) your application opens.  
2. Validate all input  never trust data from the client.  
3. Protect write operations with authentication.  
4. Keep health checks simple and reliable.  
5. Unbounded responses (like the old /api/posts) can become serious availability issues.

Understanding the API surface helps you find weaknesses early and build more secure, reliable systems.
