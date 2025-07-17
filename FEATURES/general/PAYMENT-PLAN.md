# PAYMENT-PLAN.md - MercadoPago Payment Module Implementation

## Overview
This document tracks the implementation of a modular payment system for Museo, starting with MercadoPago and designed to support multiple payment providers in the future. The module will be self-contained and integrate seamlessly with the existing checkout flow.

## Core Architecture Decisions
- **Multi-Provider Support**: Payment modules are self-contained and can be activated/deactivated via environment variables
- **Module Isolation**: All payment-related code lives in `src/modules/payment/` directories
- **Clean Architecture**: Following existing patterns with proper separation of concerns
- **UI Preservation**: Keep current checkout UI, integrate payment providers underneath
- **Security First**: Tokenization, PCI compliance, encrypted storage

## Implementation Status Tracker

### Phase 1: Database Foundation ✅ COMPLETED

#### 1.1 Payment Providers Table ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1751800000001_create_payment_providers.ts`
- [x] Create payment_provider_type enum (mercado_pago, stripe, etc.)
- [x] Create payment_providers table with configuration
- [x] Add indexes and constraints
- [x] Create view for active providers
- [x] Add updated_at trigger and single default provider constraint

#### 1.2 Payment Transactions Table ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1751800000002_create_payment_transactions.ts`
- [x] Create payment_transaction_status enum
- [x] Create payment_transactions table
- [x] Link to customers and baskets
- [x] Add audit fields and encrypted token storage
- [x] Create views for recent transactions and statistics
- [x] Add net amount calculation trigger

### Phase 2: API Module Foundation ✅ COMPLETED

#### 2.1 Payment Module Structure ✅ COMPLETED
**Structure**: 
```
/api/src/modules/payment/
  mercado-pago/
    domain/
      models/
        PaymentTransaction.ts
        MercadoPagoPayment.ts
      services/
        IPaymentProvider.ts
        IMercadoPagoService.ts
    infrastructure/
      services/
        MercadoPagoService.ts
    application/
      dto/
        PaymentDTOs.ts
      usecases/
        CreatePaymentUseCase.ts
        HandleWebhookUseCase.ts
        GetPaymentStatusUseCase.ts
```

#### 2.2 MercadoPago Domain Layer ✅ COMPLETED
- [x] PaymentTransaction entity with full state management
- [x] MercadoPagoPayment entity with status mapping and validation
- [x] IPaymentProvider interface for multi-provider support
- [x] IMercadoPagoService interface with MercadoPago-specific methods

#### 2.3 MercadoPago Infrastructure ✅ COMPLETED
- [x] MercadoPagoService implementation with full API integration
- [x] Token encryption/decryption (framework in place)
- [x] API communication layer with error handling
- [x] Webhook validation and processing

### Phase 3: API Application Layer ✅ COMPLETED

#### 3.1 Payment DTOs ✅ COMPLETED
- [x] CreatePaymentRequest/Response with validation
- [x] PaymentStatusResponse with comprehensive status info
- [x] WebhookPayload validation and processing DTOs
- [x] MercadoPago-specific DTOs for tokens and installments
- [x] PaymentDTOMapper utility with validation methods

#### 3.2 Payment Use Cases ✅ COMPLETED
- [x] CreatePaymentUseCase with full payment flow
- [x] HandleWebhookUseCase with order integration
- [x] GetPaymentStatusUseCase with status refresh
- [x] Repository interfaces for data access
- [x] Error handling and retry logic

### Phase 4: API Presentation Layer ✅ COMPLETED

#### 4.1 Payment Controller ✅ COMPLETED
- [x] PaymentController with comprehensive endpoint handlers
- [x] Webhook endpoints with validation and processing
- [x] Error handling with specific payment error codes
- [x] Input validation with express-validator
- [x] Repository pattern integration

#### 4.2 Module Registration ✅ COMPLETED
- [x] Payment module router with conditional activation
- [x] Environment-based module enabling/disabling
- [x] Webhook routes separation for security
- [x] Integration guide and module factory pattern
- [x] Repository implementations (PostgreSQL)

#### 4.3 Additional Features ✅ COMPLETED
- [x] Card token validation endpoint
- [x] Installment options endpoint
- [x] Payment statistics endpoint (admin)
- [x] Payment retry functionality
- [x] Comprehensive error handling middleware

### Phase 5: Web Payment Module ❌ NOT STARTED

#### 5.1 Payment Module Structure ❌ NOT STARTED
**Structure**:
```
/web/src/modules/payment/
  mercado-pago/
    components/
    services/
    types/
    hooks/
```

#### 5.2 MercadoPago Components ❌ NOT STARTED
- [ ] MercadoPagoProvider wrapper
- [ ] PaymentForm integration
- [ ] Error handling components

#### 5.3 Payment Service Integration ❌ NOT STARTED
- [ ] PaymentService with provider selection
- [ ] MercadoPagoService implementation
- [ ] Token management
- [ ] Payment status polling

### Phase 6: Checkout Integration ❌ NOT STARTED

#### 6.1 Payment Provider Selection ❌ NOT STARTED
- [ ] Modify CheckoutPayment to support multiple providers
- [ ] Provider selection UI (when multiple active)
- [ ] Default provider logic

#### 6.2 Payment Flow Integration ❌ NOT STARTED
- [ ] Replace mock payment with real providers
- [ ] Maintain existing 3-step flow
- [ ] Add payment status handling
- [ ] Order completion logic

### Phase 7: Testing & Security ❌ NOT STARTED

#### 7.1 Module Tests ❌ NOT STARTED
- [ ] Unit tests for MercadoPago service
- [ ] Integration tests for payment flow
- [ ] Webhook handling tests
- [ ] E2E payment tests

#### 7.2 Security Audit ❌ NOT STARTED
- [ ] Token encryption verification
- [ ] PCI compliance validation
- [ ] Webhook signature verification
- [ ] Environment variable security

## Environment Variables Needed

### API (.env)
```
# MercadoPago Configuration
MERCADO_PAGO_ENABLED=true
MERCADO_PAGO_ACCESS_TOKEN=
MERCADO_PAGO_WEBHOOK_SECRET=

# Payment Encryption
PAYMENT_ENCRYPTION_KEY=
```

### Web (.env.local)  
```
# MercadoPago Public Configuration
NEXT_PUBLIC_MERCADO_PAGO_ENABLED=true
NEXT_PUBLIC_MERCADO_PAGO_PUBLIC_KEY=
```

## Module Architecture

### Payment Provider Interface
- Standardized interface for all payment providers
- Provider-specific implementations in separate modules
- Environment-based activation/deactivation
- Unified error handling and status reporting

### Database Design
- Generic payment_transactions table
- Provider-specific configuration in payment_providers
- Encrypted token storage for security
- Audit trail for all payment activities

### Integration Points
- Seamless integration with existing checkout flow
- Backward-compatible API design
- Minimal changes to existing CheckoutPayment component
- Support for multiple active providers

## Success Criteria
- [ ] MercadoPago credit card payments working
- [ ] Existing checkout UI preserved
- [ ] Module can be disabled via environment variables
- [ ] Ready for additional payment providers
- [ ] All security requirements met
- [ ] Tests passing and security audit complete

## Notes
- Start with MercadoPago as proof of concept
- Design for multiple providers from the beginning
- Maintain existing user experience
- Follow existing code patterns and architecture
- Ensure all payment data is properly encrypted
- Implement comprehensive error handling and logging