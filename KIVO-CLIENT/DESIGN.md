# KIVO CLIENT - Technical Design

## System Architecture Review

### Current Museo Web Architecture
- **Framework**: Next.js 15.3.4 with App Router
- **UI Library**: React 19.0.0
- **Language**: TypeScript 5
- **State Management**: Zustand 5.0.6
- **Styling**: SCSS with 7-1 architecture
- **Design Pattern**: Atomic Design (atoms → molecules → organisms)
- **API Communication**: Service layer with BaseAPIService
- **Authentication**: Google OAuth with JWT tokens
- **Payment**: Modular system (MercadoPago, Sara)
- **i18n**: Custom implementation with context

### KIVO Client Architecture Decisions

#### 1. Single Page Application Approach
- Main route: `/` - Contains entire shopping experience
- Supporting routes: `/auth/*` for OAuth flow only
- No traditional navigation - everything happens in chat

#### 2. Component Architecture

```
kivo-client/
├── src/
│   ├── app/
│   │   ├── layout.tsx              # Minimal layout without header/footer
│   │   ├── page.tsx                # Main chat interface page
│   │   ├── auth/
│   │   │   ├── callback/page.tsx   # Reuse existing OAuth callback
│   │   │   └── signin/page.tsx     # Minimal signin redirect
│   │   └── globals.css             # Reset and base styles
│   │
│   ├── components/
│   │   ├── atoms/
│   │   │   ├── MessageBubble/      # Chat message display
│   │   │   ├── MinimalButton/      # Simple action button
│   │   │   ├── FloatingBadge/      # Cart item counter
│   │   │   ├── QuickReply/         # Suggested response chip
│   │   │   └── TypingIndicator/    # Reuse existing
│   │   │
│   │   ├── molecules/
│   │   │   ├── ProductCard/        # Compact product in chat
│   │   │   ├── CartItem/           # Minimal cart item
│   │   │   ├── PaymentSummary/     # Payment details card
│   │   │   ├── OrderStatus/        # Order tracking card
│   │   │   └── AuthPrompt/         # Google login prompt
│   │   │
│   │   └── organisms/
│   │       ├── ChatInterface/      # Main conversation UI
│   │       ├── CartDrawer/         # Slide-out cart
│   │       └── PaymentFlow/        # Inline checkout
│   │
│   ├── services/
│   │   ├── KivoAssistant.ts        # Chat logic and responses
│   │   └── CommerceService.ts      # Simplified commerce API
│   │
│   ├── store/
│   │   └── KivoStore.ts            # Unified conversation state
│   │
│   ├── hooks/
│   │   ├── useKivoChat.ts          # Chat interactions
│   │   └── useMinimalCart.ts       # Cart management
│   │
│   └── styles/
│       └── kivo-theme.scss          # WhatsApp-inspired theme
```

#### 3. State Management Design

```typescript
// KivoStore.ts
interface KivoState {
  // Conversation
  messages: ChatMessage[]
  isTyping: boolean
  context: ConversationContext
  
  // Commerce
  cart: CartItem[]
  isCartOpen: boolean
  checkoutStep: CheckoutStep | null
  
  // User
  user: User | null
  isAuthenticated: boolean
  
  // Actions
  sendMessage: (text: string) => void
  addToCart: (product: Product) => void
  updateCartItem: (id: string, quantity: number) => void
  startCheckout: () => void
}

interface ConversationContext {
  lastViewedProducts: Product[]
  currentTopic: 'browsing' | 'cart' | 'checkout' | 'support'
  preferences: UserPreferences
}
```

#### 4. Chat Message Types

```typescript
type MessageType = 
  | 'text'           // Plain text message
  | 'product'        // Product card
  | 'productList'    // Multiple products
  | 'cart'           // Cart summary
  | 'payment'        // Payment form
  | 'orderConfirm'   // Order confirmation
  | 'quickReplies'   // Suggested actions

interface ChatMessage {
  id: string
  type: MessageType
  sender: 'user' | 'assistant'
  content: MessageContent
  timestamp: Date
  status: 'sending' | 'sent' | 'error'
}
```

#### 5. Styling Strategy

**IMPORTANT**: Styles must match the current Museo web design as closely as possible while maintaining the conversational interface.

**Reuse Museo Web Design Tokens**:
```scss
// Import existing Museo variables
@import '@/styles/abstracts/variables';
@import '@/styles/abstracts/mixins';

// Use existing Museo colors
$kivo-primary: $color-primary;         // Museo primary blue
$kivo-secondary: $color-secondary;     // Museo secondary
$kivo-background: $color-gray-50;      // Light background
$kivo-message-user: $color-primary-50; // User message (light blue)
$kivo-message-bot: $color-white;       // Assistant message
$kivo-text: $color-text-primary;       // Museo text color
$kivo-text-light: $color-text-secondary; // Secondary text

// Reuse existing spacing, typography, shadows
// Mobile-first using Museo breakpoints
```

