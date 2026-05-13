⚖️ Trade-off: We're using a named Docker volume (sqlserver_data) instead of a bind mount. This means your database data persists across docker compose down and up cycles. If you used docker compose down -v it would wipe it — useful when you want a clean slate, dangerous if you forget the -v flag.

on MacBook Air 
git pull                  # get latest code
docker compose up -d      # start SQL Server
dotnet run                # start the API



# Steps
## step 1 - pull SQL Server via Docker
    docker pull mcr.microsoft.com/mssql/server:2022-latest

## Step 2 - Create the solution structure 
# Make sure you're inside your cloned repo
cd servicedesk-pro

# Create the solution file
dotnet new sln -n ServiceDesk

# Create the four projects
dotnet new webapi -n ServiceDesk.API -o src/ServiceDesk.API --no-openapi
dotnet new classlib -n ServiceDesk.Domain -o src/ServiceDesk.Domain
dotnet new classlib -n ServiceDesk.Application -o src/ServiceDesk.Application
dotnet new classlib -n ServiceDesk.Infrastructure -o src/ServiceDesk.Infrastructure
dotnet new nunit -n ServiceDesk.Tests -o src/ServiceDesk.Tests

ServiceDesk.Domain
The heart of the system. No dependencies on anything else — not even a NuGet package if we can help it. Contains:

Entities — Ticket, User, Team, Department
Enums — TicketStatus, TicketPriority, UserRole
Business rules — the ticket state machine lives here

This project should be explainable to a non-developer. It's pure business logic with zero framework noise.

ServiceDesk.Application
Orchestrates the use cases. Knows what needs to happen but not how (no database code, no HTTP code). Contains:

Commands and Queries — CreateTicket, ApproveTicket, GetInbox
Interfaces — ITicketRepository, IEmailService, IStorageService
DTOs — what gets returned to the API
Validators — FluentValidation rules


Depends on Domain only. If Domain is the "what," Application is the "when and in what order."


ServiceDesk.Infrastructure
The "how." Implements everything the Application layer defined as interfaces. Contains:

EF Core DbContext and repositories
JWT token service
Email service (MailKit)
Storage service (S3 or Azure Blob — decided later)
Hangfire background jobs

Depends on Application and Domain. This is the only project allowed to talk to external systems.

ServiceDesk.API
The entry point. Thin layer — no business logic here. Contains:

Controllers — receive HTTP requests, call MediatR, return responses
Middleware — global exception handling, request logging
SignalR Hubs — real-time notifications
Program.cs — wires everything together


Depends on Application and Infrastructure. Controllers should be so thin you could swap them for a CLI or gRPC with minimal effort.