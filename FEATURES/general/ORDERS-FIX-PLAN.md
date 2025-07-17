# ORDERS-FIX-PLAN.md

## Problem Statement
Orders created from baskets have all amount fields as 0 (unit_price, tax_amount, total_amount) because the `createOrderFromBasket` method queries the database instead of using the basket object in memory that already contains all the pricing data.

## Current State
- ✅ Basket in memory has complete pricing data: lineTotal, originalPrice, savings, totalTax, total
- ❌ createOrderFromBasket queries database for basket items (which only store quantity, not pricing)
- ❌ Order creation results in $0.00 amounts for all fields

## Example Basket Data Available in Memory
```json
{
  "items": [
    {
      "lineTotal": { "amount": 3071925, "currency": "COP" },
      "originalPrice": { "amount": 4095900, "currency": "COP" },
      "savings": { "amount": 1023975, "currency": "COP" },
      "quantity": 1,
      "productVariantId": "iphone15pro-128gb-nt"
    }
  ],
  "calculation": {
    "subtotal": { "amount": 3071925, "currency": "COP" },
    "totalTax": { "amount": 583665.75, "currency": "COP" },
    "total": { "amount": 3655590.75, "currency": "COP" },
    "promotionDiscount": { "amount": 1023975, "currency": "COP" }
  }
}
```

## Solution Plan

### Phase 1: Fix Fundamentals (unit_price, tax_amount, total_amount)
1. **Modify OrderService.createOrderFromBasket()** to accept and use the basket object parameter instead of querying database
2. **Update order creation logic** to map basket data to order fields:
   - **Order Level Mapping:**
     - `calculation.subtotal.amount` → `order.subtotal`
     - `calculation.totalTax.amount` → `order.taxAmount`
     - `calculation.promotionDiscount.amount` → `order.discountAmount`
     - `calculation.total.amount` → `order.totalAmount`
   - **Item Level Mapping:**
     - `item.originalPrice.amount / quantity` → `orderItem.unitPrice` (price BEFORE promotions per unit)
     - `item.savings.amount / quantity` → `orderItem.discountAmount` (discount per unit)
     - `0` → `orderItem.taxAmount` (tax is calculated at basket level, not item level)
3. **Remove database queries** for basket items in order creation
4. **Add validation** to ensure basket object has required pricing data

### Phase 2: Complete Discount Implementation
1. **Map promotion data** from basket.appliedPromotions to order.metadata
2. **Set order.discountAmount** from basket.calculation.promotionDiscount
3. **Preserve promotion details** in order for audit/customer service

## Key Changes Needed
- OrderService.createOrderFromBasket() - use basket parameter instead of DB query
- PostgresOrderRepository.create() - accept basket object and map pricing data
- Payment controllers already fetch basket correctly, just need to pass it through

## Expected Outcome
Orders will have correct amounts:
- **Order Level:**
  - subtotal: 9218850 (calculation.subtotal.amount)
  - tax_amount: 1751581.5 (calculation.totalTax.amount)
  - discount_amount: 3072950 (calculation.promotionDiscount.amount)
  - total_amount: 10970431.5 (calculation.total.amount)
- **Item Level:**
  - unit_price: 8195900 (item.originalPrice.amount / quantity - price BEFORE promotions per unit)
  - discount_amount: 2048975 (item.savings.amount / quantity - discount per unit)
  - tax_amount: 0 (tax is calculated at basket level, not item level)

## Files to Modify
1. `/api/src/domain/services/IOrderService.ts` - Update interface
2. `/api/src/application/services/OrderService.ts` - Main logic changes
3. `/api/src/infrastructure/repositories/PostgresOrderRepository.ts` - Data mapping
4. Payment controllers (already working correctly)

## Current Issue Root Cause
The payment controllers call:
```typescript
const basket = await this.basketService.getBasketById(requestData.basketId);
const orderResult = await this.orderService.createOrderFromBasket({
  basketId: requestData.basketId,  // Only passing basketId
  customerId: requestData.customerId,
  paymentTransactionId: result.transaction.id
});
```

But `createOrderFromBasket` should receive the full basket object with pricing data, not just the basketId.