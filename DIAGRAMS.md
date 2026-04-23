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






# TaskBit - Architecture Diagrams in Mermaid Format

## 1. Overall System Architecture

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

## 2. Three-Tier Client-Server Architecture

```mermaid
graph TB
    subgraph "Presentation Tier"
        Browser["Web Browser"]
        UI["React UI Components<br/>Vite + Tailwind CSS"]
        State["Redux State<br/>Management"]
        Router["React Router<br/>Navigation"]
    end

    subgraph "Application Tier"
        Gateway["API Gateway<br/>Express.js"]
        Auth["Authentication<br/>Clerk OAuth"]
        WS["WebSocket Server<br/>Real-time Updates"]
        
        subgraph "Controllers"
            WC["Workspace<br/>Controller"]
            PC["Project<br/>Controller"]
            TC["Task<br/>Controller"]
            CC["Comment<br/>Controller"]
            GC["GitHub<br/>Controller"]
        end
        
        subgraph "Services"
            WS_Service["Workspace<br/>Service"]
            PS["Project<br/>Service"]
            TS["Task<br/>Service"]
            GHS["GitHub<br/>Service"]
        end
    end

    subgraph "Data Tier"
        ORM["Prisma ORM<br/>Data Access"]
        Cache["Redis Cache<br/>Session Store"]
        DB["PostgreSQL<br/>Neon Database"]
        Queue["Inngest<br/>Job Queue"]
    end

    subgraph "External Services"
        GitHub["GitHub API"]
        Email["Nodemailer<br/>SMTP"]
    end

    Browser --> UI
    UI --> State
    State --> Router
    Router -->|API Calls| Gateway
    Browser -->|WebSocket| WS
    
    Gateway --> Auth
    Gateway --> WC
    Gateway --> PC
    Gateway --> TC
    Gateway --> CC
    Gateway --> GC
    
    WC --> WS_Service
    PC --> PS
    TC --> TS
    GC --> GHS
    
    WS_Service --> ORM
    PS --> ORM
    TS --> ORM
    GHS --> ORM
    
    ORM --> Cache
    ORM --> DB
    
    TS -->|Emit Events| Queue
    Queue --> Email
    Queue --> ORM
    
    GHS --> GitHub
    GitHub -->|Webhooks| Gateway

    style Browser fill:#e8f4f8,stroke:#333
    style UI fill:#61dafb,color:#000,stroke:#333
    style State fill:#764abc,color:#fff,stroke:#333
    style Gateway fill:#90c53f,color:#fff,stroke:#333
    style Auth fill:#1f3a93,color:#fff,stroke:#333
    style ORM fill:#5b4b8a,color:#fff,stroke:#333
    style DB fill:#0066ff,color:#fff,stroke:#333
    style Queue fill:#ed8936,color:#fff,stroke:#333
```

---

## 3. Component Hierarchy & Data Flow

```mermaid
graph TD
    App["App Component<br/>Root"]
    
    App --> Dashboard["Dashboard<br/>Main Container"]
    App --> ProjectDetail["Project Details<br/>Container"]
    App --> TaskDetail["Task Details<br/>Container"]
    
    Dashboard --> ProjectsList["Projects List<br/>Component"]
    Dashboard --> StatsGrid["Stats Grid<br/>Component"]
    Dashboard --> RecentActivity["Recent Activity<br/>Component"]
    
    ProjectDetail --> ProjectHeader["Project Header<br/>Component"]
    ProjectDetail --> Tabs["Tab Navigation<br/>Component"]
    
    Tabs --> KanbanBoard["Kanban Board<br/>Component"]
    Tabs --> TasksTable["Tasks Table<br/>Component"]
    Tabs --> Calendar["Project Calendar<br/>Component"]
    Tabs --> Analytics["Analytics Dashboard<br/>Component"]
    Tabs --> Settings["Project Settings<br/>Component"]
    
    TaskDetail --> TaskInfo["Task Information<br/>Component"]
    TaskDetail --> Comments["Comments Section<br/>Component"]
    TaskDetail --> GitHubLink["GitHub Link<br/>Component"]
    
    ProjectsList -->|Fetch Data| Redux
    StatsGrid -->|Read State| Redux
    KanbanBoard -->|Update State| Redux
    Comments -->|Dispatch Action| Redux
    
    Redux -->|HTTP| API
    API --> Controllers
    Controllers --> Database
    
    Comments -->|WebSocket| WS
    KanbanBoard -->|WebSocket| WS
    WS -->|Broadcast| Redux

    style App fill:#61dafb,color:#000,stroke:#333,stroke-width:2px
    style Redux fill:#764abc,color:#fff,stroke:#333
    style API fill:#90c53f,color:#fff,stroke:#333
    style Database fill:#0066ff,color:#fff,stroke:#333
    style WS fill:#ed8936,color:#fff,stroke:#333
```

---

## 4. Data Model Relationships

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

## 5. Request-Response Flow

```mermaid
sequenceDiagram
    participant User as User<br/>Browser
    participant Frontend as React<br/>Frontend
    participant Redux as Redux<br/>Store
    participant Clerk as Clerk<br/>Auth
    participant API as Express<br/>API
    participant DB as PostgreSQL<br/>Database
    participant WS as WebSocket<br/>Server

    User->>Frontend: Click "Create Task"
    Frontend->>Redux: Dispatch createTask
    Redux->>Frontend: Update UI (loading state)
    Frontend->>API: POST /api/tasks (with token)
    
    API->>Clerk: Verify JWT token
    Clerk-->>API: Token valid ✓
    
    API->>DB: INSERT task record
    DB-->>API: Task created
    
    API-->>Frontend: Return task data
    Frontend->>Redux: Dispatch taskCreated
    Redux->>Frontend: Update store
    Frontend->>User: Show success toast
    
    API->>WS: Broadcast taskCreated event
    WS->>Frontend: Send WebSocket message
    Frontend->>Redux: Update other connected clients
    Redux->>User: Real-time task update
```

---

## 6. GitHub Integration Flow

```mermaid
graph TD
    A["User Opens Project Settings"] --> B["Click Connect GitHub"]
    B --> C["Generate OAuth State Token"]
    C --> D["Redirect to GitHub Auth URL"]
    D --> E["User Grants Permissions"]
    E --> F["GitHub Returns Authorization Code"]
    F --> G["Server Exchanges Code for Access Token"]
    G --> H["Store Token in Database"]
    H --> I["Fetch User's Repositories"]
    I --> J["Display Repository List"]
    J --> K["User Selects Repository"]
    K --> L["Generate Webhook Secret"]
    L --> M["Create Webhook in GitHub"]
    M --> N["GitHub Sends Webhook Events"]
    N --> O["Server Receives Issue Event"]
    O --> P["Verify Webhook Signature"]
    P --> Q{Signature Valid?}
    Q -->|No| R["Reject Request"]
    Q -->|Yes| S["Extract Issue Data"]
    S --> T["Map Issue State to Task Status"]
    T --> U{Task Exists?}
    U -->|Yes| V["Update Task Status"]
    U -->|No| W["Create New Task"]
    V --> X["Emit WebSocket Event"]
    W --> X
    X --> Y["Broadcast to Connected Clients"]
    Y --> Z["Update UI in Real-time"]
    Z --> AA["Send Email Notification"]

    style A fill:#e8f4f8,stroke:#333
    style G fill:#90c53f,color:#fff,stroke:#333
    style H fill:#0066ff,color:#fff,stroke:#333
    style N fill:#333,color:#fff,stroke:#333
    style P fill:#764abc,color:#fff,stroke:#333
    style X fill:#ed8936,color:#fff,stroke:#333
    style AA fill:#5b4b8a,color:#fff,stroke:#333
```

