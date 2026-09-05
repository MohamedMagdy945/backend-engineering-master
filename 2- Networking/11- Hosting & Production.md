#  Hosting & Production

##  Overview

Building an application is only part of the job.

A production application must be able to:

- Run continuously
    
- Accept requests from real users
    
- Be accessible through a domain
    
- Use HTTPS
    
- Store data safely
    
- Restart if it crashes
    
- Protect secrets
    
- Generate logs
    
- Be monitored
    
- Be backed up
    
- Be updated without breaking users
    

This document focuses on **hosting and running an ASP.NET Core application in a real production environment**.

---

# 1. What Is Production?

During development, your application runs on your computer.

```text
Developer Computer

ASP.NET Core API
      │
      ▼
http://localhost:5000
```

This is not production.

In production, the application runs on a server that is available to users.

```text
Users
  │
  ▼
Internet
  │
  ▼
Production Server
  │
  ▼
Your Application
```

A production environment must be designed to run reliably for a long time.

The goal is not simply:

> "The application is running."

The real goal is:

> "The application is secure, stable, recoverable, observable, and accessible to users."

---

# 2. What Is Hosting?

**Hosting** means providing an environment where your application can run and be accessed by users.

For example:

```text
Your ASP.NET Core Application
            │
            ▼
        Linux Server
            │
            ▼
          Internet
            │
            ▼
           Users
```

The server can be:

- VPS
    
- Virtual Machine
    
- Cloud Server
    
- Physical Server
    
- Container Platform
    

For a small or medium application, a common setup is:

```text
VPS
 │
 ├── Linux
 │
 ├── Docker
 │
 ├── Nginx
 │
 ├── ASP.NET Core API
 │
 └── Database
```

---

# 3. What Is a Production Server?

A production server is simply a computer dedicated to running applications.

For example:

```text
Server

CPU
RAM
Disk
Network
Operating System
```

A typical Linux server might run:

```text
Ubuntu Server
      │
      ├── Docker
      │     │
      │     ├── API Container
      │     └── Other Services
      │
      ├── Nginx
      │
      └── Monitoring
```

The server usually has a public IP address.

Example:

```text
203.xxx.xxx.xxx
```

Users do not normally access this IP directly.

Instead:

```text
api.myapp.com
      │
      ▼
DNS
      │
      ▼
Server Public IP
```

---

# 4. Production Architecture

A typical production architecture looks like this:

```text
                        Users
                          │
                          │ HTTPS
                          ▼
                       Internet
                          │
                          ▼
                         DNS
                          │
                          ▼
                    Public Server
                          │
                          ▼
                  Reverse Proxy
                       Nginx
                          │
                          ▼
                 ASP.NET Core API
                          │
                          ▼
                       Database
```

With Docker:

```text
                         SERVER
┌────────────────────────────────────────────┐
│                                            │
│   Internet                                 │
│      │                                     │
│      ▼                                     │
│    Nginx                                   │
│      │                                     │
│      ▼                                     │
│  Docker Network                            │
│      │                                     │
│      ├───────────────┐                     │
│      ▼               ▼                     │
│   API Container   Other Services           │
│      │                                     │
│      ▼                                     │
│ Database                                   │
│                                            │
└────────────────────────────────────────────┘
```

---

# 5. The Complete User Request Flow

Suppose a user requests:

```text
https://api.myapp.com/users
```

The request travels through several layers.

```text
User
 │
 │ https://api.myapp.com/users
 ▼
Internet
 │
 ▼
DNS
 │
 ▼
Server IP
 │
 ▼
Nginx
 │
 ▼
ASP.NET Core API
 │
 ▼
Database
 │
 ▼
ASP.NET Core API
 │
 ▼
Nginx
 │
 ▼
User
```

Let's understand every step.

---

# 6. DNS Resolves the Domain

The user enters:

```text
api.myapp.com
```

The browser needs to know:

> Where is this server?

DNS resolves the domain.

```text
api.myapp.com
        │
        ▼
DNS Lookup
        │
        ▼
203.xxx.xxx.xxx
```

For example, an `A Record` might be:

```text
api.myapp.com → 203.xxx.xxx.xxx
```

Now the browser knows which server to contact.

---

# 7. The Request Reaches the Server

The user sends:

```text
HTTPS Request
```

to:

```text
Server IP:443
```

Port `443` is normally used for HTTPS.

The request reaches the production server.

```text
Internet
   │
   ▼
Server
   │
   ▼
Port 443
```

But your ASP.NET Core application usually does not directly handle the public internet traffic.

Instead, the request goes to a **reverse proxy**.

---

# 8. Reverse Proxy

A reverse proxy sits in front of your application.

A common choice is Nginx.

```text
Internet
    │
    ▼
  Nginx
    │
    ▼
ASP.NET Core API
```

