# TaskBit - Technical Documentation Suite

## Overview
TaskBit is a modern project management and task tracking application with GitHub integration capabilities. This document provides comprehensive architectural and data flow documentation using Mermaid diagrams.

---

## 1. System Architecture Diagram

```mermaid
graph TB
    subgraph "Client Layer"
        WebApp["React 19 Frontend<br/>Vite | Tailwind CSS"]
        Redux["Redux Toolkit<br/>State Management"]
        Router["React Router<br/>Navigation"]
        Toast["React Hot Toast<br/>Notifications"]
    end

    subgraph "Authentication Layer"
        ClerkAuth["Clerk Authentication<br/>OAuth Provider"]
        AuthMiddleware["Auth Middleware<br/>Token Verification"]
    end

    subgraph "API Gateway & Server"
        Express["Express 5.1<br/>REST API"]
        CORS["CORS Configuration"]
        WebSocket["WebSocket Server<br/>Real-time Updates"]
    end

    subgraph "Business Logic Layer"
        WorkspaceCtrl["Workspace Controller<br/>Management & CRUD"]
        ProjectCtrl["Project Controller<br/>Creation & Updates"]
        TaskCtrl["Task Controller<br/>Assignment & Status"]
        CommentCtrl["Comment Controller<br/>Collaboration"]
        GitHubCtrl["GitHub Controller<br/>Integration Logic"]
        UserCtrl["User Controller<br/>Profile Management"]
    end

    subgraph "Data Access Layer"
        PrismaORM["Prisma ORM<br/>Query Builder"]
        Migrations["Database Migrations"]
    end

    subgraph "Database Layer"
        NeonDB["Neon Serverless<br/>PostgreSQL Database"]
    end

    subgraph "Third-Party Services"
        GitHubAPI["GitHub API<br/>Repository & Issues"]
        Nodemailer["Nodemailer<br/>Email Service"]
        Inngest["Inngest<br/>Background Jobs & Events"]
    end

    subgraph "Supporting Services"
        Multer["Multer<br/>File Upload"]
        DotEnv["Environment Config<br/>dotenv"]
    end

    WebApp --> Redux
    Redux --> Router
    Router --> Toast
    WebApp -->|HTTP/REST| Express
    WebApp -->|WebSocket| WebSocket
    ClerkAuth -->|Authenticate| AuthMiddleware
    AuthMiddleware -->|Verify| Express
    Express --> CORS
    Express -->|Route| WorkspaceCtrl
    Express -->|Route| ProjectCtrl
    Express -->|Route| TaskCtrl
    Express -->|Route| CommentCtrl
    Express -->|Route| GitHubCtrl
    Express -->|Route| UserCtrl
    WorkspaceCtrl --> PrismaORM
    ProjectCtrl --> PrismaORM
    TaskCtrl --> PrismaORM
    CommentCtrl --> PrismaORM
    GitHubCtrl --> PrismaORM
    UserCtrl --> PrismaORM
    PrismaORM -->|Query/Mutate| NeonDB
    Migrations -->|Schema| NeonDB
    GitHubCtrl -->|API Calls| GitHubAPI
    TaskCtrl -->|Trigger Events| Inngest
    Inngest -->|Send Email| Nodemailer
    Inngest -->|Sync Data| PrismaORM
    TaskCtrl --> Multer
    Express --> DotEnv

    style WebApp fill:#61dafb,color:#000
    style Express fill:#90c53f,color:#fff
    style NeonDB fill:#0066ff,color:#fff
    style ClerkAuth fill:#1f3a93,color:#fff
    style GitHubAPI fill:#333,color:#fff
    style Inngest fill:#f8f8f8,stroke:#333
```

---