---

## 7. Real-time Update Architecture

```mermaid
graph LR
    subgraph "User 1 Session"
        U1["User 1<br/>Browser"]
        WS1["WebSocket<br/>Connection"]
    end

    subgraph "User 2 Session"
        U2["User 2<br/>Browser"]
        WS2["WebSocket<br/>Connection"]
    end

    subgraph "Server"
        Server["Express Server"]
        WSServer["WebSocket<br/>Server"]
        Redux["Redux Store"]
        DB["Database"]
    end

    U1 -->|Update Task Status| Redux
    Redux -->|Save to DB| DB
    Redux -->|Broadcast Event| WSServer
    
    WSServer -->|Send Update| WS1
    WSServer -->|Send Update| WS2
    
    WS1 -->|WebSocket Event| U1
    WS2 -->|WebSocket Event| U2
    
    U1 -->|Refresh UI| U1
    U2 -->|Refresh UI| U2

    style U1 fill:#61dafb,color:#000,stroke:#333
    style U2 fill:#61dafb,color:#000,stroke:#333
    style WSServer fill:#ed8936,color:#fff,stroke:#333
    style Redux fill:#764abc,color:#fff,stroke:#333
    style DB fill:#0066ff,color:#fff,stroke:#333
```

---

## 8. Authentication Flow

```mermaid
graph TD
    A["User Visits Login Page"] --> B["Click 'Sign in with Google/GitHub'"]
    B --> C["Redirect to Clerk<br/>Authorization Page"]
    C --> D["User Authenticates"]
    D --> E["Clerk Returns JWT Token"]
    E --> F["Store Token in Cookie<br/>httpOnly"]
    F --> G["Redirect to Dashboard"]
    G --> H["Include Token in API Requests"]
    H --> I["Express Server<br/>Receives Request"]
    I --> J["Clerk Middleware<br/>Validates Token"]
    J --> K{Token Valid?}
    K -->|No| L["Return 401 Unauthorized"]
    K -->|Yes| M["Extract User ID"]
    M --> N["Attach to req.auth"]
    N --> O["Route Handler Processes"]
    O --> P["Database Query with User ID"]
    P --> Q["Return Data"]
    Q --> R["Send Response to Frontend"]
    R --> S["Update Redux Store"]
    S --> T["Display User Dashboard"]

    style C fill:#1f3a93,color:#fff,stroke:#333
    style F fill:#90c53f,color:#fff,stroke:#333
    style J fill:#764abc,color:#fff,stroke:#333
    style K fill:#ed8936,color:#fff,stroke:#333
    style P fill:#0066ff,color:#fff,stroke:#333
    style T fill:#61dafb,color:#000,stroke:#333
```

---

## 9. Background Job Processing

```mermaid
graph TD
    A["Task Updated"] --> B["Emit Event to Inngest"]
    B --> C["Event Queue"]
    C --> D["Task 1: Send Email"]
    C --> E["Task 2: Update GitHub"]
    C --> F["Task 3: Update Analytics"]
    C --> G["Task 4: Audit Log"]
    
    D --> D1["Fetch Task Details"]
    D1 --> D2["Fetch Assignee Email"]
    D2 --> D3["Render Email Template"]
    D3 --> D4["Send via Nodemailer"]
    D4 --> D5["Log Email Sent"]
    
    E --> E1["Check GitHub Integration"]
    E1 --> E2["Extract Issue Number"]
    E2 --> E3["Call GitHub API"]
    E3 --> E4["Update Issue State"]
    E4 --> E5["Log Update"]
    
    F --> F1["Calculate Progress"]
    F1 --> F2["Update Project Metrics"]
    F2 --> F3["Store in Cache"]
    F3 --> F4["Emit Analytics Event"]
    
    D5 --> H["Completed"]
    E5 --> H
    F4 --> H
    G --> H

    style B fill:#ed8936,color:#fff,stroke:#333
    style C fill:#ed8936,color:#fff,stroke:#333
    style D fill:#90c53f,color:#fff,stroke:#333
    style E fill:#333,color:#fff,stroke:#333
    style F fill:#764abc,color:#fff,stroke:#333
    style H fill:#22c55e,color:#fff,stroke:#333
```

---

## 10. Deployment Architecture

```mermaid
graph TB
    subgraph "Development Environment"
        DevClient["React Dev Server<br/>localhost:5173"]
        DevServer["Node Dev Server<br/>localhost:5000"]
        DevDB["Local PostgreSQL<br/>Docker Container"]
    end

    subgraph "Production Environment on Vercel"
        subgraph "Frontend Deployment"
            CDN["Vercel CDN<br/>Global Edge Network"]
            FrontendBuild["Next.js/React Build<br/>Static + Dynamic"]
        end
        
        subgraph "Backend Deployment"
            ServerlessFunc["Vercel Serverless<br/>Functions"]
            ExpressServer["Express API<br/>Auto-scaling"]
        end
        
        subgraph "Database"
            NeonDB["Neon PostgreSQL<br/>Serverless Cloud"]
            Backup["Automated Backups<br/>& Replication"]
        end
    end

    subgraph "External Services"
        GitHub["GitHub API<br/>OAuth & Webhooks"]
        Clerk["Clerk Auth<br/>User Management"]
        Email["Nodemailer<br/>SMTP Service"]
        Inngest["Inngest Cloud<br/>Background Jobs"]
    end

    DevClient -->|Build| FrontendBuild
    DevServer -->|Deploy| ExpressServer
    DevDB -->|Migrate Schema| NeonDB
    
    FrontendBuild -->|Cache| CDN
    ExpressServer -->|Query| NeonDB
    NeonDB -->|Backup| Backup
    
    ExpressServer -->|OAuth Flow| Clerk
    ExpressServer -->|API Calls| GitHub
    ExpressServer -->|Events| Inngest
    Inngest -->|Send Emails| Email
    
    CDN -->|Serve| User["End Users<br/>Global Access"]
    ExpressServer -->|REST API| User
    User -->|WebSocket| ExpressServer

    style CDN fill:#61dafb,color:#000,stroke:#333
    style ExpressServer fill:#90c53f,color:#fff,stroke:#333
    style NeonDB fill:#0066ff,color:#fff,stroke:#333
    style Clerk fill:#1f3a93,color:#fff,stroke:#333
    style Inngest fill:#ed8936,color:#fff,stroke:#333
```

---

## 11. Microservices-Style Module Architecture

```mermaid
graph TB
    API["API Gateway<br/>Express.js<br/>Port: 5000"]
    
    API --> AuthModule["🔐 Authentication Module<br/>Clerk OAuth<br/>Token Validation"]
    API --> WorkspaceModule["🏢 Workspace Module<br/>Organization Management<br/>Multi-tenancy"]
    API --> ProjectModule["📊 Project Module<br/>Project CRUD<br/>Status Tracking"]
    API --> TaskModule["✓ Task Module<br/>Task Management<br/>Assignment"]
    API --> CommentModule["💬 Comment Module<br/>Collaboration<br/>Threading"]
    API --> GitHubModule["🐙 GitHub Module<br/>OAuth Integration<br/>Webhook Handler"]
    API --> UserModule["👤 User Module<br/>Profile Management<br/>Preferences"]
    
    AuthModule --> Database["PostgreSQL<br/>Neon Serverless"]
    WorkspaceModule --> Database
    ProjectModule --> Database
    TaskModule --> Database
    CommentModule --> Database
    GitHubModule --> Database
    UserModule --> Database
    
    TaskModule --> Inngest["Inngest<br/>Event Queue"]
    WorkspaceModule --> Inngest
    
    GitHubModule --> GitHubAPI["GitHub API"]
    Inngest --> Email["Nodemailer<br/>Email Service"]
    
    TaskModule --> WebSocket["WebSocket Server<br/>Real-time Updates"]
    CommentModule --> WebSocket
    
    Database --> Cache["Redis Cache<br/>Session Store"]

    style API fill:#90c53f,color:#fff,stroke:#333,stroke-width:2px
    style AuthModule fill:#1f3a93,color:#fff,stroke:#333
    style WorkspaceModule fill:#764abc,color:#fff,stroke:#333
    style ProjectModule fill:#0066ff,color:#fff,stroke:#333
    style TaskModule fill:#22c55e,color:#fff,stroke:#333
    style CommentModule fill:#ed8936,color:#fff,stroke:#333
    style GitHubModule fill:#333,color:#fff,stroke:#333
    style Database fill:#0066ff,color:#fff,stroke:#333,stroke-width:2px
    style Inngest fill:#ed8936,color:#fff,stroke:#333
    style WebSocket fill:#764abc,color:#fff,stroke:#333
```

