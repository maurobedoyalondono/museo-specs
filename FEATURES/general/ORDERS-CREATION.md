# ORDERS-CREATION.md - Order Management System Implementation

## Overview
This document tracks the implementation of the Order Management System for Museo, building upon the existing payment infrastructure to create a complete purchase flow from checkout to order fulfillment.

## Current Status Summary

### What Exists:
- ✅ Baskets (shopping cart) system
- ✅ Payment processing with MercadoPago
- ✅ Customer management system
- ✅ Payment transactions tracking
- ✅ Product inventory system

### What's Missing:
- ❌ Order entity and database tables
- ❌ Order creation after payment
- ❌ Event-driven architecture
- ❌ Email notifications
- ❌ Order status tracking
- ❌ Web UI issues in payment form

## Implementation Plan

### Phase 1: Fix MercadoPago Form Issues ❌ NOT STARTED
**Estimated Time**: 1-2 hours

#### 1.1 Form Integration Issues
- [ ] Fix CheckoutPayment component props to match CheckoutPage expectations
- [ ] Update data flow between components
- [ ] Fix the missing `data` and `onSubmit` props issue
- [ ] Ensure payment data persists through checkout steps

#### 1.2 Payment Form Validation
- [ ] Verify MercadoPago SDK initialization
- [ ] Fix validation method calls (validateCardNumber, validateExpiryDate, etc.)
- [ ] Add proper error handling for missing public key
- [ ] Ensure environment variables are loaded correctly

#### 1.3 Checkout Flow Integration
- [ ] Update CheckoutPage to properly handle payment callbacks
- [ ] Fix state management between checkout steps
- [ ] Add payment processing states
- [ ] Handle payment success/failure scenarios

### Phase 2: Database - Create Order Tables ✅ COMPLETED
**Estimated Time**: 2-3 hours

#### 2.1 Orders Table Migration ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1751900000001_create_orders.ts`
```sql
- order_status enum (pending, processing, paid, shipped, delivered, cancelled, refunded)
- orders table:
  - id (UUID)
  - order_number (unique, human-readable)
  - customer_id (FK to customers)
  - payment_transaction_id (FK to payment_transactions)
  - status (order_status)
  - subtotal, tax_amount, shipping_amount, discount_amount, total_amount
  - currency, locale
  - billing_address_id, shipping_address_id (FK to customer_addresses)
  - notes, internal_notes
  - metadata (JSONB)
  - created_at, updated_at, paid_at, shipped_at, delivered_at, cancelled_at
```

#### 2.2 Order Items Table Migration ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1751900000002_create_order_items.ts`
```sql
- order_items table:
  - id (UUID)
  - order_id (FK to orders)
  - product_variant_id (FK to product_variants)
  - quantity, unit_price, discount_amount, tax_amount, total_amount
  - metadata (JSONB)
  - created_at, updated_at
```

#### 2.3 Order Status History Migration ✅ COMPLETED
**File**: `/infrastructure/migrations/db/1751900000003_create_order_status_history.ts`
```sql
- order_status_history table:
  - id (UUID)
  - order_id (FK to orders)
  - previous_status, new_status
  - changed_by (user_id or 'system')
  - reason, notes
  - created_at
```

### Phase 3: API - Order Management Module ✅ COMPLETED
**Estimated Time**: 4-5 hours

#### 3.1 Order Domain Layer ✅ COMPLETED
**Structure**:
```
/api/src/domain/order/
  models/
    Order.ts
    OrderItem.ts
    OrderStatus.ts
  services/
    IOrderService.ts
    IOrderNumberGenerator.ts
  events/
    OrderCreatedEvent.ts
    OrderPaidEvent.ts
    OrderStatusChangedEvent.ts
```

#### 3.2 Order Infrastructure ✅ COMPLETED
**Structure**:
```
/api/src/infrastructure/order/
  repositories/
    OrderRepository.ts
  services/
    OrderNumberGenerator.ts (format: ORD-YYYYMMDD-XXXXX)
```

#### 3.3 Order Application Layer ✅ COMPLETED
**Structure**:
```
/api/src/application/order/
  dto/
    CreateOrderDTO.ts
    OrderResponseDTO.ts
    OrderItemDTO.ts
  usecases/
    CreateOrderFromBasketUseCase.ts
    UpdateOrderStatusUseCase.ts
    GetOrderDetailsUseCase.ts
    ListCustomerOrdersUseCase.ts
```

#### 3.4 Order Presentation Layer ✅ COMPLETED
**Structure**:
```
/api/src/presentation/order/
  controllers/
    OrderController.ts
  routes/
    orderRoutes.ts
```

