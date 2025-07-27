# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

Immich is a self-hosted photo and video management solution with a multi-service architecture:

- **Mobile App**: Flutter/Dart mobile client using Drift database with Riverpod state management
- **Web App**: TypeScript SvelteKit frontend with Tailwind CSS
- **Server**: NestJS/TypeScript backend with Hexagonal Architecture pattern
- **Machine Learning**: Python FastAPI service for AI/ML operations
- **Database**: PostgreSQL with vector extensions for embeddings
- **Cache/Queue**: Redis with BullMQ for background job processing

The server follows Hexagonal Architecture with clear separation between controllers, services, and repositories. Services contain business logic while repositories handle data access.

## Development Commands

### Environment Setup
```bash
# Setup development environment (requires Docker)
make dev

# Setup individual components
make setup-server-dev    # Server only
make setup-web-dev      # Web only (includes SDK build)
```

### Server Development
```bash
# From server/ directory
npm run start:dev        # Development server with hot reload
npm run start:debug      # Debug mode on port 9230
npm run build           # Build production
npm run test            # Run unit tests
npm run test:cov        # Run tests with coverage
npm run test:medium     # Integration tests with database
npm run lint            # ESLint
npm run lint:fix        # Fix linting issues
npm run format          # Check Prettier formatting
npm run format:fix      # Fix formatting
npm run check           # TypeScript type checking

# Database operations
npm run migrations:generate  # Generate new migration
npm run migrations:run      # Run pending migrations
npm run schema:reset       # Drop and recreate schema
npm run sync:sql          # Sync SQL schema files
```

### Web Development
```bash
# From web/ directory
npm run dev             # Development server
npm run build           # Production build
npm run test            # Run tests
npm run test:cov        # Tests with coverage
npm run lint            # ESLint
npm run lint:fix        # Fix linting
npm run format          # Check formatting
npm run format:fix      # Fix formatting
npm run check:svelte    # Svelte component checks
npm run check:typescript # TypeScript checks

# Connect to remote backend
IMMICH_SERVER_URL=https://demo.immich.app/ npm run dev
```

### Mobile Development
```bash
# From mobile/ directory
make build              # Code generation (build_runner)
make watch              # Watch mode for code generation
make pigeon             # Generate platform interface code
make translation        # Generate translation files
make test               # Run Flutter tests
make analyze            # Static analysis with DCM
make format             # Format Dart code
make migration          # Generate Drift database migration
flutter pub get         # Install dependencies
flutter run             # Run app
```

### Multi-Service Operations
```bash
# From root directory
make build-all          # Build all components
make install-all        # Install all dependencies
make check-all          # Type check all components
make lint-all           # Lint all components
make format-all         # Format all components
make test-all           # Test all components
make hygiene-all        # Run all code quality checks

# OpenAPI generation
make open-api           # Generate all OpenAPI clients
make open-api-dart      # Dart client only
make open-api-typescript # TypeScript client only
```

### Docker Development
```bash
make dev                # Start all development services
make dev-down           # Stop development services
make dev-update         # Rebuild and start services
make dev-scale          # Start with multiple server instances
make e2e                # Start E2E test environment
make prod               # Start production environment
```

## Key Architecture Patterns

### Server Architecture (NestJS)
- **Controllers** (`src/controllers/`): HTTP endpoint handlers following REST conventions
- **Services** (`src/services/`): Business logic layer, isolated from external dependencies
- **Repositories** (`src/repositories/`): Data access layer with interface-based design
- **DTOs** (`src/dtos/`): Data Transfer Objects for API input/output validation
- **Middleware** (`src/middleware/`): Cross-cutting concerns (auth, logging, errors)

### Database Schema Management
- **Migrations** (`src/migrations/`): TypeORM migrations for schema changes
- **SQL Queries** (`src/queries/`): Raw SQL files for complex repository operations
- **Schema Tools** (`src/sql-tools/`): Custom schema comparison and generation tools

### Mobile Architecture (Flutter/Dart)
- **Entities** (`lib/entities/`): Database models using Drift ORM
- **Models** (`lib/domain/models/`): Business logic data structures
- **Services** (`lib/domain/services/`): Business logic layer
- **Repositories** (`lib/infrastructure/repositories/`): Data access implementations
- **Providers** (`lib/providers/`): Riverpod state management
- **Pages** (`lib/pages/`): UI screens and navigation

### Background Job System
- Uses BullMQ with Redis for job queues
- Separate microservices worker for job processing
- Job types include: thumbnails, metadata extraction, ML processing, transcoding

