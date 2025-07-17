# AUTH-PLAN.md - Multi-Provider OAuth Implementation

## Overview
This document tracks the implementation of a multi-provider OAuth system for Museo, starting with Google OAuth and designed to support LinkedIn, Facebook, and other providers in the future.

## Core Architecture Decisions
- **Multi-Provider Support**: Users can link multiple social accounts to one customer account
- **Clean Architecture**: Following the existing pattern in API with proper separation of concerns
- **Security First**: Encrypted tokens, proper validation, rate limiting
- **Progressive Enhancement**: Start with Google, easy to add more providers

## Implementation Status Tracker

### Phase 1: Database Foundation ✅ COMPLETED

#### 1.1 Customer Auth Fields Migration ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1751700000010_add_customer_auth_fields.ts`
- [x] Add password_hash field (nullable)
- [x] Add email_verified field (default false)
- [x] Add email_verification_token field
- [x] Add password_reset_token field
- [x] Add password_reset_expires field
- [x] Added additional security fields (failed_login_attempts, locked_until)
- [x] Test migration up/down

#### 1.2 Customer Auth Providers Table ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1751700000011_create_customer_auth_providers.ts`
```sql
CREATE TABLE customer_auth_providers (
  id VARCHAR(100) PRIMARY KEY,
  customer_id VARCHAR(100) NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
  provider auth_provider_type NOT NULL,  -- enum type
  provider_id VARCHAR(255) NOT NULL,
  email VARCHAR(255),
  display_name VARCHAR(255),
  profile_picture TEXT,
  profile_data JSONB,
  access_token TEXT,
  refresh_token TEXT,
  token_expires_at TIMESTAMP WITH TIME ZONE,
  scopes TEXT[],
  is_primary BOOLEAN DEFAULT FALSE,
  linked_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  last_used_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(provider, provider_id)
);
```
- [x] Create table structure with enum type
- [x] Add indexes (customer_id, provider, email, primary)
- [x] Add updated_at trigger
- [x] Create customer_auth_summary view
- [x] Create link_auth_provider function
- [x] Test migration up/down

### Phase 2: API Domain Layer ✅ COMPLETED

#### 2.1 Domain Entities ✅ COMPLETED
**Files**:
- `/api/src/domain/models/Customer.ts`
  - [x] Create Customer interface with auth properties
  - [x] Create CustomerEntity class with auth methods
  - [x] Add password validation methods
  - [x] Add email verification methods
  - [x] Add account locking logic
  
- `/api/src/domain/models/AuthProvider.ts`
  - [x] Create AuthProviderEntity class
  - [x] Add OAuth profile and token interfaces
  - [x] Add provider validation methods
  - [x] Add authorization URL generation

#### 2.2 Domain Interfaces ✅ COMPLETED
**Files**:
- `/api/src/domain/repositories/ICustomerRepository.ts`
  - [x] Full CRUD operations interface
  - [x] Auth-specific methods (tokens, verification)
  - [x] Search and validation helpers
  - [x] Address and preferences management
  
- `/api/src/domain/repositories/IAuthProviderRepository.ts`
  - [x] Complete auth provider repository interface
  - [x] Provider linking/unlinking operations
  - [x] Token management methods
  - [x] Validation and statistics methods
  
- `/api/src/domain/services/IAuthService.ts`
  - [x] Authentication methods (password + OAuth)
  - [x] Token generation/validation
  - [x] Email verification and password reset
  - [x] Session management
  
- `/api/src/domain/services/IOAuthService.ts`
  - [x] OAuth flow management interface
  - [x] Provider-specific methods
  - [x] Token refresh and validation
  
- `/api/src/domain/services/IPasswordService.ts`
  - [x] Password hashing and validation
  - [x] Password policy enforcement
  - [x] Password history management

### Phase 3: API Infrastructure Layer ✅ COMPLETED (Core Components)

#### 3.1 Repository Implementations ✅ COMPLETED
**Files**:
- `/api/src/infrastructure/repositories/PostgresCustomerRepository.ts`
  - [x] Implement complete customer repository
  - [x] Add auth-related methods (tokens, verification, locking)
  - [x] Add address and preferences management
  - [x] Add proper error handling and transactions
  
- `/api/src/infrastructure/repositories/PostgresAuthProviderRepository.ts`
  - [x] Implement complete auth provider repository
  - [x] Add token encryption/decryption
  - [x] Add provider linking/unlinking with safety checks
  - [x] Add comprehensive search and statistics methods

#### 3.2 External Services ✅ COMPLETED
**Files**:
- `/api/src/infrastructure/services/JWTService.ts`
  - [x] Implement complete JWT token service
  - [x] Access and refresh token generation
  - [x] Token validation and parsing
  - [x] Special tokens (email verification, password reset)
  
- `/api/src/infrastructure/services/EncryptionService.ts`
  - [x] Implement AES-256-GCM encryption
  - [x] Token encryption/decryption for secure storage
  - [x] Secure random token generation
  - [x] HMAC signing and verification
  
- `/api/src/infrastructure/services/GoogleOAuthService.ts`
  - [x] Complete Google OAuth 2.0 implementation
  - [x] Authorization URL generation
  - [x] Code exchange for tokens
  - [x] Profile fetching and token refresh
  - [x] Token validation and revocation

