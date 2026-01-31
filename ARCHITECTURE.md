# Fleetbase Architecture

## Overview

Fleetbase is a modular logistics and supply chain operating system (LSOS) built with a modern microservices-inspired architecture. The system separates concerns between backend API services and frontend console applications, enabling flexible deployment and extensibility.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Fleetbase Platform                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐         ┌──────────────┐                      │
│  │   Frontend   │◄────────┤   Backend    │                      │
│  │   Console    │  REST   │   API        │                      │
│  │  (Ember.js)  │  WS     │ (Laravel)    │                      │
│  └──────────────┘         └──────────────┘                      │
│         │                         │                              │
│         │                         ▼                              │
│         │                  ┌──────────────┐                     │
│         │                  │   Services   │                     │
│         │                  └──────────────┘                     │
│         │                         │                              │
│         │          ┌──────────────┼──────────────┐             │
│         │          ▼              ▼              ▼              │
│         │    ┌─────────┐    ┌─────────┐   ┌─────────┐         │
│         │    │  MySQL  │    │  Redis  │   │   S3    │         │
│         │    └─────────┘    └─────────┘   └─────────┘         │
│         │                                                        │
│         └────────────────────────────────────────────────────┐ │
│                                                                │ │
│  ┌────────────────────────────────────────────────────────┐  │ │
│  │                   Extension Modules                     │  │ │
│  ├────────────────────────────────────────────────────────┤  │ │
│  │  FleetOps  │  IAM  │  Pallet  │  Ledger  │  Storefront │  │ │
│  └────────────────────────────────────────────────────────┘  │ │
│                                                                │ │
└────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Backend API (`/api`)

**Technology Stack:**
- **Framework**: Laravel (PHP)
- **Database**: MySQL 8.0
- **Cache**: Redis
- **Real-time**: SocketCluster WebSocket

**Structure:**
```
/api
├── app/               # Application code
│   ├── Http/         # Controllers, Middleware
│   ├── Models/       # Eloquent ORM models
│   ├── Services/     # Business logic
│   └── Events/       # Event handlers
├── config/           # Configuration files
├── database/         # Migrations, seeders
├── routes/           # API routes
└── tests/            # Backend tests
```

**Responsibilities:**
- RESTful API endpoints
- Authentication & authorization
- Business logic processing
- Database operations
- WebSocket connections
- Webhook management
- Background job processing

### 2. Frontend Console (`/console`)

**Technology Stack:**
- **Framework**: Ember.js 5.4+ (Octane edition)
- **Styling**: Tailwind CSS
- **Build Tool**: Ember CLI
- **State Management**: Ember Data
- **Internationalization**: ember-intl

**Structure:**
```
/console
├── app/
│   ├── components/        # Reusable UI components
│   ├── routes/           # Application routes
│   ├── services/         # Ember services
│   ├── models/           # Data models
│   ├── adapters/         # API adapters
│   └── serializers/      # Data serializers
├── translations/         # i18n translation files
├── public/              # Static assets
└── tests/               # Frontend tests
```

**Responsibilities:**
- User interface rendering
- User interaction handling
- API communication
- Real-time updates via WebSocket
- State management
- Internationalization

### 3. Extension System (`/packages`)

Fleetbase uses a modular extension architecture allowing functionality to be added or removed.

**Core Extensions:**

1. **FleetOps** - Fleet and order management
2. **IAM Engine** - Identity and Access Management
3. **Pallet** - Warehouse and inventory management
4. **Ledger** - Accounting and invoicing
5. **Storefront** - E-commerce functionality
6. **Dev Engine** - Developer tools and console
7. **Registry Bridge** - Extension registry integration

**Extension Structure:**
```
/packages/{extension-name}
├── addon/               # Frontend code (Ember addon)
│   ├── components/
│   ├── services/
│   └── routes/
├── server/              # Backend code (Laravel package)
│   ├── src/
│   │   ├── Http/
│   │   ├── Models/
│   │   └── Services/
│   └── routes/
└── translations/        # Extension-specific translations
```

## Data Flow

### 1. Request Flow

```
User Action (Browser)
    ↓
Ember Route/Component
    ↓
Ember Service/Adapter
    ↓
HTTP/WebSocket Request
    ↓
Laravel Route
    ↓
Controller
    ↓
Service Layer
    ↓
Model/Repository
    ↓
Database
    ↓
Response (JSON)
    ↓
Ember Serializer
    ↓
Ember Data Store
    ↓
Component Update
    ↓
UI Update
```

### 2. Real-Time Updates

```
Server Event
    ↓
SocketCluster Server
    ↓
WebSocket Connection
    ↓
Ember Socket Service
    ↓
Application State Update
    ↓
Component Re-render
```

## Key Design Patterns

### Backend Patterns

1. **Repository Pattern**: Data access abstraction
2. **Service Layer**: Business logic encapsulation
3. **Event-Driven**: Loosely coupled components
4. **Dependency Injection**: Laravel's service container
5. **API Resources**: Consistent API responses

### Frontend Patterns

1. **Component-Based**: Reusable UI components
2. **Data Down, Actions Up**: Unidirectional data flow
3. **Service-Oriented**: Shared functionality via services
4. **Route-Based**: URL-driven application state
5. **Adapter Pattern**: API communication abstraction

## Security Architecture

### Authentication

- **JWT Tokens**: Stateless authentication
- **API Keys**: For programmatic access
- **OAuth 2.0**: Third-party integrations
- **Session Management**: Secure session handling

### Authorization