---

## 12. Error Handling & Recovery Flow

```mermaid
graph TD
    A["API Request"] --> B{Validation<br/>Success?}
    B -->|No| C["Return 400<br/>Bad Request"]
    B -->|Yes| D{Authentication<br/>Success?}
    
    D -->|No| E["Return 401<br/>Unauthorized"]
    D -->|Yes| F{Authorization<br/>Check}
    
    F -->|No| G["Return 403<br/>Forbidden"]
    F -->|Yes| H{Database<br/>Query}
    
    H -->|Error| I["Rollback Transaction"]
    I --> J["Log Error"]
    J --> K["Return 500<br/>Server Error"]
    
    H -->|Success| L["Return 200<br/>OK with Data"]
    
    C --> M["Frontend Error Handler"]
    E --> M
    G --> M
    K --> M
    L --> N["Success Handler<br/>Update Redux"]
    
    M --> O["Display Error Toast"]
    N --> P["Display Success Toast"]
    O --> Q["Log to Analytics"]
    P --> Q

    style B fill:#ed8936,color:#fff,stroke:#333
    style D fill:#ed8936,color:#fff,stroke:#333
    style F fill:#ed8936,color:#fff,stroke:#333
    style C fill:#dc2626,color:#fff,stroke:#333
    style E fill:#dc2626,color:#fff,stroke:#333
    style G fill:#dc2626,color:#fff,stroke:#333
    style K fill:#dc2626,color:#fff,stroke:#333
    style L fill:#22c55e,color:#fff,stroke:#333
```

---


mermaid

## 1. USE CASE DIAGRAM

```mermaid
graph TB
    User["👤 User"]
    Admin["👨‍💼 Admin"]
    GitHubUser["🐙 GitHub User"]
    System["TaskBit System"]

    User -->|View Dashboard| System
    User -->|Create/Manage Projects| System
    User -->|Create/Update Tasks| System
    User -->|Add Comments| System
    User -->|View Analytics| System
    User -->|Search Tasks| System
    User -->|View Calendar| System
    User -->|View Kanban Board| System
    
    Admin -->|Manage Workspace| System
    Admin -->|Invite Members| System
    Admin -->|Set Permissions| System
    Admin -->|Configure GitHub Integration| System
    Admin -->|View Reports| System
    
    GitHubUser -->|Trigger Webhooks| System
    System -->|Sync Issues| GitHubUser
    System -->|Update Repositories| GitHubUser

    style System fill:#e8f4f8,stroke:#333,stroke-width:2px
    style Admin fill:#1f3a93,color:#fff
    style GitHubUser fill:#333,color:#fff
```

---

## 2. COMPONENT DIAGRAM

```mermaid
graph TB
    subgraph "Frontend Components"
        Auth["Authentication<br/>Component"]
        Dashboard["Dashboard<br/>Component"]
        ProjectList["Project List<br/>Component"]
        ProjectDetail["Project Detail<br/>Component"]
        TaskForm["Task Form<br/>Component"]
        KanbanBoard["Kanban Board<br/>Component"]
        Calendar["Calendar<br/>Component"]
        Analytics["Analytics<br/>Component"]
        Comments["Comments<br/>Component"]
    end

    subgraph "Backend Components"
        AuthCtrl["Auth<br/>Controller"]
        WorkspaceCtrl["Workspace<br/>Controller"]
        ProjectCtrl["Project<br/>Controller"]
        TaskCtrl["Task<br/>Controller"]
        CommentCtrl["Comment<br/>Controller"]
        GitHubCtrl["GitHub<br/>Controller"]
        UserCtrl["User<br/>Controller"]
    end

    subgraph "Data Layer Components"
        PrismaORM["Prisma<br/>ORM"]
        PostgreSQL["PostgreSQL<br/>Database"]
        RedisCache["Redis<br/>Cache"]
    end

    subgraph "External Components"
        ClerkAuth["Clerk<br/>Authentication"]
        GitHubAPI["GitHub<br/>API"]
        Nodemailer["Nodemailer<br/>Email"]
        Inngest["Inngest<br/>Events"]
    end

    Auth -->|Uses| AuthCtrl
    Dashboard -->|Uses| WorkspaceCtrl
    ProjectList -->|Uses| ProjectCtrl
    ProjectDetail -->|Uses| ProjectCtrl
    TaskForm -->|Uses| TaskCtrl
    KanbanBoard -->|Uses| TaskCtrl
    Calendar -->|Uses| TaskCtrl
    Analytics -->|Uses| TaskCtrl
    Comments -->|Uses| CommentCtrl

    AuthCtrl -->|Query| PrismaORM
    WorkspaceCtrl -->|Query| PrismaORM
    ProjectCtrl -->|Query| PrismaORM
    TaskCtrl -->|Query| PrismaORM
    CommentCtrl -->|Query| PrismaORM
    GitHubCtrl -->|Query| PrismaORM

    PrismaORM -->|Access| PostgreSQL
    PrismaORM -->|Cache| RedisCache

    AuthCtrl -->|Verify| ClerkAuth
    GitHubCtrl -->|Call| GitHubAPI
    TaskCtrl -->|Event| Inngest
    Inngest -->|Send| Nodemailer

    style Frontend fill:#61dafb,color:#000,stroke:#333
    style Backend fill:#90c53f,color:#fff,stroke:#333
    style DataLayer fill:#0066ff,color:#fff,stroke:#333
    style External fill:#333,color:#fff,stroke:#333
```

---

## 3. DEPLOYMENT DIAGRAM