#### 3.3 Middleware ⏸️ DEFERRED
**File**: `/api/src/infrastructure/middleware/AuthMiddleware.ts`
- [ ] JWT validation middleware (implement in Phase 5)
- [ ] User injection into request
- [ ] Role-based access control
- [ ] Rate limiting for auth endpoints

### Phase 4: API Application Layer ✅ COMPLETED

#### 4.1 DTOs ✅ COMPLETED
**Files**:
- `/api/src/application/dto/auth/AuthDTOs.ts`
  - [x] Complete set of request/response DTOs
  - [x] Validation helpers for auth requests
  - [x] Mapping functions for domain to DTO conversion
  - [x] Error response DTOs

#### 4.2 Use Cases ✅ COMPLETED
**Files**:
- `/api/src/application/usecases/auth/AuthenticateWithGoogleUseCase.ts`
  - [x] Handle Google OAuth callback
  - [x] Create new customers or link existing ones
  - [x] Generate JWT tokens
- `/api/src/application/usecases/auth/RefreshTokenUseCase.ts`
  - [x] Validate refresh tokens
  - [x] Generate new token pairs
- `/api/src/application/usecases/auth/GetCurrentUserUseCase.ts`
  - [x] Get authenticated user with providers
- `/api/src/application/usecases/auth/InitiateGoogleOAuthUseCase.ts`
  - [x] Generate OAuth authorization URLs
  - [x] Secure state parameter handling

### Phase 5: API Presentation Layer ✅ COMPLETED

#### 5.1 Controllers ✅ COMPLETED
**File**: `/api/src/api/controllers/AuthController.ts`
- [x] Complete auth controller with error handling
- [x] OAuth initiation and callback endpoints
- [x] Token refresh and logout endpoints
- [x] Current user and auth status endpoints
- [x] Secure cookie management

#### 5.2 Routes & Validation ✅ COMPLETED
**Files**:
- `/api/src/api/routes/authRoutes.ts`
  - [x] Complete auth route definitions
  - [x] Auth middleware for protecting routes
  - [x] Optional auth middleware
  - [x] Integrated with main router
  
#### 5.3 Dependency Injection ⏸️ SIMPLIFIED
- [x] Direct instantiation in routes (simple approach)
- [ ] Full DI container integration (can be done later)

### Phase 6: Web Authentication ✅ COMPLETED

#### 6.1 Auth Service ✅ COMPLETED
**File**: `/web/src/shared/services/AuthService.ts`
- [x] Complete AuthService extending BaseAPIService
- [x] Google OAuth flow (initiate, callback, handle)
- [x] Token management with automatic refresh
- [x] Current user and auth status methods
- [x] Secure token storage in localStorage
- [x] Automatic token refresh logic

#### 6.2 Auth Store ✅ COMPLETED
**File**: `/web/src/shared/stores/AuthStore.ts`
- [x] Complete Zustand store with persistence
- [x] User and provider state management
- [x] Authentication status and loading states
- [x] Error and success message handling
- [x] Auth flow actions (login, logout, refresh)
- [x] Auto-initialization on app start

#### 6.3 Components ✅ COMPLETED
**Files**:
- `/web/src/shared/components/molecules/GoogleOAuthButton/`
  - [x] Complete Google OAuth button component
  - [x] Loading states and error handling
  - [x] Proper Google branding and icons
  - [x] Accessibility support
  
- `/web/src/shared/components/molecules/CheckoutAuth/CheckoutAuth.tsx`
  - [x] Updated to use real Google OAuth
  - [x] Automatic redirect when authenticated
  - [x] Error handling and user feedback

#### 6.4 Integration ✅ COMPLETED
- [x] OAuth callback page (`/app/auth/callback/page.tsx`)
- [x] Updated CheckoutAuth with real authentication
- [x] Auth store initialization in AppStateWrapper
- [x] Added auth translations (en.json)
- [x] Integrated with existing UI patterns

### Phase 7: Testing & Security ❌ NOT STARTED

#### 7.1 API Tests ❌ NOT STARTED
- [ ] Unit tests for repositories
- [ ] Unit tests for services
- [ ] Integration tests for auth flow
- [ ] E2E tests for full OAuth flow

#### 7.2 Security Audit ❌ NOT STARTED
- [ ] Token encryption verification
- [ ] SQL injection prevention
- [ ] CSRF protection
- [ ] Rate limiting verification
- [ ] Secure cookie configuration

## Environment Variables Needed

### API (.env)
```
JWT_SECRET=
JWT_REFRESH_SECRET=
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_CALLBACK_URL=http://localhost:8000/api/auth/google/callback

ENCRYPTION_KEY=
```

### Web (.env.local)
```
NEXT_PUBLIC_GOOGLE_CLIENT_ID=
NEXT_PUBLIC_API_URL=http://localhost:8000
```

## Success Criteria
- [ ] User can sign in with Google
- [ ] User can link multiple providers
- [ ] Tokens are properly encrypted
- [ ] Clean Architecture maintained
- [ ] All tests passing
- [ ] Security audit passed

## Notes
- Each phase should be completed and tested before moving to the next
- Update this document after completing each checklist item
- Follow CORE PRINCIPLES: No shortcuts, proper architecture
- Ensure all text is internationalized
- Maintain existing code style and patterns