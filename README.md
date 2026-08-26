# Spring Boot Authenticated Hello World API

This project implements the backend assessment with Spring Boot 3.2.0 and Java 17.

## Features

- `POST /api/auth/login`: validates a username and password and creates an HTTP session.
- `GET /api/hello`: protected by Spring Security and returns `Hello World` after login.
- `POST /api/auth/logout`: invalidates the current session.
- Passwords are stored with BCrypt, even for the in-memory demo account.
- Integration tests cover authentication success, failure, validation, authorization, and logout.

Demo credentials:

```text
username: test
password: 123456
```

## Run

Requirements: JDK 17 and Maven 3.9+.

```bash
mvn spring-boot:run
```

The application starts at `http://localhost:8080`.

## Try the API

The cookie jar is important because the authenticated state is stored in the server-side HTTP session.

```bash
# Before login: returns HTTP 401
curl -i http://localhost:8080/api/hello

# Login and save the JSESSIONID cookie
curl -i -c cookies.txt \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"123456"}' \
  http://localhost:8080/api/auth/login

# Reuse the cookie: returns HTTP 200 and Hello World
curl -i -b cookies.txt http://localhost:8080/api/hello

# Logout
curl -i -b cookies.txt -X POST http://localhost:8080/api/auth/logout
```

PowerShell example:

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$body = @{ username = 'test'; password = '123456' } | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri http://localhost:8080/api/auth/login `
  -ContentType 'application/json' -Body $body -WebSession $session
Invoke-RestMethod -Uri http://localhost:8080/api/hello -WebSession $session
```

## Test

```bash
mvn test
```

## Project structure

```text
src/main/java/com/d5data/backend/
├── auth/       Login API and request/response DTOs
├── config/     Spring Security configuration and demo user
├── hello/      Protected Hello World API
└── BackendHelloWorldApplication.java
```

## Authentication flow

1. The login controller passes the submitted credentials to Spring Security's `AuthenticationManager`.
2. `DaoAuthenticationProvider` loads the in-memory user and checks the BCrypt password hash.
3. On success, the controller stores the resulting `Authentication` in a `SecurityContext` inside the HTTP session.
4. On later requests, Spring Security restores that context from the `JSESSIONID` session cookie.
5. The authorization rules allow the login endpoint anonymously and require authentication everywhere else.

For a production system, replace the in-memory user with a database-backed `UserDetailsService`, keep secrets outside source control, enable an appropriate CSRF strategy, and configure HTTPS and secure cookies.