### Testing Strategy
- **Unit Tests**: Jest/Vitest for isolated component testing
- **Integration Tests**: Database-backed tests with TestContainers
- **E2E Tests**: Full-stack testing with Playwright
- Mobile testing uses Flutter's built-in test framework

### Code Generation
- **OpenAPI**: Auto-generates client SDKs from server DTOs
- **Build Runner**: Dart code generation for serialization, routing
- **Pigeon**: Type-safe platform channel code for Flutter

## Development Workflow (Change-Test-Verify Loop)

### 1. Environment Setup
```bash
# Initial setup (one-time)
cp docker/example.env docker/.env
# Edit docker/.env to set UPLOAD_LOCATION and other vars
make dev  # Start all services
```

### 2. Development URLs
- **Web App**: http://localhost:3000
- **API**: http://localhost:2283/api  
- **API Docs**: http://localhost:2283/api/docs
- **Database**: localhost:5432 (user: postgres, pass: postgres, db: immich)
- **Mobile Server**: http://your-machine-ip:2283/api

### 3. Change-Test-Verify Cycle

#### Making Changes
```bash
# Server changes (NestJS/TypeScript)
cd server
npm run start:dev  # Hot reload enabled

# Web changes (SvelteKit)
cd web  
npm run dev  # Hot reload enabled

# Mobile changes (Flutter)
cd mobile
flutter run  # Hot reload enabled
```

#### API/DTO Changes (Full Loop)
```bash
# 1. Change server DTOs/endpoints
# 2. Regenerate API clients
make open-api

# 3. Test server changes
cd server
npm run test  # Unit tests
npm run test:medium  # Integration tests with DB

# 4. Update web/mobile clients to use new APIs
# 5. Test client changes
cd web && npm run test
cd mobile && make test
```

#### Testing Commands
```bash
# Individual component testing
make test-server
make test-web  
make test-mobile
make test-e2e  # Full end-to-end tests

# Comprehensive testing
make test-all  # All components
make test-medium  # Server integration tests
```

#### Code Quality Verification
```bash
# Individual component checks
cd server && npm run check  # TypeScript
cd server && npm run lint   # ESLint
cd server && npm run format # Prettier

cd web && npm run check:typescript && npm run check:svelte
cd web && npm run lint && npm run format

cd mobile && make analyze  # DCM + dart analyzer
cd mobile && make format

# All-in-one quality checks
make hygiene-all  # Format, lint, check everything
make check-all    # Type check all components
make lint-all     # Lint all components
make format-all   # Format all components
```

#### Database Operations
```bash
cd server
npm run migrations:run      # Apply pending migrations
npm run migrations:generate # Create new migration
npm run schema:reset       # Reset database (destructive)
npm run sync:sql          # Sync SQL query files
```

#### Development Utilities
```bash
# Stop/restart services
make dev-down     # Stop all services
make dev-update   # Rebuild and restart (after dependency changes)

# Build everything
make build-all    # Build all components
make install-all  # Fresh install all dependencies

# Clean slate
make clean       # Remove node_modules, dist, build folders
```

#### Monitoring Development
```bash
# View logs
docker compose -f docker/docker-compose.dev.yml logs -f immich-server
docker compose -f docker/docker-compose.dev.yml logs -f immich-web

# Debug server (attach debugger to port 9230)
# Debug web (Vite dev tools on port 24678)
```

### 4. Common Development Patterns

#### Adding New Feature
1. Design database schema changes (if needed)
2. Create/update DTOs in `server/src/dtos/`
3. Add controller endpoints in `server/src/controllers/`
4. Implement business logic in `server/src/services/`
5. Add repository methods in `server/src/repositories/`
6. Run `make open-api` to generate clients
7. Update web UI in `web/src/`
8. Update mobile UI in `mobile/lib/`
9. Add tests for all layers
10. Run `make hygiene-all` before commit

#### Debugging Issues
- Server: Attach debugger to port 9230 or check Docker logs
- Web: Use browser dev tools + Vite HMR
- Mobile: Use Flutter dev tools + hot reload
- Database: Connect to localhost:5432 with any Postgres client
- API: Use Swagger docs at http://localhost:2283/api/docs

### Key Configuration Files
- `docker/.env`: Environment variables for development
- `server/src/config.ts`: Server configuration management
- `mobile/pubspec.yaml`: Flutter dependencies and configuration
- `web/package.json`: Web dependencies and build scripts
- `dev-data/`: Local development data (gitignored)