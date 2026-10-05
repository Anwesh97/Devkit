# Devkit

Devkit is a web-based Integrated Development Environment (IDE) built around **Code Server**, providing isolated, on-demand development workspaces for users.

The platform supports a hierarchical project/workspace structure with administrators at different levels, workspace-level users, and dynamically launched Code Server pods.

---

## 1. Overview

Devkit consists of:

- **Angular** — Frontend web application
- **Spring Boot** — Backend REST API and business layer
- **Code Server** — Browser-based VS Code development environment
- **Kubernetes** — Runs isolated Code Server pods
- **Database** — Stores users, projects, workspaces, roles, permissions, and configuration

### High-Level Flow

```text
                         ┌─────────────────────┐
                         │       User          │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │   Angular Frontend  │
                         │       (Web UI)      │
                         └──────────┬──────────┘
                                    │ REST API
                                    ▼
                         ┌─────────────────────┐
                         │   Spring Boot API   │
                         │                     │
                         │ Auth / RBAC         │
                         │ Projects            │
                         │ Workspaces          │
                         │ Users               │
                         │ Pod Lifecycle       │
                         └──────────┬──────────┘
                                    │
                          Kubernetes API
                                    │
                                    ▼
                    ┌───────────────────────────┐
                    │       Kubernetes          │
                    │                           │
                    │  ┌─────────────────────┐  │
                    │  │ Code Server Pod     │  │
                    │  │ Workspace A         │  │
                    │  └─────────────────────┘  │
                    │                           │
                    │  ┌─────────────────────┐  │
                    │  │ Code Server Pod     │  │
                    │  │ Workspace B         │  │
                    │  └─────────────────────┘  │
                    └───────────────────────────┘
```

---

# 2. Core Concepts

## 2.1 Project

A **Project** is the highest-level organizational unit.

Example:

```text
Organization
└── Project A
```

A project can contain multiple workspaces.

Project-level administrators can manage the resources underneath their project depending on their assigned permissions.

---

## 2.2 Workspace

A **Workspace** represents an isolated development environment.

Example:

```text
Project A
├── Workspace 1
├── Workspace 2
└── Workspace 3
```

Each workspace can have:

- Assigned users
- Workspace administrators
- Development configuration
- Resource limits
- Code Server configuration
- Persistent storage
- Runtime configuration

A user can launch a workspace if they have permission to access it.

---

# 3. Administrative Hierarchy

Devkit uses hierarchical administration.

```text
                    Platform
                       │
                       ▼
                    Project
                       │
          ┌────────────┴────────────┐
          │                         │
      Finance Admin            Operations Admin
                                    │
                                    ▼
                               Workspaces
                                    │
                                    ▼
                                  Users
```

The exact hierarchy and permissions should be enforced by the backend.

---

# 4. Roles

## 4.1 Platform Admin

Platform administrators have global administrative privileges.

Possible responsibilities:

- Manage projects
- Manage organizations
- Manage global configuration
- Manage administrators
- View platform-wide resources
- Manage Kubernetes/runtime configuration

---

## 4.2 Finance Admin

Finance administrators are responsible for financial and usage-related operations.

Possible responsibilities:

- View workspace usage
- View resource consumption
- Manage billing information
- View cost reports
- Manage quotas
- Generate usage reports

Finance admins should not automatically receive development or operational privileges.

---

## 4.3 Operations Admin

Operations administrators manage infrastructure and workspace operations.

Possible responsibilities:

- Manage workspaces
- Start/stop workspace environments
- View pod status
- Restart failed environments
- Manage workspace resources
- Monitor Code Server instances
- Manage runtime configuration

---

## 4.4 Workspace User

Workspace users are developers/end users.

They can:

- View assigned workspaces
- Launch assigned workspaces
- Access Code Server
- Work with files and repositories
- Stop their development environment if permitted

Example:

```text
User
 │
 ├── Workspace A
 │      └── Launch
 │
 └── Workspace C
        └── Launch
```

---

# 5. Workspace Launch Flow

The primary user workflow is:

```text
User
 │
 │ Login
 ▼
Angular Application
 │
 │ Get assigned workspaces
 ▼
Workspace List
 │
 │ Click "Launch"
 ▼
Spring Boot API
 │
 │ Validate user permission
 ▼
Workspace Service
 │
 │ Check existing environment
 ▼
Kubernetes
 │
 │ Create / start Code Server Pod
 ▼
Code Server
 │
 │ Generate access URL
 ▼
Spring Boot
 │
 │ Return launch information
 ▼
Angular
 │
 │ Open Code Server
 ▼
Browser-based IDE
```

---

# 6. Code Server Lifecycle