- **Role-Based Access Control (RBAC)**: User roles and permissions
- **Resource Policies**: Fine-grained access control
- **API Rate Limiting**: Prevent abuse
- **CORS Configuration**: Cross-origin security

### Data Protection

- **Encryption at Rest**: Database encryption
- **Encryption in Transit**: TLS/SSL
- **Secrets Management**: Environment variables
- **Input Validation**: XSS and SQL injection prevention

## Scalability Considerations

### Horizontal Scaling

- **Stateless API**: Can run multiple instances
- **Redis Session Store**: Shared session storage
- **CDN Integration**: Static asset distribution
- **Database Read Replicas**: Read scaling

### Performance Optimization

- **Caching Strategy**:
  - Redis for application cache
  - Browser caching for static assets
  - Query result caching
  
- **Database Optimization**:
  - Indexed queries
  - Connection pooling
  - Query optimization

- **Asset Optimization**:
  - Code splitting
  - Lazy loading
  - Minification and compression

## Deployment Architecture

### Docker Deployment

```
docker-compose.yml
├── application      # Laravel API container
├── console          # Ember.js console container
├── socket           # WebSocket server container
├── mysql            # Database container
├── redis            # Cache container
├── ipinfo           # IP geolocation service
└── nginx            # Reverse proxy (optional)
```

### AWS Cloud Deployment

- **Compute**: ECS Fargate (containerized services)
- **Database**: RDS MySQL (managed database)
- **Cache**: ElastiCache Redis
- **Storage**: S3 + CloudFront
- **Load Balancing**: Application Load Balancer
- **Monitoring**: CloudWatch

## Extension Development

### Creating an Extension

1. **Backend Component** (Laravel Package)
   - Create service provider
   - Define routes
   - Implement models and controllers
   - Register migrations

2. **Frontend Component** (Ember Addon)
   - Create addon structure
   - Define routes and components
   - Implement services
   - Add translations

3. **Integration**
   - Register extension in Fleetbase
   - Configure permissions
   - Add menu items
   - Implement hooks

### Extension Lifecycle

1. **Discovery**: Extension registry lookup
2. **Installation**: Package installation
3. **Activation**: Enable in system
4. **Configuration**: Setup and customization
5. **Updates**: Version management
6. **Deactivation**: Disable functionality
7. **Uninstallation**: Complete removal

## API Architecture

### REST API Design

- **Versioning**: URL-based versioning (`/api/v1`)
- **Resource-Based**: RESTful resource structure
- **HTTP Methods**: Standard CRUD operations
- **Status Codes**: Consistent HTTP status codes
- **Pagination**: Cursor-based pagination
- **Filtering**: Query parameter filtering
- **Sorting**: Flexible sorting options

### WebSocket API

- **Channels**: Topic-based subscriptions
- **Events**: Real-time event broadcasting
- **Authentication**: Token-based connection auth
- **Reconnection**: Automatic reconnection handling

## Internationalization (i18n)

### Translation System

- **Format**: YAML translation files
- **Locales**: 12+ supported languages
- **Fallback**: English (en-US) as default
- **Context**: Namespace-based organization
- **Pluralization**: ICU message format
- **Dynamic Loading**: Async locale loading

### Supported Languages

- English (US/UK)
- Arabic (UAE)
- Chinese (Simplified)
- Spanish (Spain/Mexico)
- French
- Portuguese (Brazil)
- Russian
- Vietnamese
- Bulgarian
- Persian
- Mongolian

## Testing Strategy

### Backend Testing

- **Unit Tests**: PHPUnit for isolated tests
- **Feature Tests**: API endpoint testing
- **Integration Tests**: Service integration
- **Database Tests**: Migration and seeding

### Frontend Testing

- **Unit Tests**: Component isolation tests
- **Integration Tests**: Service and route tests
- **Acceptance Tests**: End-to-end user flows
- **Visual Regression**: UI consistency tests

## Monitoring and Observability

### Logging

- **Application Logs**: Laravel log channels
- **Access Logs**: Web server logs
- **Error Tracking**: Exception monitoring
- **Audit Logs**: User action tracking

### Metrics

- **Performance Metrics**: Response times, throughput
- **Business Metrics**: Orders, deliveries, revenue
- **System Metrics**: CPU, memory, disk usage
- **Database Metrics**: Query performance, connections

## Technology Choices Rationale

### Why Laravel?

- Mature PHP framework with excellent ecosystem
- Built-in authentication and authorization
- Eloquent ORM for database operations
- Queue system for background jobs
- Rich package ecosystem

### Why Ember.js?

- Convention over configuration
- Excellent routing and state management
- Strong TypeScript support
- Long-term stability guarantee
- Built-in testing framework

### Why Docker?

- Consistent development environment
- Easy deployment and scaling
- Service isolation
- Infrastructure as code
- Cross-platform compatibility

## Future Architecture Considerations

### Microservices Migration

- Potential service separation for scalability
- API gateway implementation
- Service mesh for inter-service communication
- Event-driven architecture expansion

### AI Integration

- Machine learning model integration
- Predictive analytics
- Automated decision making
- Natural language processing

### Mobile-First Architecture

- Progressive Web App (PWA) support
- Native mobile app integration
- Offline-first data synchronization
- Push notification system

## References

- [Laravel Documentation](https://laravel.com/docs)
- [Ember.js Guides](https://guides.emberjs.com/)
- [Docker Documentation](https://docs.docker.com/)
- [Fleetbase Documentation](https://docs.fleetbase.io/)