```mermaid
graph TB
    subgraph "Developer Machine"
        LocalClient["React Dev Server<br/>localhost:5173"]
        LocalServer["Node Server<br/>localhost:5000"]
        LocalDB["Docker PostgreSQL<br/>localhost:5432"]
    end

    subgraph "Vercel Cloud Platform"
        subgraph "Frontend Deployment"
            CDNNA["Vercel CDN<br/>North America"]
            CDNEU["Vercel CDN<br/>Europe"]
            CDNAP["Vercel CDN<br/>Asia-Pacific"]
        end

        subgraph "Backend Deployment"
            ServerNA["Serverless Functions<br/>North America"]
            ServerEU["Serverless Functions<br/>Europe"]
            ServerAP["Serverless Functions<br/>Asia-Pacific"]
        end
    end

    subgraph "Cloud Services"
        NeonDB["Neon PostgreSQL<br/>Managed Database"]
        NeonBackup["Automated Backups<br/>& Replicas"]
        RedisCloud["Redis Cloud<br/>Cache"]
    end

    subgraph "Third-Party Services"
        Clerk["Clerk Auth<br/>SaaS"]
        GitHub["GitHub<br/>Repository"]
        Inngest["Inngest<br/>Events Platform"]
        Sendgrid["SendGrid<br/>Email"]
    end

    LocalClient -->|Deploy| CDNNA
    LocalServer -->|Deploy| ServerNA
    LocalDB -->|Migrate| NeonDB

    CDNNA -->|Cached| EndUser["End Users"]
    CDNEU -->|Cached| EndUser
    CDNAP -->|Cached| EndUser

    ServerNA -->|API| EndUser
    ServerEU -->|API| EndUser
    ServerAP -->|API| EndUser

    ServerNA -->|Query| NeonDB
    ServerEU -->|Query| NeonDB
    ServerAP -->|Query| NeonDB

    NeonDB -->|Backup| NeonBackup
    ServerNA -->|Cache| RedisCloud

    ServerNA -->|OAuth| Clerk
    ServerNA -->|Webhooks| GitHub
    ServerNA -->|Events| Inngest
    Inngest -->|Send Email| Sendgrid

    style LocalClient fill:#e8f4f8,stroke:#333
    style LocalServer fill:#e8f4f8,stroke:#333
    style LocalDB fill:#e8f4f8,stroke:#333
    style CDNNA fill:#61dafb,color:#000,stroke:#333
    style ServerNA fill:#90c53f,color:#fff,stroke:#333
    style NeonDB fill:#0066ff,color:#fff,stroke:#333
    style Clerk fill:#1f3a93,color:#fff,stroke:#333
    style GitHub fill:#333,color:#fff,stroke:#333
```

---

## 4. SEQUENCE DIAGRAM - USER LOGIN

```mermaid
sequenceDiagram
    participant User as User<br/>Browser
    participant Frontend as React<br/>Frontend
    participant Clerk as Clerk<br/>Service
    participant API as Express<br/>API
    participant DB as PostgreSQL<br/>Database
    participant Redux as Redux<br/>Store

    User->>Frontend: Click "Sign In"
    Frontend->>Clerk: Initiate OAuth
    Clerk->>User: Show Auth Page
    User->>Clerk: Enter Credentials
    Clerk->>Clerk: Validate Credentials
    Clerk->>Frontend: Return JWT Token
    Frontend->>Frontend: Store Token (httpOnly)
    Frontend->>API: GET /api/workspaces (with token)
    API->>API: Verify Token with Clerk
    API->>DB: Query user workspaces
    DB-->>API: Return workspaces + projects
    API-->>Frontend: Return data (JSON)
    Frontend->>Redux: Dispatch setWorkspaces
    Redux-->>Frontend: Update store
    Frontend->>User: Render Dashboard
    User->>User: View Projects
```

---

## 5. SEQUENCE DIAGRAM - CREATE TASK

```mermaid
sequenceDiagram
    participant User as User
    participant Frontend as React<br/>Frontend
    participant Redux as Redux
    participant API as Express<br/>API
    participant DB as PostgreSQL
    participant WS as WebSocket<br/>Server
    participant Inngest as Inngest<br/>Queue
    participant Email as Email<br/>Service

    User->>Frontend: Fill Task Form
    User->>Frontend: Click "Create"
    Frontend->>Redux: Dispatch createTaskAsync
    Redux->>Frontend: Set loading = true
    Frontend->>API: POST /api/tasks
    API->>DB: INSERT task record
    DB-->>API: Task created
    API-->>Frontend: Return task data
    Frontend->>Redux: Dispatch taskCreated
    Redux->>Frontend: Add to store
    Frontend->>User: Show success toast
    API->>WS: Broadcast taskCreated
    WS->>Frontend: Send WebSocket event
    Frontend->>Redux: Update store (real-time)
    API->>Inngest: Emit taskAssigned event
    Inngest->>DB: Update task metadata
    Inngest->>Email: Send notification
    Email->>Email: Queue email
```

---

## 6. SEQUENCE DIAGRAM - GITHUB WEBHOOK

```mermaid
sequenceDiagram
    participant GH as GitHub<br/>Repository
    participant WebHook as Webhook<br/>Endpoint
    participant API as Express<br/>API
    participant Verify as Signature<br/>Verification
    participant DB as PostgreSQL
    participant WS as WebSocket
    participant Inngest as Inngest

    GH->>WebHook: POST issue.opened event
    WebHook->>Verify: Verify HMAC signature
    Verify->>Verify: Calculate HMAC-SHA256
    Verify-->>WebHook: Signature valid ✓
    WebHook->>API: Extract issue data
    API->>DB: Find linked task
    alt Task exists
        API->>DB: Update task status
    else Task not found
        API->>DB: Create new task from issue
    end
    DB-->>API: Task saved
    API->>WS: Broadcast update
    WS->>API: Emit to clients
    API->>Inngest: Emit taskUpdated event
    Inngest->>Inngest: Process background jobs
```

---

## 7. CLASS DIAGRAM

```mermaid
classDiagram
    class User {
        -id: String
        -name: String
        -email: String
        -image: String
        -designation: String
        -department: String
        +getWorkspaces()
        +getAssignedTasks()
        +updateProfile()
    }

    class Workspace {
        -id: String
        -name: String
        -slug: String
        -description: String
        -settings: JSON
        -ownerId: String
        +addMember()
        +removeMember()
        +getProjects()
        +getMembers()
    }

    class Project {
        -id: String
        -name: String
        -description: String
        -status: String
        -priority: String
        -startDate: DateTime
        -endDate: DateTime
        -progress: Integer
        +addTask()
        +removeTask()
        +getTasks()
        +getMembers()
        +updateProgress()
    }

    class Task {
        -id: String
        -title: String
        -description: String
        -status: String
        -priority: String
        -type: String
        -dueDate: DateTime
        -assigneeId: String
        -githubIssueNumber: Integer
        +updateStatus()
        +assignTo()
        +addComment()
        +linkGitHubIssue()
    }

    class Comment {
        -id: String
        -content: String
        -userId: String
        -taskId: String
        -createdAt: DateTime
        +updateContent()
        +delete()
    }

    class ProjectGitHubIntegration {
        -id: String
        -projectId: String
        -repository: String
        -accessToken: String
        -webhookSecret: String
        +generateWebhookSecret()
        +verifyWebhookSignature()
        +createWebhook()
        +disconnect()
    }

    class WorkspaceMember {
        -id: String
        -userId: String
        -workspaceId: String
        -role: String
        +updateRole()
        +remove()
    }

    class ProjectMember {
        -id: String
        -userId: String
        -projectId: String
        +remove()
    }

    User "1" -- "*" Workspace : owns
    User "1" -- "*" WorkspaceMember : joins
    Workspace "1" -- "*" WorkspaceMember : contains
    Workspace "1" -- "*" Project : contains
    User "1" -- "*" Project : leads
    User "1" -- "*" ProjectMember : joins
    Project "1" -- "*" ProjectMember : contains
    Project "1" -- "*" Task : contains
    User "1" -- "*" Task : assigns
    User "1" -- "*" Comment : writes
    Task "1" -- "*" Comment : has
    Project "1" -- "0..1" ProjectGitHubIntegration : has
```

---

## 8. STATE DIAGRAM - TASK LIFECYCLE

```mermaid
stateDiagram-v2
    [*] --> TODO
    
    TODO --> IN_PROGRESS: assign_task / notify_assignee
    TODO --> DONE: mark_complete / if urgent
    
    IN_PROGRESS --> TODO: revert / reassign
    IN_PROGRESS --> DONE: complete_task / update_github
    
    DONE --> IN_PROGRESS: reopen / needs_rework
    
    DONE --> [*]: archive

    note right of TODO
        Initial state
        No assignee
        Ready to start
    end note

    note right of IN_PROGRESS
        Active work
        Assigned to user
        Can have subtasks
    end note

    note right of DONE
        Completed work
        Can be archived
        Links to GitHub issue
    end note
```