**Mobile-First Approach**:
- Base styles for mobile (320px+)
- Tablet adjustments (768px+)
- Desktop enhancements (1024px+)
- Max content width: 600px (phone-like on desktop)

#### 6. Integration Points

**Reused from Museo Web**:
- AuthService for Google OAuth
- BaseAPIService for API communication
- UnifiedSessionStore for user/session
- Payment modules (MercadoPago/Sara)
- i18n context and translations

**New Services**:
- KivoAssistant: Handles chat logic, product search, recommendations
- CommerceService: Simplified wrapper around existing services

#### 7. User Flow Architecture

```
1. Landing
   └── Welcome message with quick actions
   
2. Product Discovery
   ├── Natural language search
   ├── Product cards in chat
   └── Add to cart actions
   
3. Cart Management
   ├── Floating badge with count
   ├── Slide-out drawer
   └── Inline quantity updates
   
4. Authentication (when needed)
   ├── Prompt in chat
   ├── Google OAuth flow
   └── Return to conversation
   
5. Checkout
   ├── Summary in chat
   ├── Payment selection
   ├── Inline payment form
   └── Confirmation message
```

#### 8. MCP Integration & API Communication Pattern

**Intent Flow Architecture**:
```
User Message → MCP Server → Intent + Context → KIVO → Direct API Call → Response UI
```

```typescript
// MCP-powered intent recognition
interface MCPIntent {
  type: 'search' | 'view_product' | 'add_to_cart' | 'view_cart' | 
        'checkout' | 'track_order' | 'support' | 'greeting';
  confidence: number;
  entities: {
    product?: string;
    category?: string;
    orderId?: string;
    quantity?: number;
  };
  context: {
    conversationHistory: string[];
    userState: 'browsing' | 'shopping' | 'checking_out';
  };
}

class KivoAssistant {
  private mcpService: MCPService;
  private apiServices: {
    product: ProductService;
    basket: BasketService;
    order: OrderService;
    auth: AuthService;
  };

  async processMessage(message: string): Promise<AssistantResponse> {
    // 1. Send to MCP for intent recognition
    const intent = await this.mcpService.analyzeIntent({
      message,
      context: this.getConversationContext(),
      userId: this.getUserId()
    });
    
    // 2. Route to appropriate API based on intent
    switch(intent.type) {
      case 'search':
        const products = await this.apiServices.product.search(intent.entities.product);
        return this.formatProductResponse(products);
        
      case 'add_to_cart':
        await this.apiServices.basket.addItem(intent.entities);
        return this.formatCartUpdateResponse();
        
      case 'checkout':
        const checkout = await this.apiServices.basket.startCheckout();
        return this.formatCheckoutResponse(checkout);
        
      // ... more intent handlers
    }
  }
}
```

**MCP Service Integration**:
```typescript
class MCPService {
  constructor(private mcpServerUrl: string, private apiKey: string) {}
  
  async analyzeIntent(request: IntentRequest): Promise<MCPIntent> {
    // Calls MCP server for NLU processing
    const response = await fetch(`${this.mcpServerUrl}/analyze`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(request)
    });
    
    return response.json();
  }
}
```

#### 9. Performance Optimizations

- Lazy load payment modules
- Virtual scrolling for long conversations
- Debounced typing indicators
- Optimistic UI updates for cart
- Service worker for offline cart
- Image optimization for product cards

#### 10. Security Considerations

- Reuse existing auth token handling
- CSRF protection for cart operations  
- XSS prevention in chat messages
- Secure payment form embedding
- Rate limiting on chat messages

## Database Schema (No changes needed)

The KIVO client will use the same API endpoints and database schema as the existing Museo web application. No database modifications are required.

## File Structure Summary

```
kivo-client/
├── package.json          # Same deps as Museo web
├── next.config.ts        # Minimal Next.js config
├── tsconfig.json         # TypeScript config
├── .env.local            # Environment variables
├── public/               # Static assets
└── src/                  # Source code (as detailed above)
```

## Key Implementation Notes

1. **No Header/Footer**: The layout.tsx will be minimal, containing only the chat interface
2. **Mobile-First**: All components designed for mobile, enhanced for desktop
3. **Conversation State**: Messages persist in session, cleared on logout
4. **Progressive Enhancement**: Basic features work without JS, enhanced with interactivity
5. **Accessibility**: Full keyboard navigation, screen reader support, ARIA labels

This design maintains compatibility with the existing Museo infrastructure while providing a completely different user experience focused on conversational commerce.