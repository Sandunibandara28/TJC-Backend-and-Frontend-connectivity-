# TJC-Backend-and-Frontend-connectivity

# System Architecture Overview

This project follows a client–server architecture where,
* Frontend Vue + Vite
* Backend is a RESTful API built using Spring Boot.
* Communication happens via HTTP requests (JSON format)
* Security is handled using JWT (JSON Web Tokens)

User → Vue Frontend → Axios → Spring Boot API → Database
                     ↑
                   JWT Token

# Backend (Spring Boot) 
## REST API Concept

The backend exposes RESTful endpoints, such as:
* POST /api/visit-reports
* GET  /api/farmers
* POST /api/auth/login

Each endpoint:
* Accepts JSON requests
* Returns JSON responses
* Is stateless (important for JWT)

# JWT Authentication
JWT is a stateless authentication mechanism.

## JWT Structure
A JWT consists of three parts:
HEADER.PAYLOAD.SIGNATURE

Header Contains: Token type, Algorithm used
Payload (Claims): Contains user data

Signature: Created using secret key, Prevents token tampering

If someone changes payload → signature fails → token invalid

Example: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Why JWT?
* No server-side session storage
* Scalable
* Secure
* Widely supported

# JWT Authentication Flow
* User logs in with username & password
* Backend validates credentials
* Backend generates a JWT token
* Token is sent to frontend
* Frontend stores token
* Token is sent in every protected request

Authorization: Bearer <JWT_TOKEN>

# Frontend (Vue)
## Axios for HTTP Communication
* Axios is used to:
* Send API requests
* Handle responses
* Attach JWT tokens automatically

# API Configuration
api.ts (Central Axios Instance)
import axios from "axios";

const API_BASE_URL =
  (import.meta as any).env?.VITE_API_BASE_URL ||
  "https://tjc-qr-api.gtsactive.lk/api";

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    "Content-Type": "application/json",
  },
});

# Why This Is Important
* Single place to manage backend URL
* Easy to switch environments (dev / prod)
* Avoids repeating base URL everywhere

# Connecting Frontend to Backend (Without JWT)

Example: Fetch Farmers
const res = await api.get("/farmers");
farmers.value = res.data;

This calls: GET http://localhost:8080/api/farmers

# Connecting Frontend to Backend (With JWT)
Login API (Backend)
POST /api/auth/login

Response:
{
  "token": "jwt-token-here",
  "role": "FIELD_OFFICER"
}

Store JWT Token (Frontend)
localStorage.setItem("token", response.data.token);

# Attach JWT Automatically (Axios Interceptor)
Update request interceptor:
api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});

Now every request automatically sends JWT

# Data Flow Example (Visit Report Page)
Step-by-Step
* User fills form
* Frontend validates input
* Payload is built (buildPayload())
* Axios sends POST request
* JWT is attached automatically
* Backend validates token
* Backend saves data
* Response returned

# -----------------------------------------------------------------------
# JWT
JWT (JSON Web Token) is a secure way to identify a logged-in user without storing sessions on the server.
Instead of:
Server remembering every logged-in user (session-based)

* Statelessness: The backend doesn't "remember" who you are.
* The Token: When a user logs in, the Spring Boot backend generates a JWT. This token contains user info (encrypted) and an expiration date.
* The Handshake: The frontend stores this token (usually in localStorage or a Cookie) and sends it in the Authorization Header of every request.

* Server gives the user a token
* User sends that token with every request
* Server verifies the token and decides who you are and what you can access

# JWT helps:
* Identify who is calling the API
* Control which endpoints they can access

# Backend Setup (Spring Boot + Spring Security)

To connect to Vue app, Spring Boot needs two things: CORS configuration and JWT Security.

  ## Step A: Enable CORS
Since your Vue app (e.g., localhost:5173) and Spring Boot (e.g., localhost:8080) run on different ports, the browser will block requests unless you allow them.

  ## Step B: JWT Security Flow
Dependencies: Add spring-boot-starter-security and jjwt (Java JWT library) to pom.xml.

Filter: Create a JwtAuthenticationFilter that intercepts every request, looks for the token in the header, and validates it.

SecurityConfig

# Frontend Setup (Vue + Axios)
codes already uses an Axios Interceptor. This is the best way to handle JWT.

Step A: Managing the Token
Update api.ts to automatically attach the token if it exists in storage.
Step B: Handling Expired Tokens
If the backend returns a 401 Unauthorized, the user should be redirected to login.

# Connecting the Pages (The "TJC" Way)
Ex:
In FarmerRegistration.vue, call registerFarmer(submissionData).

The Trigger: When you click "Register Farmer", the method registerFarmer is called.

The Service: It goes to farmerService.ts, which calls api.post('/farmers/addfarmers', data).

The Network: Axios sends the request to https://tjc-qr-api.gtsactive.lk/api/farmers/addfarmers.

The Interceptor: interceptor injects the JWT into the header right before it leaves the browser.

The Result: Spring Boot validates the token, processes the data, and sends back a 200 OK.

# Spring Boot JWT Components (Theory)

Spring Boot project will have these layers:
## Authentication Controller
Purpose: Handle login requests
Endpoint: POST /auth/login

Responsibilities:
Accept username & password
Call authentication service
Return JWT

## User Entity
Represents system users:
User
- id
- username
- password
- role (ADMIN, FIELD_OFFICER, FARMER)

This role will be placed inside JWT.

## JWT Utility Class
Purpose:
Generate token
Validate token
Extract username & role

## JWT Filter 
This runs before every request.

What it does:
Reads Authorization header
Extracts token
Validates token
Sets authenticated user in Spring Security Context

Without this → JWT won’t work.

## Spring Security Configuration

Controls:
Which endpoints are public
Which need authentication
Role-based access

Example logic:
/auth/** → public
/farmer/** → FARMER only
/field-officer/** → FIELD_OFFICER only
/admin/** → ADMIN only