Nginx receives the public request first.

For example:

```text
https://api.myapp.com/users
```

Nginx forwards it internally:

```text
http://api:8080/users
```

Or:

```text
http://localhost:5000/users
```

The user does not need to know where your application is actually running.

---

# 9. Why Use a Reverse Proxy?

The reverse proxy acts as the public entry point.

```text
                 Internet
                     │
                     ▼
                  Nginx
              ┌──────┴──────┐
              │             │
              ▼             ▼
            API 1         API 2
```

It can handle:

- HTTPS
    
- SSL certificates
    
- HTTP to HTTPS redirects
    
- Request forwarding
    
- Load balancing
    
- Rate limiting
    
- Request size limits
    
- Static files
    
- Security rules
    

Your application can stay inside a private network.

---

# 10. Kestrel in Production

ASP.NET Core uses **Kestrel** as its web server.

Your application might listen on:

```text
Port 8080
```

Example:

```text
Nginx
   │
   ▼
Kestrel
   │
   ▼
ASP.NET Core
```

Inside Docker:

```text
Nginx
   │
   ▼
api:8080
```

The important idea is:

```text
Public Traffic
      │
      ▼
Nginx
      │
      ▼
Internal Application
```

You usually do not want to expose every application port directly to the internet.

---

# 11. Docker in Production

Docker packages your application with its required runtime environment.

Instead of manually installing everything:

```text
Install .NET
Copy Application
Configure Service
Configure Dependencies
```

You can create an image:

```text
ASP.NET Core Application
          │
          ▼
      Dockerfile
          │
          ▼
      Docker Image
          │
          ▼
   Production Server
          │
          ▼
   Docker Container
```

A production server may run:

```text
Docker

├── api
├── worker
├── redis
└── other services
```

Docker helps ensure:

```text
Works Locally
      ↓
Works in Production
```

Because the application runs inside the same type of container environment.

---

# 12. Docker Compose in Production

For multiple services, Docker Compose can manage the application.

Example:

```text
Production Server

docker-compose.yml

├── API
├── Redis
└── Background Worker
```

Architecture:

```text
                 Docker Network

        ┌─────────────────────────┐
        │                         │
        │       API               │
        │        │                │
        │        ▼                │
        │      Redis              │
        │                         │
        │       Worker            │
        │                         │
        └─────────────────────────┘
```

Services can communicate internally using service names.

For example:

```text
api → redis
worker → api
```

Instead of depending on public IP addresses.

---

# 13. Environment Variables

Production configuration should not be hardcoded into your application.

Bad:

```text
Connection String
JWT Secret
API Key
Password
```

inside:

```text
appsettings.json
```

or directly inside the source code.

Instead:

```text
Production Server
       │
       ▼
Environment Variables
       │
       ▼
Application
```

Example:

```text
ConnectionStrings__DefaultConnection

Jwt__Secret

Jwt__Issuer

Redis__ConnectionString
```

Your application reads the configuration when it starts.

---

# 14. Development vs Production Configuration

You should separate environments.

```text
Development
    │
    ▼
Local Database
Detailed Errors
Debugging
Test Services
```

Production:

```text
Production
    │
    ▼
Real Database
Secure Secrets
HTTPS
Minimal Error Details
Logging
Monitoring
```

Example:

```text
appsettings.json

appsettings.Development.json

appsettings.Production.json
```

Sensitive values should preferably come from secure environment configuration or a secrets-management solution rather than being committed to Git.

---

# 15. Secrets in Production

Secrets include:

```text
Database Password
JWT Secret
API Keys
SMTP Password
Payment Gateway Keys
```

Never commit secrets to Git.

Bad:

```text
GitHub Repository
        │
        └── .env
             └── DATABASE_PASSWORD=123456
```

Better:

```text
Git Repository
      │
      └── Application Code

Production Server
      │
      └── Secrets
            │
            ├── Database Password
            ├── JWT Secret
            └── API Keys
```

The production server provides these values when the application starts.

---

# 16. The Database in Production

Your database is one of the most important parts of your production system.

A basic architecture:

```text
Internet
   │
   ▼
  API
   │
   ▼
Database
```

A better security model:

```text
Internet
   │
   ▼
Nginx
   │
   ▼
  API
   │
   ▼
Private Database
```

The database should normally not be publicly accessible.

The API communicates with it through a private network.

---

# 17. Database Persistence

Containers can be removed and recreated.

Therefore, database data must be stored in persistent storage.

Bad:

```text
Database Container
       │
       ▼
Container Storage
       │
       ▼
Container Removed
       │
       ▼
Data Lost
```

Better:

```text
Database Container
       │
       ▼
Docker Volume
       │
       ▼
Persistent Disk
```

Example:

```text
SQL Server Container
        │
        ▼
Docker Volume
        │
        ▼
Server Disk
```