## 2. Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    USER ||--o{ WORKSPACE : "owns"
    USER ||--o{ WORKSPACEMEMBER : "joins"
    WORKSPACE ||--o{ WORKSPACEMEMBER : "contains"
    WORKSPACE ||--o{ PROJECT : "contains"
    USER ||--o{ PROJECT : "leads"
    USER ||--o{ PROJECTMEMBER : "joins"
    PROJECT ||--o{ PROJECTMEMBER : "contains"
    PROJECT ||--o{ TASK : "contains"
    USER ||--o{ TASK : "assigns"
    USER ||--o{ COMMENT : "writes"
    TASK ||--o{ COMMENT : "has"
    PROJECT ||--o| PROJECTGITHUBINTEGRATION : "has"
    PROJECTGITHUBINTEGRATION ||--o{ GITHUBOAUTHSTATE : "generates"

    USER {
        string id PK
        string name
        string email UK
        string image
        string designation
        string department
        string about
        datetime createdAt
        datetime updatedAt
    }

    WORKSPACE {
        string id PK
        string name
        string slug UK
        string description
        json settings
        string ownerId FK
        string image_url
        datetime createdAt
        datetime updatedAt
    }

    WORKSPACEMEMBER {
        string id PK
        string userId FK
        string workspaceId FK
        string message
        enum role "ADMIN, MEMBER"
        unique "userId, workspaceId"
    }

    PROJECT {
        string id PK
        string name
        string description
        enum priority "LOW, MEDIUM, HIGH"
        enum status "ACTIVE, PLANNING, COMPLETED, ON_HOLD, CANCELLED"
        datetime start_date
        datetime end_date
        string team_lead FK
        string workspaceId FK
        int progress
        datetime createdAt
        datetime updatedAt
    }

    PROJECTMEMBER {
        string id PK
        string userId FK
        string projectId FK
        unique "userId, projectId"
    }

    TASK {
        string id PK
        string projectId FK
        string title
        string description
        enum status "TODO, IN_PROGRESS, DONE"
        enum type "TASK, BUG, FEATURE, IMPROVEMENT, OTHER"
        enum priority "LOW, MEDIUM, HIGH"
        string assigneeId FK
        datetime due_date
        int githubIssueNumber
        string githubIssueUrl
        string githubRepository
        datetime createdAt
        datetime updatedAt
    }

    COMMENT {
        string id PK
        string content
        string userId FK
        string taskId FK
        datetime createdAt
    }

    PROJECTGITHUBINTEGRATION {
        string id PK
        string projectId FK UK
        string githubAccountLogin
        string githubUserId
        string repository
        string accessToken
        string webhookSecret
        datetime webhookSecretUpdatedAt
        datetime createdAt
        datetime updatedAt
    }

    GITHUBOAUTHSTATE {
        string id PK
        string state UK
        string userId
        string projectId
        string integrationId FK
        datetime expiresAt
        datetime createdAt
    }
```

---

## 3. Data Flow Diagram - Level 0 (Context)

```mermaid
graph LR
    User["👤 User/Team Members"]
    GitHub["🐙 GitHub<br/>External API"]
    Email["📧 Email Service<br/>Nodemailer"]
    Admin["👨‍💼 Workspace Admin"]
    
    TaskBit["<b>TaskBit System</b><br/>Project & Task<br/>Management Platform"]
    
    User -->|Create/Manage Tasks<br/>View Projects<br/>Collaborate| TaskBit
    Admin -->|Manage Workspace<br/>Setup GitHub<br/>Invite Members| TaskBit
    GitHub -->|Fetch Repos<br/>Webhook Events<br/>Issue Updates| TaskBit
    TaskBit -->|Sync Issues<br/>Create Tasks| GitHub
    TaskBit -->|Send Notifications<br/>Task Reminders<br/>Invitations| Email
    
    style TaskBit fill:#4a5568,color:#fff,stroke:#2d3748,stroke-width:3px
    style User fill:#48bb78,color:#fff
    style Admin fill:#ed8936,color:#fff
    style GitHub fill:#2d3748,color:#fff
    style Email fill:#9f7aea,color:#fff
```

---

## 4. Data Flow Diagram - Level 1 (Functional)

```mermaid
graph LR
    User["User/Team Member"]
    Admin["Admin/Project Lead"]
    GitHubExt["GitHub API"]
    EmailSvc["Email Service"]
    
    subgraph "Frontend Application"
        Auth["Authentication<br/>Clerk OAuth"]
        Dashboard["Dashboard<br/>& UI Components"]
        Forms["Forms<br/>Create/Edit"]
    end
    
    subgraph "Core Modules"
        AuthMod["Auth Module<br/>Token Verification"]
        WorkspaceMod["Workspace Module<br/>Organization Management"]
        ProjectMod["Project Module<br/>Project CRUD"]
        TaskMod["Task Module<br/>Task Management"]
        CommentMod["Collaboration Module<br/>Comments & Discussion"]
        GitHubMod["GitHub Module<br/>Integration & Sync"]
        UserMod["User Module<br/>Profile & Settings"]
    end
    
    subgraph "Data Layer"
        Database["PostgreSQL Database<br/>Neon Serverless"]
    end
    
    subgraph "Background Services"
        Inngest["Event Queue<br/>Inngest"]
        Jobs["Background Jobs<br/>Email, Sync, Notifications"]
    end
    
    User -->|Login| Auth
    Auth -->|Verify| AuthMod
    User -->|Browse/Create| Dashboard
    Dashboard -->|Submit Data| Forms
    Forms -->|API Requests| ProjectMod
    Forms -->|API Requests| TaskMod
    Forms -->|API Requests| CommentMod
    Admin -->|Setup| GitHubMod
    Admin -->|Manage| WorkspaceMod
    
    AuthMod --> Database
    WorkspaceMod --> Database
    ProjectMod --> Database
    TaskMod --> Database
    CommentMod --> Database
    GitHubMod --> Database
    UserMod --> Database
    
    TaskMod -->|Emit Event| Inngest
    Inngest --> Jobs
    Jobs -->|Send Email| EmailSvc
    Jobs -->|Sync Data| Database
    
    GitHubMod -->|Fetch Data| GitHubExt
    GitHubMod -->|Create Issues| GitHubExt
    GitHubExt -->|Webhook Events| GitHubMod
    
    style Frontend fill:#61dafb,stroke:#333
    style Database fill:#0066ff,color:#fff
    style GitHubExt fill:#333,color:#fff
    style EmailSvc fill:#9f7aea,color:#fff
    style Inngest fill:#ed8936,color:#fff
```

---

## 5. Data Flow Diagram - Level 2 (Detailed: GitHub Integration + Project Management)

```mermaid
graph TD
    UserInit["User/Admin Initiates<br/>GitHub Setup"]
    
    subgraph "OAuth Flow"
        ReqAuth["1. Request GitHub Auth<br/>Generate State Token"]
        AuthPage["2. Redirect to GitHub<br/>Authorization Page"]
        UserAuth["3. User Grants Permissions<br/>GitHub Returns Code"]
        CallbackHandle["4. Handle Callback<br/>Validate State Token"]
        ExchangeToken["5. Exchange Code<br/>for Access Token"]
        StoreToken["6. Store Token<br/>& Integration Data"]
    end
    
    subgraph "Webhook Setup"
        GetRepos["7. Fetch User Repositories<br/>via GitHub API"]
        SelectRepo["8. User Selects Repository"]
        RegWebhook["9. Register Webhook<br/>with GitHub"]
        GenSecret["10. Generate Webhook Secret<br/>Store in DB"]
    end
    
    subgraph "Webhook Event Handler"
        WebhookRecv["11. GitHub sends Webhook<br/>Issue/PR Event"]
        VerifySignature["12. Verify Webhook<br/>HMAC Signature"]
        ExtractData["13. Extract Issue Data<br/>and Metadata"]
        ExtractTaskId["14. Extract Task ID<br/>from Description"]
    end
    
    subgraph "Task Sync Logic"
        CheckTask["15. Check if Task<br/>Exists in DB"]
        TaskFound{"Task ID<br/>Found?"}
        CreateNewTask["16a. Create New Task<br/>in Project"]
        UpdateTask["16b. Update Existing<br/>Task Status"]
        LinkIssue["17. Link GitHub Issue<br/>to TaskBit Task"]
    end
    
    subgraph "Notification & Data Store"
        TriggerEvent["18. Trigger Event<br/>Task Updated/Created"]
        SaveIntegration["19. Save Integration<br/>Metadata to DB"]
        NotifyUser["20. Emit WebSocket<br/>Real-time Update"]
        SendEmail["21. Send Email<br/>via Inngest"]
    end
    
    UserInit --> ReqAuth
    ReqAuth --> AuthPage
    AuthPage --> UserAuth
    UserAuth --> CallbackHandle
    CallbackHandle --> ExchangeToken
    ExchangeToken --> StoreToken
    StoreToken --> GetRepos
    GetRepos --> SelectRepo
    SelectRepo --> RegWebhook
    RegWebhook --> GenSecret
    GenSecret -->|Ready| WebhookRecv
    WebhookRecv --> VerifySignature
    VerifySignature -->|Valid| ExtractData
    ExtractData --> ExtractTaskId
    ExtractTaskId --> CheckTask
    CheckTask --> TaskFound
    TaskFound -->|Yes| UpdateTask
    TaskFound -->|No| CreateNewTask
    UpdateTask --> LinkIssue
    CreateNewTask --> LinkIssue
    LinkIssue --> TriggerEvent
    TriggerEvent --> SaveIntegration
    SaveIntegration --> NotifyUser
    SaveIntegration --> SendEmail
    
    style OAuth fill:#e6f3ff,stroke:#0066ff,stroke-width:2px
    style WebhookSetup fill:#ffe6f0,stroke:#d81b60,stroke-width:2px
    style WebhookEventHandler fill:#f0e6ff,stroke:#7c3aed,stroke-width:2px
    style TaskSyncLogic fill:#e6ffe6,stroke:#22c55e,stroke-width:2px
    style NotificationDataStore fill:#fff9e6,stroke:#f59e0b,stroke-width:2px
```

---

## Key Components Reference

### Frontend Routes (React Router)
- `/` - Dashboard
- `/team` - Team Management
- `/projects` - Projects List
- `/projectsDetail` - Project Details & Tasks
- `/taskDetails` - Task Details & Comments
- `/github/callback` - GitHub OAuth Callback Handler

### Backend API Endpoints
- `POST/GET/PUT/DELETE /api/workspaces` - Workspace CRUD
- `POST/GET/PUT/DELETE /api/projects` - Project CRUD
- `POST/GET/PUT/DELETE /api/tasks` - Task CRUD
- `POST/GET/PUT/DELETE /api/comments` - Comments
- `GET/POST /api/github/*` - GitHub Integration
- `POST /api/github/webhook/:projectId` - GitHub Webhooks (Raw Body)
- `GET/PUT /api/users` - User Profile

### Background Jobs (Inngest)
1. `sync-user-from-clerk` - Sync new user creation
2. `delete-user-with-clerk` - Sync user deletion
3. `update-user-from-clerk` - Sync user updates
4. `sync-workspace-from-clerk` - Sync workspace creation
5. `update-workspace-from-clerk` - Sync workspace updates
6. `delete-workspace-with-clerk` - Sync workspace deletion
7. `sync-workspace-member-from-clerk` - Sync member invitations
8. `send-task-assignment-mail` - Email + Reminder workflow

### Technology Stack Summary

| Layer | Technology |
|-------|-----------|
| **Frontend** | React 19, Vite, Redux Toolkit, Tailwind CSS, Axios |
| **Backend** | Express 5.1, Node.js |
| **Database** | PostgreSQL (Neon Serverless) |
| **ORM** | Prisma 6.17 |
| **Authentication** | Clerk (OAuth) |
| **Real-time** | WebSocket (ws) |
| **Background Jobs** | Inngest |
| **Email** | Nodemailer |
| **File Upload** | Multer |
| **External APIs** | GitHub API v2022-11-28 |
| **UI Components** | Lucide React, Recharts |
| **Deployment** | Vercel (Client & Server) |

### Enums & Constants

**Task Status**: TODO, IN_PROGRESS, DONE

**Task Type**: TASK, BUG, FEATURE, IMPROVEMENT, OTHER

**Priority**: LOW, MEDIUM, HIGH

**Project Status**: ACTIVE, PLANNING, COMPLETED, ON_HOLD, CANCELLED

**Workspace Role**: ADMIN, MEMBER

---

## Architecture Highlights

1. **Microservices-Ready**: Modular structure with separate concerns (Auth, Projects, Tasks, GitHub)
2. **Event-Driven**: Uses Inngest for asynchronous task processing and event handling
3. **Real-time Capabilities**: WebSocket integration for live updates
4. **OAuth-First Auth**: Clerk provides enterprise-grade authentication
5. **Serverless Database**: Neon provides scalable PostgreSQL
6. **GitHub-First Integration**: Deep GitHub integration with webhooks and OAuth
7. **Email Automation**: Inngest-driven email notifications with reminders
8. **Type Safety**: Prisma ensures type-safe database operations
9. **API Gateway Pattern**: Express serves as the API gateway
10. **State Management**: Redux Toolkit for predictable client-side state

---

## Data Flow Summary

1. **User Authentication** → Clerk OAuth → Session Token
2. **Create Project** → API Request → Prisma Query → PostgreSQL → Emit Event
3. **Assign Task** → Task Controller → Trigger Inngest Event → Email + Notification
4. **GitHub Integration Setup** → OAuth Flow → Store Credentials → Register Webhook
5. **GitHub Webhook Event** → Verify Signature → Extract Data → Create/Update Task → Notify User
6. **Real-time Updates** → WebSocket → Redux Update → UI Re-render

---

## Security Considerations

- ✅ CORS enabled for cross-origin requests
- ✅ Clerk middleware for authentication verification
- ✅ GitHub webhook signature verification (HMAC)
- ✅ Role-based access control (ADMIN/MEMBER)
- ✅ Project-level permission checks
- ✅ Environment variable configuration (no hardcoded secrets)
- ✅ Prisma prevents SQL injection via parameterized queries
- ✅ OAuth tokens stored securely

---

*Documentation Generated: 2026-04-23*
*TaskBit - Modern Project Management Platform*
