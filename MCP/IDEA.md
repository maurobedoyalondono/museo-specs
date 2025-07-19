Complete MCP E-commerce System Design (Core System Only)
Architecture Overview
Single Container (Port 3300+)
├── Orchestrator MCP (WebSocket server on 3300)
├── Intent MCP (stdio subprocess)
├── Ecommerce Admin MCP (stdio subprocess)
├── Ecommerce Storefront MCP (stdio subprocess)
├── Notifications MCP (stdio subprocess)
└── Redis (conversation cache)

External Clients connect to port 3300
1. Repository Structure (Core System Only)
mcp-ecommerce-system/
├── src/
│   ├── orchestrator/
│   │   ├── index.ts                    # Main server (port 3300)
│   │   ├── mcp-client.ts              # MCP stdio client wrapper
│   │   ├── conversation-cache.ts       # Redis conversation management
│   │   ├── response-formatter.ts       # Format responses by client/locale
│   │   ├── client-router.ts           # Route requests to appropriate MCPs
│   │   └── process-manager.ts         # Manage MCP subprocess lifecycle
│   ├── mcps/
│   │   ├── intent/
│   │   │   ├── index.ts               # MCP server entry point
│   │   │   ├── tools/
│   │   │   │   ├── analyze-intent.ts
│   │   │   │   ├── validate-mapping.ts
│   │   │   │   └── get-suggestions.ts
│   │   │   ├── services/
│   │   │   │   ├── pattern-matcher.ts
│   │   │   │   ├── ai-analyzer.ts
│   │   │   │   └── intent-mapper.ts
│   │   │   ├── providers/
│   │   │   │   ├── claude-provider.ts
│   │   │   │   └── groq-provider.ts
│   │   │   ├── patterns/
│   │   │   │   ├── english-patterns.ts
│   │   │   │   ├── spanish-patterns.ts
│   │   │   │   └── french-patterns.ts
│   │   │   └── types/
│   │   │       └── intent-types.ts
│   │   ├── ecommerce-admin/
│   │   │   ├── index.ts               # MCP server entry point
│   │   │   ├── tools/
│   │   │   │   ├── create-product.ts
│   │   │   │   ├── create-from-file.ts
│   │   │   │   ├── list-products.ts
│   │   │   │   ├── update-product.ts
│   │   │   │   └── delete-product.ts
│   │   │   ├── services/
│   │   │   │   ├── product-service.ts
│   │   │   │   ├── file-processor.ts
│   │   │   │   └── validation-service.ts
│   │   │   ├── parsers/
│   │   │   │   ├── csv-parser.ts
│   │   │   │   ├── excel-parser.ts
│   │   │   │   └── json-parser.ts
│   │   │   ├── api/
│   │   │   │   └── admin-api-client.ts
│   │   │   └── types/
│   │   │       └── admin-types.ts
│   │   ├── ecommerce-storefront/
│   │   │   ├── index.ts               # MCP server entry point
│   │   │   ├── tools/
│   │   │   │   ├── search-products.ts
│   │   │   │   ├── get-product-info.ts
│   │   │   │   ├── add-to-basket.ts
│   │   │   │   ├── get-basket.ts
│   │   │   │   ├── get-payment-link.ts
│   │   │   │   └── answer-question.ts
│   │   │   ├── services/
│   │   │   │   ├── product-search.ts
│   │   │   │   ├── basket-service.ts
│   │   │   │   ├── payment-service.ts
│   │   │   │   └── qa-service.ts
│   │   │   ├── api/
│   │   │   │   └── storefront-api-client.ts
│   │   │   └── types/
│   │   │       └── storefront-types.ts
│   │   └── notifications/
│   │       ├── index.ts               # MCP server entry point
│   │       ├── tools/
│   │       │   ├── send-email.ts
│   │       │   ├── send-sms.ts
│   │       │   └── send-whatsapp.ts
│   │       ├── services/
│   │       │   ├── email-service.ts
│   │       │   ├── sms-service.ts
│   │       │   └── whatsapp-service.ts
│   │       ├── templates/
│   │       │   ├── email-templates.ts
│   │       │   ├── sms-templates.ts
│   │       │   └── whatsapp-templates.ts
│   │       ├── providers/
│   │       │   ├── smtp-provider.ts
│   │       │   ├── twilio-provider.ts
│   │       │   └── meta-whatsapp-provider.ts
│   │       └── types/
│   │           └── notification-types.ts
│   ├── shared/
│   │   ├── types/
│   │   │   ├── mcp-types.ts
│   │   │   ├── client-types.ts
│   │   │   └── locale-types.ts
│   │   ├── utils/
│   │   │   ├── locale-detector.ts
│   │   │   ├── response-formatter.ts
│   │   │   ├── validation.ts
│   │   │   └── logger.ts
│   │   └── constants/
│   │       ├── locales.ts
│   │       ├── client-types.ts
│   │       └── api-endpoints.ts
│   └── config/
│       ├── mcp-servers.ts
│       ├── api-config.ts
│       ├── redis-config.ts
│       └── locale-config.ts
├── docs/
│   ├── README.md
│   ├── api-reference.md
│   ├── deployment.md
│   └── examples/
│       ├── web-client.md              # How to build web clients
│       ├── whatsapp-integration.md    # WhatsApp webhook setup
│       ├── instagram-integration.md   # Instagram webhook setup
│       ├── mobile-app.md              # Mobile app integration
│       ├── curl-examples.md           # CLI testing examples
│       └── postman-collection.json    # API testing collection
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── scripts/
│   ├── setup.sh
│   ├── deploy.sh
│   └── test-client.js                 # CLI tool for testing MCPs
├── docker-compose.yml
├── Dockerfile
├── package.json
└── .env.example
2. Port Allocation
3300: Orchestrator MCP (WebSocket/HTTP server)
6379: Redis (conversation cache)