---

## 9. STATE DIAGRAM - PROJECT LIFECYCLE

```mermaid
stateDiagram-v2
    [*] --> PLANNING
    
    PLANNING --> ACTIVE: start_project / notify_team
    PLANNING --> CANCELLED: cancel / archive_tasks
    
    ACTIVE --> ON_HOLD: pause_project / notify_stakeholders
    ACTIVE --> COMPLETED: finish_project / generate_report
    ACTIVE --> CANCELLED: cancel / refund_resources
    
    ON_HOLD --> ACTIVE: resume_project / restart_timeline
    ON_HOLD --> CANCELLED: cancel / cleanup
    
    COMPLETED --> [*]: archive
    CANCELLED --> [*]: archive

    note right of PLANNING
        Initial phase
        Setting up team
        Creating tasks
    end note

    note right of ACTIVE
        In progress
        Tasks being completed
        Team working
    end note

    note right of ON_HOLD
        Temporarily paused
        Can resume later
        Resources preserved
    end note
```

---

## 10. ACTIVITY DIAGRAM - CREATE PROJECT WORKFLOW

```mermaid
graph TD
    Start([User Initiates Project Creation]) --> Form["Display Project Form"]
    Form --> Input["User Enters:<br/>Name, Description,<br/>Dates, Priority"]
    Input --> Validate{Valid<br/>Input?}
    
    Validate -->|No| Error["Show Validation<br/>Errors"]
    Error --> Input
    
    Validate -->|Yes| CheckAuth{User<br/>Authorized?}
    CheckAuth -->|No| Denied["Show Permission<br/>Error"]
    Denied --> End1([End])
    
    CheckAuth -->|Yes| Create["Create Project<br/>in Database"]
    Create --> AddLead["Add Team Lead"]
    AddLead --> Notify["Send Notifications<br/>to Team"]
    Notify --> Webhook["Trigger Inngest<br/>Event"]
    Webhook --> Cache["Update Cache"]
    Cache --> Broadcast["Broadcast WebSocket<br/>Update"]
    Broadcast --> Success["Show Success<br/>Message"]
    Success --> Redirect["Redirect to<br/>Project Page"]
    Redirect --> End2([End])

    style Start fill:#22c55e,color:#fff
    style Validate fill:#ed8936,color:#fff
    style CheckAuth fill:#ed8936,color:#fff
    style Create fill:#0066ff,color:#fff
    style Broadcast fill:#764abc,color:#fff
    style Success fill:#22c55e,color:#fff
    style End1 fill:#dc2626,color:#fff
    style End2 fill:#22c55e,color:#fff
```

---

## 11. ACTIVITY DIAGRAM - GITHUB SYNC WORKFLOW

```mermaid
graph TD
    Start([GitHub Webhook Received]) --> Extract["Extract Event Data<br/>Issue/PR Info"]
    Extract --> CheckSig["Verify HMAC<br/>Signature"]
    
    CheckSig -->|Invalid| Reject["Reject Request<br/>Log Error"]
    Reject --> End1([End])
    
    CheckSig -->|Valid| Parse["Parse Issue Data<br/>Number, State, Title"]
    Parse --> FindTask["Query: Find<br/>Linked Task"]
    
    FindTask --> TaskExists{Task<br/>Exists?}
    
    TaskExists -->|Yes| MapState["Map Issue State<br/>to Task Status"]
    MapState --> Update["Update Task<br/>in Database"]
    Update --> SaveLink["Save GitHub<br/>Issue Link"]
    
    TaskExists -->|No| CreateTask["Create New Task<br/>from Issue"]
    CreateTask --> SaveLink
    
    SaveLink --> UpdateGH["Call GitHub API<br/>Update Issue"]
    UpdateGH --> Emit["Emit Inngest<br/>Event"]
    Emit --> Notify["Send Notifications"]
    Notify --> Broadcast["Broadcast WebSocket<br/>Update to Clients"]
    Broadcast --> Success["Log Success"]
    Success --> End2([End])

    style Start fill:#333,color:#fff
    style CheckSig fill:#ed8936,color:#fff
    style FindTask fill:#ed8936,color:#fff
    style TaskExists fill:#ed8936,color:#fff
    style Update fill:#0066ff,color:#fff
    style CreateTask fill:#0066ff,color:#fff
    style Broadcast fill:#764abc,color:#fff
    style Success fill:#22c55e,color:#fff
    style End1 fill:#dc2626,color:#fff
    style End2 fill:#22c55e,color:#fff
```

---

## 12. FLOWCHART - REQUEST HANDLING PIPELINE

```mermaid
graph TD
    A["HTTP Request Received"] --> B["Parse URL & Method"]
    B --> C["Extract Headers"]
    C --> D["CORS Check"]
    
    D -->|Failed| D1["Return 403<br/>CORS Error"]
    D1 --> End1([End])
    
    D -->|Passed| E["Parse Body"]
    E --> F["Validate JSON"]
    
    F -->|Invalid| F1["Return 400<br/>Bad Request"]
    F1 --> End1
    
    F -->|Valid| G["Extract Token<br/>from Header"]
    G --> H["Verify Token<br/>with Clerk"]
    
    H -->|Invalid| H1["Return 401<br/>Unauthorized"]
    H1 --> End1
    
    H -->|Valid| I["Extract User ID<br/>from Token"]
    I --> J["Check Route<br/>Authorization"]
    
    J -->|Denied| J1["Return 403<br/>Forbidden"]
    J1 --> End1
    
    J -->|Allowed| K["Route to<br/>Controller"]
    K --> L["Execute Business<br/>Logic"]
    L --> M["Query Database"]
    
    M -->|Error| M1["Rollback<br/>Transaction"]
    M1 --> M2["Log Error"]
    M2 --> M3["Return 500<br/>Server Error"]
    M3 --> End1
    
    M -->|Success| N["Transform<br/>Response"]
    N --> O["Add Headers"]
    O --> P["Send JSON<br/>Response 200"]
    P --> Q["Emit WebSocket<br/>Update"]
    Q --> R["Log Request"]
    R --> End2([End])

    style A fill:#e8f4f8,stroke:#333,stroke-width:2px
    style D fill:#ed8936,color:#fff
    style F fill:#ed8936,color:#fff
    style H fill:#ed8936,color:#fff
    style J fill:#ed8936,color:#fff
    style M fill:#ed8936,color:#fff
    style P fill:#22c55e,color:#fff
    style End1 fill:#dc2626,color:#fff
    style End2 fill:#22c55e,color:#fff
```

---

## 13. GANTT CHART - PROJECT TIMELINE

```mermaid
gantt
    title TaskBit Development Timeline
    dateFormat YYYY-MM-DD
    
    section Frontend
    Setup & Auth         :done, fe1, 2026-01-01, 7d
    Dashboard            :active, fe2, 2026-01-08, 14d
    Project Views        :fe3, 2026-01-22, 21d
    Task Management UI   :fe4, 2026-02-12, 21d
    Analytics & Reports  :fe5, 2026-03-05, 14d
    
    section Backend
    API Setup            :done, be1, 2026-01-01, 7d
    Workspace Module     :active, be2, 2026-01-08, 14d
    Project Module       :be3, 2026-01-22, 21d
    Task Module          :be4, 2026-02-12, 21d
    GitHub Integration   :be5, 2026-03-05, 28d
    
    section Database
    Schema Design        :done, db1, 2026-01-01, 7d
    Migrations           :active, db2, 2026-01-08, 14d
    Indexing             :db3, 2026-01-22, 7d
    Performance Tuning   :db4, 2026-02-01, 21d
    
    section Testing
    Unit Tests           :test1, 2026-02-01, 28d
    Integration Tests    :test2, 2026-02-15, 28d
    E2E Tests            :test3, 2026-03-01, 21d
    Performance Tests    :test4, 2026-03-15, 14d
    
    section Deployment
    Staging Deploy       :deploy1, 2026-03-22, 7d
    Load Testing         :deploy2, 2026-03-29, 7d
    Production Deploy    :crit, deploy3, 2026-04-05, 7d
    Launch               :crit, deploy4, 2026-04-12, 1d
```

