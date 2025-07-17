# Sara Dummy Payment Provider Implementation Plan

## Overview
Sara is a dummy payment provider for testing the payment and order system without real integration. It follows the same modular patterns as Mercado Pago but simulates payment processing internally.

## Key Design Decisions
- **Follows Mercado Pago Pattern**: Uses basket for payment creation, not amount/currency
- **No External Dependencies**: No SDK, API keys, or external services  
- **Event Integration**: Full integration with existing order event system
- **Dual Payment Modes**: Card (immediate) and Transfer (webhook-based)

## Implementation Steps

### Step 1: API Module Structure ✅
Create `/api/src/modules/payment/sara/` with:
- [x] **Domain Layer**: Models, interfaces for Sara payment simulation
- [x] **Infrastructure Layer**: Dummy service implementation, repositories
- [x] **Application Layer**: Use cases for payment creation, webhook handling
- [x] **Presentation Layer**: Controllers and routes
- [x] **Module Entry**: `index.ts` with SaraModule factory

### Step 2: Web Module Structure ✅
Create `/web/src/modules/payment/sara/` with:
- [x] **Components**: SaraPaymentForm with card/transfer options
- [x] **Hooks**: useSara for dummy payment flow
- [x] **Services**: SaraService for API communication
- [x] **Types**: TypeScript definitions

### Step 3: Environment Configuration ✅
Add to `.env` files:
- [x] API environment variables
- [x] Web environment variables

### Step 4: Sara Service Implementation ✅
- [x] **Pay Now Mode**: Auto-generate 10-digit card, validate format
- [x] **Transfer Mode**: Simply accept selection, await webhook
- [x] **Rejection Logic**: 10% random rejection for pay now
- [x] **Events**: Trigger order.created, order.paid events

### Step 5: Webhook Endpoint ✅
- [x] Create webhook endpoint at `/api/webhooks/sara`
- [x] Validate webhook secret
- [x] Process payment status updates
- [x] Trigger order events

### Step 6: Payment Form UI ✅
- [x] Card payment form with auto-generated card
- [x] Transfer payment instructions
- [x] Match Mercado Pago styling

### Step 7: Provider Registration ✅
- [x] Update `initializePaymentProviders.ts`
- [x] Add Sara to payment provider selector
- [x] Configure provider capabilities

### Step 8: Testing & Documentation ✅
- [x] Test order creation flow
- [x] Test webhook integration
- [x] Document testing procedures

## Technical Specifications

### Payment Modes
1. **Pay Now (Card)**
   - Auto-generates 10-digit card number
   - Immediate processing
   - 10% random rejection rate
   
2. **Pay with Transfer**
   - Shows bank transfer instructions
   - Awaits webhook confirmation
   - Processes asynchronously

### Webhook Format
```json
{
  "transactionId": "sara_12345",
  "status": "accepted" | "rejected",
  "code": "00" | "01" | "02",
  "reason": "Optional rejection reason"
}
```

### Status Codes
- `00`: Payment accepted
- `01`: Insufficient funds
- `02`: Invalid card/account
- `03`: General rejection

### Environment Variables
```bash
# API (.env)
SARA_ENABLED=true
SARA_REJECTION_RATE=10
SARA_WEBHOOK_SECRET=sara-test-webhook-secret

# Web (.env)
NEXT_PUBLIC_SARA_ENABLED=true
```

## File Structure
```
api/src/modules/payment/sara/
├── domain/
│   ├── models/
│   │   ├── SaraPayment.ts
│   │   └── PaymentTransaction.ts
│   └── services/
│       ├── ISaraService.ts
│       └── IPaymentProvider.ts
├── infrastructure/
│   ├── services/
│   │   └── SaraService.ts
│   └── repositories/
│       └── PostgresPaymentTransactionRepository.ts
├── application/
│   ├── dto/
│   │   └── PaymentDTOs.ts
│   └── usecases/
│       ├── CreatePaymentUseCase.ts
│       ├── HandleWebhookUseCase.ts
│       └── GetPaymentStatusUseCase.ts
├── presentation/
│   ├── controllers/
│   │   └── PaymentController.ts
│   └── routes/
│       └── paymentRoutes.ts
└── index.ts

web/src/modules/payment/sara/
├── components/
│   ├── SaraPaymentForm.tsx
│   └── PaymentModeSelector.tsx
├── hooks/
│   ├── useSara.ts
│   └── usePayment.ts
├── services/
│   └── SaraService.ts
└── types/
    └── payment.ts
```

## Testing Examples

### Create Payment (Card)
```bash
curl -X POST http://localhost:8000/api/v1/modules/payment/sara/payments \
  -H "Content-Type: application/json" \
  -d '{
    "basketId": "basket_123",
    "paymentProviderId": "sara_provider_id",
    "paymentMethodType": "card",
    "paymentToken": "auto-generated-10-digit",
    "customerEmail": "user@example.com"
  }'
```

### Create Payment (Transfer)
```bash
curl -X POST http://localhost:8000/api/v1/modules/payment/sara/payments \
  -H "Content-Type: application/json" \
  -d '{
    "basketId": "basket_123",
    "paymentProviderId": "sara_provider_id", 
    "paymentMethodType": "transfer",
    "paymentToken": "transfer",
    "customerEmail": "user@example.com"
  }'
```

### Webhook Notification
```bash
curl -X POST http://localhost:8000/api/webhooks/sara \
  -H "Content-Type: application/json" \
  -H "X-Sara-Signature: <signature>" \
  -d '{
    "transactionId": "sara_12345",
    "status": "accepted",
    "code": "00"
  }'
```

## Progress Tracking
- ⏳ Not Started
- 🚧 In Progress
- ✅ Completed

## Implementation Summary

Sara payment provider has been successfully implemented as a complete dummy payment system for testing purposes. The implementation includes:

### ✅ Completed Features
1. **API Module**: Complete Clean Architecture implementation with domain, infrastructure, application, and presentation layers
2. **Web Module**: Full React components with hooks and services
3. **Dual Payment Modes**: 
   - Card mode with auto-generated 10-digit numbers
   - Transfer mode with bank details display
4. **Webhook Support**: Full webhook endpoint for processing transfer payments
5. **Provider Registration**: Automatic registration on server startup
6. **UI Integration**: Seamless integration with existing checkout flow
7. **Styling**: Consistent with Mercado Pago styling patterns

### 🔧 Configuration
The system is configured via environment variables and both providers can be enabled simultaneously.

### 📋 Testing
Use the webhook endpoint to simulate bank transfer confirmations and test the complete order flow.

Last Updated: 2025-07-15