stdio: All MCP servers (no ports needed)
3. Core MCP Components
Orchestrator (Main Process - Port 3300)
typescript// src/orchestrator/index.ts
export class MCPOrchestrator {
  private mcpClients: Map<string, MCPClient>;
  private conversationCache: ConversationCache;
  private responseFormatter: ResponseFormatter;
  
  // HTTP/WebSocket endpoints for external clients
  // stdio communication with MCP servers
}
Intent MCP Server (stdio subprocess)
typescript// src/mcps/intent/index.ts
export class IntentMCPServer {
  private patternMatcher: PatternMatcher;
  private aiAnalyzer: AIAnalyzer;
  
  // Tools: analyze_intent, validate_mapping, get_suggestions
}

// src/mcps/intent/tools/analyze-intent.ts
export async function analyzeIntent(args: {
  message: string;
  conversationContext: Message[];
  availableTools: string[];
  locale: string;
}): Promise<IntentResult>

// src/mcps/intent/services/pattern-matcher.ts
export class PatternMatcher {
  matchPatterns(message: string, locale: string): PatternMatch[]
}

// src/mcps/intent/providers/claude-provider.ts
export class ClaudeProvider implements AIProvider {
  async analyzeIntent(prompt: string): Promise<AIIntentResult>
}
Ecommerce Admin MCP Server (stdio subprocess)
typescript// src/mcps/ecommerce-admin/index.ts
export class EcommerceAdminMCPServer {
  private productService: ProductService;
  private fileProcessor: FileProcessor;
  
  // Tools: create_product, create_from_file, list_products, update_product, delete_product
}

// src/mcps/ecommerce-admin/tools/create-product.ts
export async function createProduct(args: {
  productData: ProductData;
  userId: string;
  locale: string;
}): Promise<ProductResult>

// src/mcps/ecommerce-admin/services/file-processor.ts
export class FileProcessor {
  async processFile(filePath: string, fileType: string): Promise<ProductData[]>
}

// src/mcps/ecommerce-admin/parsers/csv-parser.ts
export class CSVParser {
  parse(content: string): ProductData[]
}
Ecommerce Storefront MCP Server (stdio subprocess)
typescript// src/mcps/ecommerce-storefront/index.ts
export class EcommerceStorefrontMCPServer {
  private productSearch: ProductSearchService;
  private basketService: BasketService;
  private paymentService: PaymentService;
  
  // Tools: search_products, get_product_info, add_to_basket, get_basket, get_payment_link, answer_question
}

// src/mcps/ecommerce-storefront/tools/search-products.ts
export async function searchProducts(args: {
  query: string;
  filters: SearchFilters;
  locale: string;
}): Promise<ProductSearchResult>

// src/mcps/ecommerce-storefront/services/basket-service.ts
export class BasketService {
  async addToBasket(basketId: string, productId: string, quantity: number): Promise<BasketResult>
}
Notifications MCP Server (stdio subprocess)
typescript// src/mcps/notifications/index.ts
export class NotificationsMCPServer {
  private emailService: EmailService;
  private smsService: SMSService;
  private whatsappService: WhatsAppService;
  
  // Tools: send_email, send_sms, send_whatsapp_message
}

// src/mcps/notifications/tools/send-email.ts
export async function sendEmail(args: {
  to: string;
  subject: string;
  body: string;
  locale: string;
}): Promise<NotificationResult>

// src/mcps/notifications/providers/smtp-provider.ts
export class SMTPProvider {
  async sendEmail(config: EmailConfig): Promise<void>
}
4. Shared Infrastructure
MCP Communication
typescript// src/shared/types/mcp-types.ts
export interface MCPRequest {
  jsonrpc: '2.0';
  id: string | number;
  method: string;
  params?: any;
}

export interface MCPResponse {
  jsonrpc: '2.0';
  id: string | number;
  result?: any;
  error?: MCPError;
}