---

## 14. GIT BRANCHING STRATEGY - GIT FLOW

```mermaid
graph LR
    subgraph "Main Branches"
        Main["main<br/>(Production)"]
        Develop["develop<br/>(Development)"]
    end

    subgraph "Feature Branches"
        F1["feature/user-auth"]
        F2["feature/github-integration"]
        F3["feature/real-time-updates"]
        F4["feature/analytics"]
    end

    subgraph "Release Branches"
        Release["release/v1.0.0"]
    end

    subgraph "Hotfix Branches"
        Hotfix["hotfix/security-patch"]
    end

    Develop -->|Create| F1
    Develop -->|Create| F2
    Develop -->|Create| F3
    Develop -->|Create| F4
    
    F1 -->|PR + Review| Develop
    F2 -->|PR + Review| Develop
    F3 -->|PR + Review| Develop
    F4 -->|PR + Review| Develop
    
    Develop -->|Release Branch| Release
    Release -->|Merge| Main
    Main -->|Tag v1.0.0| Main
    
    Main -->|Hotfix| Hotfix
    Hotfix -->|Merge| Main
    Hotfix -->|Merge| Develop

    style Main fill:#22c55e,color:#fff,stroke:#333,stroke-width:2px
    style Develop fill:#0066ff,color:#fff,stroke:#333,stroke-width:2px
    style F1 fill:#61dafb,color:#000,stroke:#333
    style F2 fill:#61dafb,color:#000,stroke:#333
    style F3 fill:#61dafb,color:#000,stroke:#333
    style F4 fill:#61dafb,color:#000,stroke:#333
    style Release fill:#ed8936,color:#fff,stroke:#333,stroke-width:2px
    style Hotfix fill:#dc2626,color:#fff,stroke:#333,stroke-width:2px
```

---

## 15. DATABASE ENTITY RELATIONSHIP (ERD) - DETAILED

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
        timestamp createdAt
        timestamp updatedAt
    }

    WORKSPACE {
        string id PK
        string name
        string slug UK
        string description
        json settings
        string ownerId FK
        string image_url
        timestamp createdAt
        timestamp updatedAt
    }

    WORKSPACEMEMBER {
        string id PK
        string userId FK
        string workspaceId FK
        string message
        enum role
        timestamp createdAt
    }

    PROJECT {
        string id PK
        string name
        string description
        enum priority
        enum status
        timestamp start_date
        timestamp end_date
        string team_lead FK
        string workspaceId FK
        int progress
        timestamp createdAt
        timestamp updatedAt
    }

    PROJECTMEMBER {
        string id PK
        string userId FK
        string projectId FK
        timestamp createdAt
    }

    TASK {
        string id PK
        string projectId FK
        string title
        string description
        enum status
        enum type
        enum priority
        string assigneeId FK
        timestamp due_date
        int githubIssueNumber
        string githubIssueUrl
        string githubRepository
        timestamp createdAt
        timestamp updatedAt
    }

    COMMENT {
        string id PK
        string content
        string userId FK
        string taskId FK
        timestamp createdAt
    }

    PROJECTGITHUBINTEGRATION {
        string id PK
        string projectId FK UK
        string githubAccountLogin
        string githubUserId
        string repository
        string accessToken
        string webhookSecret
        timestamp webhookSecretUpdatedAt
        timestamp createdAt
        timestamp updatedAt
    }

    GITHUBOAUTHSTATE {
        string id PK
        string state UK
        string userId
        string projectId
        string integrationId FK
        timestamp expiresAt
        timestamp createdAt
    }
```

---

## 16. C4 MODEL - CONTEXT LEVEL

```mermaid
graph TB
    User["👤 User<br/>(Team Member)"]
    Admin["👨‍💼 Admin<br/>(Workspace Owner)"]
    GitHub["🐙 GitHub<br/>(External System)"]
    
    TaskBit["TaskBit<br/>(Project Management<br/>& Task Tracking System)"]
    
    User -->|Uses| TaskBit
    Admin -->|Manages| TaskBit
    GitHub -->|Webhooks| TaskBit
    TaskBit -->|API Calls| GitHub
    TaskBit -->|Sends| Email["📧 Email<br/>(Notifications)"]

    style TaskBit fill:#4a5568,color:#fff,stroke:#2d3748,stroke-width:3px
    style User fill:#48bb78,color:#fff,stroke:#333
    style Admin fill:#ed8936,color:#fff,stroke:#333
    style GitHub fill:#2d3748,color:#fff,stroke:#333
    style Email fill:#9f7aea,color:#fff,stroke:#333
```

---

## 17. C4 MODEL - CONTAINER LEVEL

```mermaid
graph TB
    User["👤 User"]
    
    subgraph "TaskBit System"
        WebApp["Web Application<br/>(React 19 + Redux)"]
        API["API Server<br/>(Express.js)"]
        Database["Database<br/>(PostgreSQL)"]
        Cache["Cache<br/>(Redis)"]
        Queue["Event Queue<br/>(Inngest)"]
    end

    GitHub["GitHub API"]
    Clerk["Clerk Auth"]
    Email["Email Service<br/>(Nodemailer)"]

    User -->|Uses| WebApp
    WebApp -->|API Calls| API
    WebApp -->|Auth| Clerk
    API -->|Query| Database
    API -->|Cache| Cache
    API -->|Events| Queue
    API -->|OAuth| Clerk
    API -->|Webhooks| GitHub
    Queue -->|Send| Email

    style WebApp fill:#61dafb,color:#000,stroke:#333
    style API fill:#90c53f,color:#fff,stroke:#333
    style Database fill:#0066ff,color:#fff,stroke:#333
    style Cache fill:#764abc,color:#fff,stroke:#333
    style Queue fill:#ed8936,color:#fff,stroke:#333
```

---

## 18. C4 MODEL - COMPONENT LEVEL

```mermaid
graph TB
    WebApp["Web Application<br/>(React)"]
    
    subgraph "API Server (Express)"
        AuthC["Authentication<br/>Component"]
        WorkspaceC["Workspace<br/>Component"]
        ProjectC["Project<br/>Component"]
        TaskC["Task<br/>Component"]
        CommentC["Comment<br/>Component"]
        GitHubC["GitHub Integration<br/>Component"]
    end
    
    Database["PostgreSQL"]
    Inngest["Inngest Events"]

    WebApp -->|Login| AuthC
    WebApp -->|CRUD| WorkspaceC
    WebApp -->|CRUD| ProjectC
    WebApp -->|CRUD| TaskC
    WebApp -->|CRUD| CommentC
    WebApp -->|Setup| GitHubC
    
    AuthC -->|Query| Database
    WorkspaceC -->|Query| Database
    ProjectC -->|Query| Database
    TaskC -->|Query| Database
    CommentC -->|Query| Database
    GitHubC -->|Query| Database
    
    TaskC -->|Events| Inngest
    WorkspaceC -->|Events| Inngest

    style AuthC fill:#1f3a93,color:#fff,stroke:#333
    style WorkspaceC fill:#764abc,color:#fff,stroke:#333
    style ProjectC fill:#0066ff,color:#fff,stroke:#333
    style TaskC fill:#22c55e,color:#fff,stroke:#333
    style CommentC fill:#ed8936,color:#fff,stroke:#333
    style GitHubC fill:#333,color:#fff,stroke:#333
