# KIVO CLIENT - Implementation Plan

## Overview
Step-by-step plan to implement the KIVO minimalist commerce client, following the same patterns and architecture as Museo web.

## Pre-Implementation Checklist
- [ ] User stories approved
- [ ] Technical design approved  
- [ ] Development environment ready
- [ ] Access to Museo web codebase for reference

## Implementation Steps

### STEP 1: Project Setup and Configuration
**Goal**: Create the KIVO client project structure with Next.js configuration

**Actions**:
- [ ] Create `kivo-client` directory at `/home/negrito/src/projects/Museo/kivo-client`
- [ ] Copy `package.json` from Museo web and update name to "kivo-client"
- [ ] Copy configuration files: `next.config.ts`, `tsconfig.json`, `.env.local`
- [ ] Create directory structure: `src/app`, `src/components`, `src/services`, `src/store`, `src/styles`
- [ ] Run `npm install` to install dependencies
- [ ] Verify setup with `npm run dev`

**Verification**:
```bash
npm run build
npm run type-check
npm run lint
```

---

### STEP 2: Minimal Layout and Theme
**Goal**: Create the base layout without header/footer and WhatsApp-inspired theme

**Actions**:
- [ ] Create `src/app/layout.tsx` with minimal structure
- [ ] Copy SCSS structure from Museo web
- [ ] Create `src/styles/main.scss` importing Museo styles
- [ ] Override specific components for chat interface
- [ ] Set up font (Inter) and meta tags
- [ ] Create `src/app/page.tsx` with placeholder content
- [ ] Ensure mobile-first responsive design

**Files to create**:
- `src/app/layout.tsx`
- `src/app/globals.css`
- `src/styles/kivo-theme.scss`
- `src/app/page.tsx`

**Verification**:
```bash
npm run dev  # Visual check
npm run build
npm run type-check
```

---

### STEP 3: Chat Interface Foundation
**Goal**: Build the core chat interface with message bubbles

**Actions**:
- [ ] Create `MessageBubble` atom component
- [ ] Create `TypingIndicator` (reuse pattern from Museo)
- [ ] Create `ChatInterface` organism with message list
- [ ] Create `KivoStore` for conversation state
- [ ] Implement `useKivoChat` hook
- [ ] Add welcome message on load
- [ ] Style with WhatsApp-like appearance

**Files to create**:
- `src/components/atoms/MessageBubble/MessageBubble.tsx`
- `src/components/atoms/MessageBubble/index.ts`
- `src/components/organisms/ChatInterface/ChatInterface.tsx`
- `src/components/organisms/ChatInterface/index.ts`
- `src/store/KivoStore.ts`
- `src/hooks/useKivoChat.ts`

**Verification**:
```bash
npm run build
npm run type-check
npm run lint
```

---

### STEP 4: Product Display Components (UI Only)
**Goal**: Create product display components for chat interface

**Actions**:
- [ ] Create `ProductCard` molecule matching Museo design
- [ ] Create `QuickReply` atom for suggested actions
- [ ] Create `ProductList` component for multiple products
- [ ] Add mock product data for testing
- [ ] Implement "Add to Cart" UI (no backend yet)
- [ ] Style products to match current website

**Files to create**:
- `src/components/molecules/ProductCard/ProductCard.tsx`
- `src/components/atoms/QuickReply/QuickReply.tsx`
- `src/components/molecules/ProductList/ProductList.tsx`
- `src/data/mockProducts.ts`

**Verification**:
```bash
npm run build
npm run type-check
```

---

### STEP 5: Cart UI Components
**Goal**: Build cart interface components (UI only)

**Actions**:
- [ ] Create `FloatingBadge` atom for cart counter
- [ ] Create `CartItem` molecule with minimal design
- [ ] Create `CartDrawer` organism (slide-out panel)
- [ ] Add cart state to KivoStore
- [ ] Implement cart animations
- [ ] Create cart summary component

**Files to create**:
- `src/components/atoms/FloatingBadge/FloatingBadge.tsx`
- `src/components/molecules/CartItem/CartItem.tsx`
- `src/components/organisms/CartDrawer/CartDrawer.tsx`
- `src/components/molecules/CartSummary/CartSummary.tsx`