Important:

> A Docker volume is not a backup.

If the server or disk fails, you can still lose data.

---

# 18. Backups

Production systems need backups.

A typical strategy:

```text
Database
    │
    ├── Daily Backup
    │
    ├── Weekly Backup
    │
    └── Remote Storage
```

Better architecture:

```text
Production Server
       │
       ▼
Database Backup
       │
       ▼
Remote Storage
```

Do not store the only backup on the same server as the database.

If the server is destroyed:

```text
Server ❌
Database ❌
Local Backup ❌
```

A good backup must be stored somewhere independent.

You should also test restoring your backups.

```text
Backup Created
      │
      ▼
Can It Be Restored?
      │
      ├── Yes → Good
      │
      └── No → Backup Is Not Reliable
```

---

# 19. Process Availability

Your application can crash.

For example:

```text
API Running
     │
     ▼
Unexpected Error
     │
     ▼
Process Stops
```

Production systems need a way to restart the application.

With Docker:

```text
API Container
      │
      ▼
    Crash
      │
      ▼
Restart Policy
      │
      ▼
Container Starts Again
```

Conceptually:

```text
Application
     │
     ├── Running
     │
     └── Crashed
            │
            ▼
        Restart
```

Your production system should not depend on someone manually logging into the server every time the application stops.

---

# 20. Health Checks

A running process does not always mean a healthy application.

Example:

```text
Container = Running ✅

But:

Database Connection = Failed ❌
```

The application may be running but unable to serve users correctly.

Health checks allow the application to expose its status.

Example:

```text
/health
```

The monitoring system can ask:

```text
GET /health
```

The application responds:

```text
Healthy
```

or:

```text
Unhealthy
```

Example architecture:

```text
Monitoring System
       │
       ▼
    /health
       │
       ▼
ASP.NET Core API
       │
       ├── Database Check
       ├── Redis Check
       └── Other Dependencies
```

---

# 21. Logging in Production

When something goes wrong, you need to know:

```text
What happened?
When?
Which request?
Which user?
Which service?
Why did it fail?
```

Your application generates logs.

Example:

```text
Request Started

GET /api/users

Database Query Executed

Request Completed
```

If an error happens:

```text
Unhandled Exception
       │
       ▼
Exception Details
       │
       ▼
Log System
```

Production logs should help you investigate problems without exposing sensitive information.

Never log:

```text
Passwords
JWT Tokens
Credit Card Data
Secrets
Private Keys
```

---

# 22. Monitoring

Logging tells you what happened.

Monitoring tells you how the system is behaving.

You may monitor:

```text
CPU Usage
Memory Usage
Disk Space
Application Errors
Response Time
Request Count
Database Performance
Container Status
Server Availability
```

Example:

```text
                    Monitoring

                         ▲
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼

      Server            API            Database
```

Monitoring should answer:

> Is the application working right now?

---

# 23. Production Errors

In development, you might see:

```text
Full Exception
Stack Trace
File Name
Line Number
```

In production, users should not see internal application details.

Bad:

```text
System.Data.SqlClient.SqlException

Connection failed at:

DatabaseService.cs line 42
```

Better:

```text
Something went wrong.
```

The real error goes to your logging system.

```text
User
  │
  ▼
Safe Error Response

Server
  │
  ▼
Detailed Log
```

---

# 24. Security in Production

A basic production server should consider:

```text
HTTPS
Firewall
SSH Security
Strong Authentication
Security Updates
Secrets Management
Database Access Control
Rate Limiting
Logging
Backups
```

A simplified architecture:

```text
Internet
   │
   ▼
Firewall
   │
   ▼
Nginx
   │
   ▼
Application
   │
   ▼
Private Database
```

The database should not accept arbitrary internet traffic.

Only required ports should be exposed.

For example:

```text
Public

80   → HTTP
443  → HTTPS
```

Internal services should remain private when possible.

---

# 25. HTTPS and SSL Certificates

HTTPS encrypts communication between users and your server.

Without HTTPS:

```text
User ───── HTTP ───── Server
```

With HTTPS:

```text
User ═══ Encrypted ═══ Server
```

The typical production setup:

```text
User
 │ HTTPS
 ▼
Nginx
 │ HTTP / Internal HTTPS
 ▼
ASP.NET Core API
```

The SSL certificate is commonly managed at the reverse proxy.

```text
Nginx
 ├── SSL Certificate
 └── HTTPS Configuration
```

HTTP requests can be redirected:

```text
http://myapp.com
        │
        ▼
301 / 308 Redirect
        │
        ▼
https://myapp.com
```

---

# 26. Server Access

You normally manage a Linux server remotely using SSH.

```text
Your Computer
      │
      │ SSH
      ▼
Linux Server
```

Example:

```text
ssh user@server-ip
```

After connecting:

```text
Linux Server

├── Application Files
├── Docker
├── Logs
├── Environment Variables
└── System Services
```

Production server access should be protected carefully.

Common practices include:

```text
SSH Keys
Limited User Access
Firewall Rules
No Shared Credentials
Security Updates
```

---

# 27. Deployment Process

A typical deployment process:

```text
Developer
    │
    ▼
Write Code
    │
    ▼
Git Commit
    │
    ▼
Git Push
    │
    ▼
Build Application
    │
    ▼
Create Docker Image
    │
    ▼
Deploy to Server
    │
    ▼
Start New Version
    │
    ▼
Health Check
    │
    ▼
Production
```

With CI/CD:

```text
Developer
    │
    │ 
  git push
    ▼
GitHub
    │
    ▼
CI/CD Pipeline
    │
    ├── Build
    ├── Test
    └── Deploy
            │
            ▼
     Production Server
```

---

# 28. Updating an Application

Suppose version 1 is running:

```text
API v1
   │
   ▼
Users
```

You create a new version:

```text
API v2
```

The deployment process might be:

```text
Pull New Code/Image
        │
        ▼
Build New Version
        │
        ▼
Start New Container
        │
        ▼
Check Health
        │
        ▼
Replace Old Version
```

The goal is to avoid:

```text
Deploy New Version
       │
       ▼
Application Broken ❌
       │
       ▼
Users Cannot Access System
```

A production deployment should always consider:

```text
What happens if deployment fails?
```

---

# 29. Rollback

A rollback means returning to the previous working version.

```text
Version 1 ✅

Deploy

Version 2 ❌

Rollback

Version 1 ✅
```

This is why versioning your deployments is important.

Conceptually:

```text
Image: myapp:1.0

Image: myapp:1.1

Image: myapp:1.2
```

If version `1.2` fails:

```text
Stop 1.2

Start 1.1
```

---

# 30. Production Architecture for Your .NET Application

A practical architecture for a .NET application could be:

```text
                         USERS
                           │
                           │ 
                          HTTPS
                           ▼
                        INTERNET
                           │
                           ▼
                           DNS
                           │
                           ▼
                     PUBLIC SERVER
                           │
                           ▼
                         NGINX
                           │
                           ▼
                    DOCKER NETWORK
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
         ASP.NET Core API       Background Worker
                │
                ▼
             DATABASE
                │
                ▼
          PERSISTENT VOLUME
```

Supporting services:

```text
Application
     │
     ├── Logs
     │
     ├── Monitoring
     │
     ├── Health Checks
     │
     └── Backups
```

---

# 31. A Real Production Server Structure

Conceptually:

```text
Production Server
│
├── Operating System
│
├── Docker
│   │
│   ├── API Container
│   │
│   ├── Worker Container
│   │
│   └── Other Services
│
├── Reverse Proxy
│
├── Firewall
│
├── Logs
│
├── Monitoring
│
└── Backup Process
```

---

# 32. Production Checklist

Before deploying:

```text
[ ] Application builds successfully
[ ] Production configuration is ready
[ ] Secrets are not committed to Git
[ ] Database connection is configured
[ ] HTTPS is enabled
[ ] Domain DNS is configured
[ ] Firewall is configured
[ ] Logs are available
[ ] Health checks exist
[ ] Restart policy exists
[ ] Database data is persistent
[ ] Backups are configured
[ ] Backup restore has been tested
[ ] Monitoring is configured
[ ] Deployment rollback is possible
```

---

# 🎯 The Most Important Mental Model

When deploying an application, think about the complete path.

```text
User
 │
 ▼
Domain
 │
 ▼
DNS
 │
 ▼
Public IP
 │
 ▼
Firewall
 │
 ▼
Nginx
 │
 ▼
ASP.NET Core Application
 │
 ▼
Database
```

And also think about what happens when something fails.

```text
Application Crashes
        │
        ▼
Restart

────────────────

Server Disk Fails
        │
        ▼
Restore Backup

────────────────

New Deployment Fails
        │
        ▼
Rollback

────────────────

Application Is Slow
        │
        ▼
Monitoring + Logs
```


---

# 🚀 Final Goal

The final goal is to take your application from:

```text
ASP.NET Core API

↓

Running on localhost
```

to:

```text
  Real Users
      │
      ▼
https://api.yourapp.com
      │
      ▼
     DNS
      │
      ▼
Production Linux Server
      │
      ▼
Firewall
      │
      ▼
Nginx
      │
      ▼
Docker
      │
      ▼
ASP.NET Core API
      │
      ▼
Production Database
      │
      ├── Persistent Storage
      ├── Backups
      ├── Logging
      └── Monitoring
```

The objective is not just to **deploy an application once**.

The objective is to understand how to **run, secure, update, monitor, recover, and maintain an application in production**.