```

---

## 19. C4 MODEL - CODE LEVEL (Example: Task Module)

```mermaid
graph TD
    TaskRouter["Task Router<br/>(Express Routes)"]
    TaskController["Task Controller<br/>(Business Logic)"]
    TaskService["Task Service<br/>(Domain Logic)"]
    TaskRepository["Task Repository<br/>(Data Access)"]
    TaskModel["Task Model<br/>(Prisma)"]
    
    TaskRouter -->|Handle Requests| TaskController
    TaskController -->|Validate & Process| TaskService
    TaskService -->|CRUD Operations| TaskRepository
    TaskRepository -->|Query| TaskModel
    TaskModel -->|ORM| Database["PostgreSQL"]
    
    TaskService -->|Emit Events| EventService["Event Service<br/>(Inngest)"]
    TaskController -->|Responses| Frontend["Frontend"]

    style TaskRouter fill:#90c53f,color:#fff,stroke:#333
    style TaskController fill:#0066ff,color:#fff,stroke:#333
    style TaskService fill:#764abc,color:#fff,stroke:#333
    style TaskRepository fill:#333,color:#fff,stroke:#333
    style TaskModel fill:#5b4b8a,color:#fff,stroke:#333
    style EventService fill:#ed8936,color:#fff,stroke:#333
```

---

## 20. SYSTEM INTERACTION DIAGRAM

```mermaid
graph LR
    subgraph "User Actions"
        UA["User<br/>Actions"]
    end

    subgraph "Frontend"
        React["React UI"]
        Redux["Redux Store"]
        Network["HTTP/WS"]
    end

    subgraph "Backend"
        Express["Express API"]
        Middleware["Auth Middleware"]
        Controllers["Controllers"]
        Services["Business Services"]
    end

    subgraph "Data Layer"
        ORM["Prisma ORM"]
        DB["PostgreSQL"]
        Cache["Redis"]
    end

    subgraph "External"
        GitHub["GitHub API"]
        Clerk["Clerk OAuth"]
        Inngest["Inngest Events"]
    end

    UA -->|Interact| React
    React -->|Dispatch| Redux
    Redux -->|HTTP/WS| Network
    Network -->|Request| Express
    Express -->|Verify| Middleware
    Middleware -->|Route| Controllers
    Controllers -->|Call| Services
    Services -->|Query| ORM
    ORM -->|Access| DB
    ORM -->|Cache| Cache
    Services -->|Call| GitHub
    Services -->|Verify| Clerk
    Services -->|Event| Inngest
    Inngest -->|Update| DB
    Express -->|Response| Network
    Network -->|Update| Redux
    Redux -->|Render| React
    React -->|Display| UA

    style React fill:#61dafb,color:#000,stroke:#333
    style Redux fill:#764abc,color:#fff,stroke:#333
    style Express fill:#90c53f,color:#fff,stroke:#333
    style DB fill:#0066ff,color:#fff,stroke:#333
    style GitHub fill:#333,color:#fff,stroke:#333
```

---

## 21. ROADMAP - FEATURE DEVELOPMENT PHASES

```mermaid
graph LR
    subgraph "Phase 1: MVP (Current)"
        P1["✓ Core Features<br/>- Workspaces<br/>- Projects<br/>- Tasks<br/>- Comments<br/>- GitHub Integration"]
    end

    subgraph "Phase 2: Q2 2026"
        P2["🚀 Enhanced Collaboration<br/>- Real-time Editing<br/>- @Mentions<br/>- Activity Feeds<br/>- Advanced Permissions"]
    end

    subgraph "Phase 3: Q3 2026"
        P3["📊 Advanced Analytics<br/>- Reports<br/>- Burndown Charts<br/>- Team Metrics<br/>- Forecasting"]
    end

    subgraph "Phase 4: Q4 2026"
        P4["🔌 Integrations<br/>- Slack<br/>- Jira<br/>- Azure DevOps<br/>- Zapier"]
    end

    subgraph "Phase 5: 2027"
        P5["📱 Mobile<br/>- iOS App<br/>- Android App<br/>- Offline Support"]
    end

    P1 -->|Development| P2
    P2 -->|Development| P3
    P3 -->|Development| P4
    P4 -->|Development| P5

    style P1 fill:#22c55e,color:#fff,stroke:#333,stroke-width:2px
    style P2 fill:#0066ff,color:#fff,stroke:#333
    style P3 fill:#764abc,color:#fff,stroke:#333
    style P4 fill:#ed8936,color:#fff,stroke:#333
    style P5 fill:#1f3a93,color:#fff,stroke:#333
```

---

## 22. TESTING PYRAMID

```mermaid
graph TD
    E2E["E2E Tests<br/>(10%)<br/>Selenium/Cypress<br/>Full user workflows"]
    E2E_Label["⬆️ Slowest & Expensive"]
    
    Integration["Integration Tests<br/>(30%)<br/>Jest/Mocha<br/>API + Database"]
    
    Unit["Unit Tests<br/>(60%)<br/>Jest<br/>Individual functions<br/>⬇️ Fastest & Cheap"]

    style Unit fill:#22c55e,color:#fff,stroke:#333
    style Integration fill:#ed8936,color:#fff,stroke:#333
    style E2E fill:#dc2626,color:#fff,stroke:#333
    style E2E_Label fill:#333,color:#fff,stroke:#333,stroke-width:2px
```

---

## 23. MONITORING & LOGGING ARCHITECTURE

```mermaid
graph TB
    subgraph "Application"
        Frontend["Frontend<br/>(React)"]
        API["API Server<br/>(Express)"]
        DB["Database<br/>(PostgreSQL)"]
    end

    subgraph "Monitoring"
        APM["APM<br/>(Application Performance<br/>Monitoring)"]
        ErrorTracking["Error Tracking<br/>(Sentry)"]
        Logging["Log Aggregation<br/>(ELK/DataDog)"]
    end

    subgraph "Alerts"
        Email_Alert["Email Alerts"]
        Slack["Slack Notifications"]
        Dashboard["Monitoring Dashboard"]
    end

    Frontend -->|Metrics| APM
    Frontend -->|Errors| ErrorTracking
    Frontend -->|Logs| Logging
    
    API -->|Metrics| APM
    API -->|Errors| ErrorTracking
    API -->|Logs| Logging
    
    DB -->|Query Performance| APM
    DB -->|Connection Errors| ErrorTracking
    DB -->|Logs| Logging
    
    APM -->|Threshold Exceeded| Email_Alert
    ErrorTracking -->|Critical Error| Slack
    Logging -->|Anomaly Detected| Dashboard

    style APM fill:#0066ff,color:#fff,stroke:#333
    style ErrorTracking fill:#dc2626,color:#fff,stroke:#333
    style Logging fill:#764abc,color:#fff,stroke:#333
    style Slack fill:#333,color:#fff,stroke:#333