// src/orchestrator/mcp-client.ts
export class MCPClient {
  async callTool(toolName: string, args: any): Promise<any>
  private sendStdioMessage(message: MCPRequest): Promise<MCPResponse>
}
Conversation Management
typescript// src/orchestrator/conversation-cache.ts
export class ConversationCache {
  async getConversation(userId: string): Promise<Message[]>
  async addMessage(userId: string, message: Message): Promise<void>
  async clearConversation(userId: string): Promise<void>
}

// src/shared/types/client-types.ts
export interface Message {
  role: 'user' | 'assistant';
  content: string;
  timestamp: Date;
  locale?: string;
}
Locale Support
typescript// src/shared/utils/locale-detector.ts
export class LocaleDetector {
  static detectFromPhoneNumber(phoneNumber: string): string
  static detectFromBrowser(acceptLanguage: string): string
  static validateLocale(locale: string): string
}

// src/shared/constants/locales.ts
export const SUPPORTED_LOCALES = ['en', 'es', 'fr', 'de'] as const;
export type SupportedLocale = typeof SUPPORTED_LOCALES[number];
5. Data Flow Examples
Product Search Flow
1. External Client → POST http://localhost:3300/api/message
   Body: { userId, message: "Do you have iPhone?", clientType: "whatsapp", locale: "en" }

2. Orchestrator → Intent MCP (stdio):
   { method: "tools/call", params: { name: "analyze_intent", arguments: {...} } }

3. Intent MCP → Pattern Matcher → AI Provider (if needed) → Return intent

4. Orchestrator → Storefront MCP (stdio):
   { method: "tools/call", params: { name: "search_products", arguments: {...} } }

5. Storefront MCP → Your Business API:
   GET /api/products?q=iPhone&locale=en

6. Response flows back through chain, formatted for client type and locale
6. Configuration
MCP Server Configuration
typescript// src/config/mcp-servers.ts
export const MCP_SERVERS = {
  intent: {
    command: 'node',
    args: ['src/mcps/intent/index.js'],
    cwd: process.cwd()
  },
  'ecommerce-admin': {
    command: 'node', 
    args: ['src/mcps/ecommerce-admin/index.js'],
    cwd: process.cwd()
  },
  'ecommerce-storefront': {
    command: 'node',
    args: ['src/mcps/ecommerce-storefront/index.js'],
    cwd: process.cwd()
  },
  notifications: {
    command: 'node',
    args: ['src/mcps/notifications/index.js'],
    cwd: process.cwd()
  }
} as const;
API Configuration
typescript// src/config/api-config.ts
export const API_CONFIG = {
  business: {
    baseUrl: process.env.BUSINESS_API_URL || 'http://localhost:8000',
    timeout: 30000
  },
  claude: {
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-haiku-20240307'
  },
  groq: {
    apiKey: process.env.GROQ_API_KEY,
    model: 'llama3-8b-8192'
  }
};
7. Documentation Examples
docs/examples/web-client.md
markdown# Web Client Integration

## Connect to MCP Orchestrator
```javascript
const response = await fetch('http://localhost:3300/api/message', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    userId: 'user123',
    message: 'Do you have iPhone 15?',
    clientType: 'web-storefront',
    locale: 'en'
  })
});
Handle Responses
javascriptconst data = await response.json();
console.log(data.response); // "📱 Yes, we have iPhone 15 Pro for $999"

### **docs/examples/whatsapp-integration.md**
```markdown
# WhatsApp Integration

## Webhook Setup
```javascript
app.post('/webhook', async (req, res) => {
  const message = req.body.entry[0].changes[0].value.messages[0];
  
  const response = await fetch('http://localhost:3300/api/message', {
    method: 'POST',
    body: JSON.stringify({
      userId: message.from,
      message: message.text.body,
      clientType: 'whatsapp',
      locale: detectLocale(message.from)
    })
  });
  
  const result = await response.json();
  await sendWhatsAppMessage(message.from, result.response);
});

## 8. Deployment

### **Docker Setup**
```dockerfile
FROM node:18
WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY src/ ./src/
COPY docs/ ./docs/

EXPOSE 3300

CMD ["npm", "start"]
docker-compose.yml
yamlversion: '3.8'
services:
  mcp-ecommerce:
    build: .
    ports:
      - "3300:3300"
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - GROQ_API_KEY=${GROQ_API_KEY}
      - BUSINESS_API_URL=${BUSINESS_API_URL}
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
Testing Tool
javascript// scripts/test-client.js
const testMessage = async (message, clientType = 'web-storefront', locale = 'en') => {
  const response = await fetch('http://localhost:3300/api/message', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      userId: 'test-user',
      message,
      clientType,
      locale
    })
  });
  
  const result = await response.json();
  console.log(`[${clientType}] ${message} → ${result.response}`);
};

// Usage: node scripts/test-client.js
testMessage('Do you have iPhone 15?');
testMessage('Agregar iPhone al carrito', 'whatsapp', 'es');
This design focuses purely on the core MCP system with comprehensive documentation and examples showing how external developers can integrate with it, rather than providing full client implementations.