A workspace should not necessarily require a permanently running Code Server pod.

The backend can manage the lifecycle.

```text
             ┌─────────────┐
             │   STOPPED   │
             └──────┬──────┘
                    │
                 Launch
                    │
                    ▼
             ┌─────────────┐
             │  STARTING   │
             └──────┬──────┘
                    │
                    ▼
             ┌─────────────┐
             │   RUNNING   │
             └──────┬──────┘
                    │
          ┌─────────┴─────────┐
          │                   │
       Stop                 Failure
          │                   │
          ▼                   ▼
     ┌─────────┐        ┌────────────┐
     │ STOPPED │        │   ERROR    │
     └─────────┘        └─────┬──────┘
                              │
                            Restart
                              │
                              ▼
                          STARTING
```

The backend should maintain the desired state and actual Kubernetes state separately.

---

# 7. Backend Architecture

The backend will be implemented using **Spring Boot**.

Suggested structure:

```text
backend/
└── src/
    └── main/
        └── java/
            └── com/
                └── devkit/
                    ├── DevkitApplication.java
                    │
                    ├── auth/
                    │   ├── controller/
                    │   ├── service/
                    │   ├── repository/
                    │   └── model/
                    │
                    ├── user/
                    │   ├── controller/
                    │   ├── service/
                    │   ├── repository/
                    │   └── model/
                    │
                    ├── project/
                    │   ├── controller/
                    │   ├── service/
                    │   ├── repository/
                    │   └── model/
                    │
                    ├── workspace/
                    │   ├── controller/
                    │   ├── service/
                    │   ├── repository/
                    │   └── model/
                    │
                    ├── kubernetes/
                    │   ├── KubernetesClient.java
                    │   ├── PodService.java
                    │   └── DeploymentService.java
                    │
                    ├── billing/
                    │   ├── controller/
                    │   ├── service/
                    │   └── model/
                    │
                    └── common/
                        ├── exception/
                        ├── security/
                        └── config/
```

---

# 8. Frontend Architecture

The frontend will be implemented using **Angular**.

Suggested structure:

```text
frontend/
└── src/
    └── app/
        ├── core/
        │   ├── auth/
        │   ├── guards/
        │   ├── interceptors/
        │   └── services/
        │
        ├── shared/
        │   ├── components/
        │   ├── directives/
        │   └── pipes/
        │
        ├── features/
        │   ├── dashboard/
        │   ├── projects/
        │   ├── workspaces/
        │   ├── users/
        │   ├── administration/
        │   ├── finance/
        │   └── operations/
        │
        ├── layout/
        │   ├── header/
        │   ├── sidebar/
        │   └── navigation/
        │
        └── app.routes.ts
```

---

# 9. Repository Structure

The recommended repository structure is:

```text
devkit/
│
├── frontend/
│   └── Angular application
│
├── backend/
│   └── Spring Boot application
│
├── infrastructure/
│   ├── kubernetes/
│   │   ├── namespace/
│   │   ├── deployments/
│   │   ├── services/
│   │   ├── ingress/
│   │   └── rbac/
│   │
│   └── helm/
│       └── devkit/
│
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── security/
│   └── deployment/
│
├── scripts/
│
├── .gitignore
├── docker-compose.yml
└── README.md
```

---

# 10. Suggested Domain Model

Initial domain model:

```text
Organization
    │
    └── Project
          │
          └── Workspace
                │
                ├── WorkspaceUser
                │      │
                │      └── User
                │
                └── WorkspaceEnvironment
                       │
                       └── CodeServerPod
```

Additional administrative relationships:

```text
Project
 │
 ├── Project Admin
 ├── Finance Admin
 └── Operations Admin
```

---

# 11. Initial Database Model

A possible initial schema:

```text
users
-----
id
username
email
password_hash
status
created_at
updated_at


roles
-----
id
name


user_roles
----------
user_id
role_id


projects
--------
id
name
description
status
created_at
updated_at


workspaces
----------
id
project_id
name
description
status
resource_config
created_at
updated_at


workspace_users
---------------
workspace_id
user_id
role
created_at


workspace_environments
----------------------
id
workspace_id
status
pod_name
namespace
access_url
created_at
updated_at
```

This schema is intentionally minimal and can evolve as the domain becomes clearer.

---

# 12. REST API

Initial API structure:

```text
/api/v1
```

### Authentication

```http
POST /api/v1/auth/login
POST /api/v1/auth/logout
POST /api/v1/auth/refresh
GET  /api/v1/auth/me
```

### Projects