```

---

## 24. SECURITY ARCHITECTURE

```mermaid
graph TB
    subgraph "Client Security"
        HTTPS["HTTPS/TLS<br/>Encrypted Transport"]
        CORS["CORS Policy<br/>Origin Validation"]
        CSP["Content Security<br/>Policy"]
        HttpOnly["HttpOnly Cookies<br/>Token Storage"]
    end

    subgraph "API Security"
        AuthN["Authentication<br/>(OAuth via Clerk)"]
        AuthZ["Authorization<br/>(Role-Based Access)"]
        Validate["Input Validation<br/>(Sanitization)"]
        RateLimit["Rate Limiting<br/>(DDoS Protection)"]
    end

    subgraph "Data Security"
        Encryption["Encryption at Rest<br/>(AES-256)"]
        DBAuth["Database Auth<br/>(IAM Roles)"]
        Audit["Audit Logging<br/>(All Changes)"]
    end

    subgraph "Infrastructure Security"
        Firewall["Firewall Rules<br/>(VPC/Security Groups)"]
        SSL["SSL Certificates<br/>(Let's Encrypt)"]
        Backup["Encrypted Backups<br/>(Daily)"]
    end

    HTTPS -->|Protects| Client["User Data"]
    CORS -->|Prevents| CSRF["CSRF Attacks"]
    HttpOnly -->|Prevents| XSS["XSS Attacks"]
    
    AuthN -->|Validates| Request["API Request"]
    AuthZ -->|Checks| Permission["User Permission"]
    Validate -->|Prevents| SQLInj["SQL Injection"]
    RateLimit -->|Prevents| DDoS["DDoS"]
    
    Encryption -->|Protects| Sensitive["Sensitive Data"]
    Audit -->|Tracks| Activity["User Activity"]
    
    Firewall -->|Restricts| Access["Network Access"]
    SSL -->|Encrypts| Traffic["Data in Transit"]

    style AuthN fill:#1f3a93,color:#fff,stroke:#333,stroke-width:2px
    style AuthZ fill:#1f3a93,color:#fff,stroke:#333,stroke-width:2px
    style Encryption fill:#0066ff,color:#fff,stroke:#333,stroke-width:2px
    style Audit fill:#764abc,color:#fff,stroke:#333,stroke-width:2px
```

---

## 25. PERFORMANCE OPTIMIZATION STRATEGY

```mermaid
graph TB
    subgraph "Frontend Optimization"
        CodeSplit["Code Splitting<br/>(Reduce Bundle)"]
        LazyLoad["Lazy Loading<br/>(Load on Demand)"]
        Cache["Caching<br/>(Browser Cache)"]
        CDN["CDN<br/>(Edge Distribution)"]
    end

    subgraph "Backend Optimization"
        DBIndex["Database Indexing<br/>(Faster Queries)"]
        Query["Query Optimization<br/>(Avoid N+1)"]
        Cache_Server["Server Cache<br/>(Redis)"]
        Compression["Compression<br/>(GZIP)"]
    end

    subgraph "Infrastructure Optimization"
        AutoScale["Auto-Scaling<br/>(Handle Load)"]
        LoadBalance["Load Balancing<br/>(Distribute Traffic)"]
        CDN_Static["Static File CDN<br/>(Images, CSS)"]
        DBReplica["Database Replicas<br/>(Read Scaling)"]
    end

    subgraph "Monitoring"
        Metrics["Performance Metrics<br/>(Response Time)"]
        Alerts["Performance Alerts<br/>Threshold Exceeded"]
        Reports["Performance Reports<br/>Weekly/Monthly"]
    end

    CodeSplit -->|Improves| LoadTime["Page Load Time"]
    LazyLoad -->|Improves| LoadTime
    Cache -->|Improves| LoadTime
    CDN -->|Improves| LoadTime
    
    DBIndex -->|Improves| QueryTime["Query Response"]
    Query -->|Improves| QueryTime
    Cache_Server -->|Improves| QueryTime
    Compression -->|Improves| Bandwidth["Bandwidth Usage"]
    
    AutoScale -->|Handles| Spike["Traffic Spikes"]
    LoadBalance -->|Distributes| Traffic["User Traffic"]
    CDN_Static -->|Reduces| ServerLoad["Server Load"]
    DBReplica -->|Scales| Reads["Read Operations"]
    
    LoadTime -->|Monitored by| Metrics
    QueryTime -->|Monitored by| Metrics
    Spike -->|Monitored by| Metrics
    Metrics -->|Triggers| Alerts

    style CodeSplit fill:#61dafb,color:#000,stroke:#333
    style Cache_Server fill:#764abc,color:#fff,stroke:#333
    style DBIndex fill:#0066ff,color:#fff,stroke:#333
    style AutoScale fill:#22c55e,color:#fff,stroke:#333
    style Metrics fill:#ed8936,color:#fff,stroke:#333
```

---

## 26. MIGRATION & DATA FLOW STRATEGY

```mermaid
graph LR
    subgraph "Old System"
        OldDB["Legacy Database"]
        OldApp["Old Application"]
    end

    subgraph "Migration Phase"
        ETL["ETL Process<br/>(Extract, Transform,<br/>Load)"]
        Validate["Data Validation<br/>(Verify Integrity)"]
        BackupOld["Backup Old Data<br/>(Safety)"]
    end

    subgraph "New System"
        NewDB["PostgreSQL Neon"]
        NewApp["TaskBit"]
    end

    subgraph "Post-Migration"
        Monitor["Monitor Both<br/>Systems"]
        Fallback["Fallback Plan<br/>(If Issues)"]
        Cutover["Final Cutover<br/>(Switch Traffic)"]
    end

    OldDB -->|Extract| ETL
    OldApp -->|Extract| ETL
    ETL -->|Transform| Validate
    Validate -->|Load| NewDB
    OldDB -->|Backup| BackupOld
    NewDB -->|Data| NewApp
    NewApp -->|Monitor| Monitor
    Monitor -->|Issue Found| Fallback
    Fallback -->|Rollback| OldSystem["Revert to<br/>Old System"]
    Monitor -->|All Clear| Cutover
    Cutover -->|Success| Complete["Migration<br/>Complete"]

    style ETL fill:#ed8936,color:#fff,stroke:#333,stroke-width:2px
    style Validate fill:#ed8936,color:#fff,stroke:#333,stroke-width:2px
    style NewDB fill:#0066ff,color:#fff,stroke:#333,stroke-width:2px
    style Complete fill:#22c55e,color:#fff,stroke:#333,stroke-width:2px
```

---

## 27. DISASTER RECOVERY PLAN

```mermaid
graph TD
    Disaster["🚨 Disaster Occurs<br/>(Data Loss/Outage)"]
    
    Disaster --> Assess["1. Assess Situation<br/>- Check backup status<br/>- Determine scope"]
    Assess --> Decision{Recovery<br/>Possible?}
    
    Decision -->|No| Legal["Contact Legal<br/>Customer Notification<br/>Incident Report"]
    Decision -->|Yes| GetBackup["2. Get Latest Backup<br/>- Verify integrity<br/>- Test restore"]
    
    GetBackup --> Restore["3. Restore System<br/>- Restore database<br/>- Restore application"]
    Restore --> Verify["4. Verify Data<br/>- Check data integrity<br/>- Validate transactions"]
    Verify --> Notify["5. Notify Users<br/>- Send notifications<br/>- Provide ETA"]
    Notify --> Resume["6. Resume Operations<br/>- Switch traffic back<br/>- Monitor closely"]
    Resume --> Postmortem["7. Post-Mortem<br/>- Root cause analysis<br/>- Prevention measures"]
    Postmortem --> Complete["✓ Recovery<br/>Complete"]

    style Disaster fill:#dc2626,color:#fff,stroke:#333,stroke-width:3px
    style Decision fill:#ed8936,color:#fff,stroke:#333,stroke-width:2px
    style GetBackup fill:#0066ff,color:#fff,stroke:#333
    style Restore fill:#0066ff,color:#fff,stroke:#333
    style Complete fill:#22c55e,color:#fff,stroke:#333,stroke-width:2px
```

---