**Verification**:
```bash
npm run build
npm run type-check
# Test product search and display
```

---

### STEP 6: Authentication UI
**Goal**: Create authentication flow UI components

**Actions**:
- [ ] Create `AuthPrompt` molecule for login suggestion
- [ ] Create minimal auth pages layout
- [ ] Add auth UI state management
- [ ] Create user menu component
- [ ] Design login/logout flow in chat
- [ ] Add protected state indicators

**Files to create**:
- `src/components/molecules/AuthPrompt/AuthPrompt.tsx`
- `src/components/molecules/UserMenu/UserMenu.tsx`
- `src/app/auth/layout.tsx`

**Verification**:
```bash
npm run build
# Test Google OAuth flow
# Verify session persistence
```

---

### STEP 7: Checkout UI Flow
**Goal**: Design checkout experience within chat (UI only)

**Actions**:
- [ ] Create `CheckoutSteps` component
- [ ] Create `PaymentMethodCard` component
- [ ] Create `OrderSummary` component
- [ ] Design conversational checkout flow
- [ ] Add form components for checkout
- [ ] Create success/error states

**Files to create**:
- `src/components/molecules/CheckoutSteps/CheckoutSteps.tsx`
- `src/components/molecules/PaymentMethodCard/PaymentMethodCard.tsx`
- `src/components/molecules/OrderSummary/OrderSummary.tsx`
- `src/components/organisms/CheckoutFlow/CheckoutFlow.tsx`

**Verification**:
```bash
npm run build
npm run type-check
# Test cart operations
```

---

### STEP 8: MCP & API Integration
**Goal**: Connect UI to backend services

**Actions**:
- [ ] Create `MCPService` with real integration
- [ ] Create `KivoAssistant` service
- [ ] Connect to ProductService API
- [ ] Integrate AuthService
- [ ] Connect BasketService
- [ ] Implement OrderService
- [ ] Add error handling

**Files to create**:
- `src/services/MCPService.ts`
- `src/services/KivoAssistant.ts`
- `src/services/api/` (all API integrations)

**Verification**:
```bash
npm run build
# Test payment flow (sandbox)
# Verify order creation
```

---

### STEP 9: Payment Integration
**Goal**: Integrate payment providers

**Actions**:
- [ ] Integrate MercadoPago module
- [ ] Integrate Sara payment module
- [ ] Connect payment UI to services
- [ ] Implement payment security
- [ ] Add payment confirmation flow
- [ ] Test payment sandbox

**Files to create**:
- `src/modules/payment/` (payment integrations)

**Verification**:
```bash
npm run build
# Test payment flow (sandbox)
# Verify security measures
```

---

### STEP 10: Polish and Optimization
**Goal**: Final polish and performance optimization

**Actions**:
- [ ] Add loading skeletons
- [ ] Implement error boundaries
- [ ] Add smooth transitions
- [ ] Optimize bundle size
- [ ] Add PWA support
- [ ] Implement analytics
- [ ] Performance testing
- [ ] Accessibility audit

**Verification**:
```bash
npm run build
npm run lint
# Lighthouse audit
# Mobile device testing
```

---

## Post-Implementation Tasks

### Testing Checklist
- [ ] Test on mobile devices (iOS/Android)
- [ ] Test on desktop browsers
- [ ] Test offline functionality
- [ ] Test payment flows
- [ ] Test auth flows
- [ ] Accessibility testing

### Documentation
- [ ] Update README.md
- [ ] Create CLAUDE.md for KIVO
- [ ] Document API endpoints used
- [ ] Create deployment guide

### Deployment Preparation  
- [ ] Environment variables configured
- [ ] Docker setup if needed
- [ ] CI/CD pipeline configuration
- [ ] Production build optimization

## Success Criteria
- Fully functional chat-based commerce
- All user stories implemented
- Clean, maintainable code following Museo patterns
- Passes all build and lint checks
- Mobile-optimized performance
- Seamless payment integration

## Notes
- After each step, run verification commands
- If errors occur, fix before proceeding to next step
- Refer to Museo web for pattern examples
- Keep the UI minimal and focused on conversation
- Prioritize mobile experience