```http
GET    /api/v1/projects
POST   /api/v1/projects
GET    /api/v1/projects/{projectId}
PUT    /api/v1/projects/{projectId}
DELETE /api/v1/projects/{projectId}
```

### Workspaces

```http
GET    /api/v1/workspaces
POST   /api/v1/workspaces
GET    /api/v1/workspaces/{workspaceId}
PUT    /api/v1/workspaces/{workspaceId}
DELETE /api/v1/workspaces/{workspaceId}
```

### Workspace Users

```http
GET    /api/v1/workspaces/{workspaceId}/users
POST   /api/v1/workspaces/{workspaceId}/users
DELETE /api/v1/workspaces/{workspaceId}/users/{userId}
```

### Workspace Runtime

```http
POST /api/v1/workspaces/{workspaceId}/launch
POST /api/v1/workspaces/{workspaceId}/stop
POST /api/v1/workspaces/{workspaceId}/restart

GET /api/v1/workspaces/{workspaceId}/status
```

---

# 13. Launch API

Example:

```http
POST /api/v1/workspaces/123/launch
Authorization: Bearer <token>
```

Backend responsibilities:

1. Authenticate the user.
2. Verify workspace access.
3. Verify workspace is active.
4. Check whether an environment already exists.
5. Create or start the Kubernetes workload.
6. Wait for the environment to become ready.
7. Generate/resolve the Code Server URL.
8. Return the launch information.

Example response:

```json
{
  "workspaceId": "123",
  "status": "RUNNING",
  "url": "https://dev.example.com/workspaces/123",
  "expiresAt": null
}
```

---

# 14. Kubernetes Architecture

Each workspace should be isolated from other workspaces.

Example:

```text
Kubernetes Cluster
│
├── devkit-system
│   ├── devkit-api
│   └── devkit-frontend
│
├── workspace-123
│   └── code-server
│
├── workspace-456
│   └── code-server
│
└── workspace-789
    └── code-server
```

Depending on the security and operational requirements, workspace isolation can be implemented using:

- Namespaces
- ResourceQuotas
- LimitRanges
- NetworkPolicies
- Kubernetes RBAC
- PersistentVolumes
- ServiceAccounts
- Pod Security policies/standards

---

# 15. Persistent Workspace Storage

Code Server pods should be treated as disposable.

User files should **not** depend on the lifecycle of the pod.

```text
                 ┌─────────────────────┐
                 │   Persistent Volume │
                 │                     │
                 │ /workspace          │
                 └──────────┬──────────┘
                            │
                            │ Mount
                            ▼
                 ┌─────────────────────┐
                 │    Code Server      │
                 │        Pod          │
                 └─────────────────────┘
```

When a pod is restarted:

```text
Old Pod
   │
   X
Deleted

Persistent Volume
   │
   │ remains
   ▼

New Code Server Pod
   │
   ▼
Same Workspace Files
```

---

# 16. Security

Security is a core requirement.

The system should provide:

- Authentication
- Role-based access control
- Workspace-level authorization
- API authorization
- Kubernetes workload isolation
- Secure Code Server access
- TLS
- Audit logging
- Session management
- Resource quotas

A user must never be able to access a workspace merely by knowing its ID.

For example:

```text
GET /api/v1/workspaces/999
```

must verify:

```text
Current User
      │
      ▼
Has access to Workspace 999?
      │
 ┌────┴────┐
 │         │
YES       NO
 │         │
 ▼         ▼
200       403
```

---

# 17. Audit Logging

Important actions should be audited.

Examples:

```text
USER_LOGIN
USER_LOGOUT

WORKSPACE_CREATED
WORKSPACE_UPDATED
WORKSPACE_DELETED

WORKSPACE_USER_ADDED
WORKSPACE_USER_REMOVED

WORKSPACE_LAUNCHED
WORKSPACE_STOPPED
WORKSPACE_RESTARTED

ADMIN_ROLE_ASSIGNED
ADMIN_ROLE_REMOVED
```

Example audit record:

```json
{
  "actor": "user-123",
  "action": "WORKSPACE_LAUNCHED",
  "resourceType": "WORKSPACE",
  "resourceId": "workspace-456",
  "timestamp": "2026-10-05T14:30:00Z"
}
```

---

# 18. Observability

The platform should expose:

- Application logs
- Kubernetes pod logs
- API metrics
- Workspace lifecycle metrics
- Resource usage
- Audit events
- Health checks

Spring Boot endpoints can expose:

```text
/actuator/health
/actuator/metrics
```

Future monitoring can integrate with:

- Prometheus
- Grafana
- OpenTelemetry

---

# 19. Local Development

## Backend

Requirements:

```text
Java 21+
Maven 3.9+
```

Run:

```bash
cd backend
./mvnw spring-boot:run
```

Backend:

```text
http://localhost:8080
```

---

## Frontend

Requirements:

```text
Node.js
npm
Angular CLI
```

Run:

```bash
cd frontend
npm install
npm start
```

Frontend:

```text
http://localhost:4200
```

---

# 20. Development Environment

For local development, infrastructure can initially be simulated using Docker Compose.

Example:

```text
docker-compose.yml

services:

  postgres:
    image: postgres

  backend:
    build: ./backend

  frontend:
    build: ./frontend
```

Kubernetes integration can then be introduced using:

```text
Minikube
Kind
K3d
```

---

# 21. Configuration

Configuration should be environment-based.

Example backend configuration:

```yaml
devkit:
  kubernetes:
    namespace: devkit
    code-server:
      image: codercom/code-server:latest

  workspace:
    default-cpu: "1"
    default-memory: "2Gi"
```

Environment-specific configuration:

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

Secrets should never be committed to Git.

---

# 22. Deployment

Production deployment:

```text
                    Internet
                       │
                       ▼
                 Load Balancer
                       │
                       ▼
                    Ingress
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
     Angular App              Spring Boot API
                                    │
                                    ▼
                              Kubernetes API
                                    │
                                    ▼
                         Workspace Code Servers
```

---

# 23. Initial Development Phases

## Phase 1 — Foundation

- [ ] Create repository
- [ ] Create Angular application
- [ ] Create Spring Boot application
- [ ] Setup PostgreSQL
- [ ] Setup Docker Compose
- [ ] Establish frontend/backend communication
- [ ] Define initial database migrations

## Phase 2 — Authentication

- [ ] User model
- [ ] Login
- [ ] Logout
- [ ] Token/session management
- [ ] Role model
- [ ] Authorization
- [ ] Angular route guards

## Phase 3 — Project & Workspace Management

- [ ] Project CRUD
- [ ] Workspace CRUD
- [ ] Workspace-user assignment
- [ ] Admin management
- [ ] Workspace permissions

## Phase 4 — Code Server

- [ ] Kubernetes client integration
- [ ] Workspace pod creation
- [ ] Persistent storage
- [ ] Workspace launch
- [ ] Workspace stop
- [ ] Workspace restart
- [ ] Pod status tracking
- [ ] Code Server URL generation

## Phase 5 — Administration

- [ ] Operations dashboard
- [ ] Finance dashboard
- [ ] Usage reporting
- [ ] Resource quotas
- [ ] Audit logs

## Phase 6 — Production Hardening

- [ ] TLS
- [ ] Kubernetes RBAC
- [ ] Network policies
- [ ] Resource limits
- [ ] Monitoring
- [ ] Logging
- [ ] Backup/recovery
- [ ] Security testing
- [ ] Load testing

---

# 24. Engineering Principles

### Backend

- REST-first API design
- Layered architecture
- Domain-driven boundaries
- Stateless API where possible
- Centralized authorization
- Database migrations
- Automated tests
- Kubernetes operations isolated behind services

### Frontend

- Feature-based Angular architecture
- Lazy-loaded features
- Route guards
- Centralized API services
- Reusable components
- Strong typing
- Consistent error handling

### Infrastructure

- Infrastructure as code
- Immutable containers
- Disposable Code Server pods
- Persistent workspace storage
- Resource isolation
- Least-privilege Kubernetes permissions

---

# 25. Target Repository

```text
devkit/
│
├── frontend/                 # Angular application
│
├── backend/                  # Spring Boot application
│
├── infrastructure/           # Kubernetes / Helm
│
├── docs/                     # Architecture and technical docs
│
├── scripts/                  # Developer/CI scripts
│
├── .github/
│   └── workflows/            # CI/CD
│
├── .gitignore
├── docker-compose.yml
├── LICENSE
└── README.md
```

---

# 26. Future Roadmap

Potential future capabilities:

- Git repository integration
- Multiple Code Server images
- Workspace templates
- Workspace snapshots
- Auto-suspend
- Auto-resume
- Scheduled workspace shutdown
- CPU/memory usage tracking
- Cost calculation
- Team collaboration
- Terminal access
- File storage integration
- Secret management
- SSO/OIDC
- Enterprise directory integration
- Multi-cluster Kubernetes support
- Workspace cloning
- Workspace backup/restore

---

# 27. Status

**Project:** Devkit  
**Status:** Initial architecture / development  
**Frontend:** Angular  
**Backend:** Spring Boot  
**Runtime:** Kubernetes  
**IDE Runtime:** Code Server  
**Database:** PostgreSQL  

---

## License

TBD.