**Endpoints**:
- `POST /api/v1/orders` - Create order from basket
- `GET /api/v1/orders/:orderId` - Get order details
- `GET /api/v1/orders` - List customer orders
- `PATCH /api/v1/orders/:orderId/status` - Update order status
- `GET /api/v1/orders/:orderId/invoice` - Get order invoice

### Phase 4: Event-Driven Architecture ❌ NOT STARTED
**Estimated Time**: 3-4 hours

#### 4.1 Event System Core
**Structure**:
```
/api/src/infrastructure/events/
  EventBus.ts
  IEventHandler.ts
  EventDispatcher.ts
```

#### 4.2 Email Service
**Structure**:
```
/api/src/infrastructure/email/
  EmailService.ts (using nodemailer with Gmail)
  templates/
    orderConfirmation.ts
    paymentReceived.ts
    orderShipped.ts
```

**Environment Variables**:
```env
# Email Configuration
EMAIL_ENABLED=true
EMAIL_FROM=noreply@museostore.com
GMAIL_USER=
GMAIL_APP_PASSWORD=

# MercadoPago Configuration
NEXT_PUBLIC_MERCADO_PAGO_INSTALLMENTS_ENABLED=false
```

#### 4.3 Event Handlers
**Structure**:
```
/api/src/application/events/handlers/
  OrderCreatedHandler.ts
  OrderPaidHandler.ts
  PaymentFailedHandler.ts
  OrderStatusChangedHandler.ts
```

### Phase 5: Payment Flow Integration ❌ NOT STARTED
**Estimated Time**: 2-3 hours

#### 5.1 Update Payment Success Flow
- [ ] Modify HandleWebhookUseCase to trigger order creation
- [ ] Create order after payment approval
- [ ] Link payment transaction to order
- [ ] Mark basket as converted
- [ ] Trigger order.created event

#### 5.2 Payment-Order Integration
- [ ] Add order creation logic in payment webhook handler
- [ ] Handle edge cases (duplicate webhooks, race conditions)
- [ ] Implement idempotency for order creation
- [ ] Add transaction rollback on failures

### Phase 6: Web UI Enhancements ❌ NOT STARTED
**Estimated Time**: 2-3 hours

#### 6.1 Order Confirmation Page
**File**: `/web/src/app/order-confirmation/[orderId]/page.tsx`
- [ ] Create order confirmation component
- [ ] Display order number and summary
- [ ] Show estimated delivery information
- [ ] Add print/download receipt functionality

#### 6.2 Order Management UI
**Files**:
- `/web/src/app/account/orders/page.tsx` - Order list
- `/web/src/app/account/orders/[orderId]/page.tsx` - Order details
- `/web/src/shared/components/molecules/OrderList/`
- `/web/src/shared/components/molecules/OrderDetails/`
- `/web/src/shared/services/OrderService.ts`

## Architecture Decisions

### 1. Payment-First Approach
- Payment is processed and confirmed before order creation
- Ensures no orders without successful payment
- Basket remains until order is successfully created

### 2. Event-Driven Design
- Loose coupling between modules
- Async processing for non-critical tasks (emails)
- Easy to add new event handlers without modifying core

### 3. Order Immutability
- Orders cannot be modified after creation
- Use status changes and notes for tracking updates
- Maintains audit trail and data integrity

### 4. Idempotency
- Webhook handlers are idempotent
- Prevents duplicate order creation
- Uses payment transaction ID as idempotency key

## Implementation Sequence

1. **First**: Fix MercadoPago form issues (blocking payment flow)
2. **Second**: Create database migrations (foundation for orders)
3. **Third**: Build Order API module (core functionality)
4. **Fourth**: Implement event system (notifications)
5. **Fifth**: Integrate payment with order creation
6. **Sixth**: Enhance web UI with order features

## Success Criteria

- [ ] Users can complete payment through MercadoPago
- [ ] Orders are created automatically after successful payment
- [ ] Customers receive order confirmation emails
- [ ] Order status can be tracked and updated
- [ ] Order history is available in customer account
- [ ] System handles payment failures gracefully
- [ ] No duplicate orders from webhook retries

## Testing Requirements

- [ ] Unit tests for order creation logic
- [ ] Integration tests for payment-to-order flow
- [ ] E2E tests for complete checkout process
- [ ] Load tests for webhook handling
- [ ] Email delivery tests

## Security Considerations

- [ ] Validate all order data before creation
- [ ] Ensure proper authentication for order access
- [ ] Implement rate limiting on order endpoints
- [ ] Sanitize data in emails
- [ ] Log all order status changes

## Notes

- Gmail SMTP requires app-specific password
- Order numbers should be human-readable
- Consider timezone for order timestamps
- Plan for order export/reporting features
- Design for multi-currency support from start