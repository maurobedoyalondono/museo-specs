# MCP E-commerce System - Technical Design

## Overview

This document outlines the technical design for the MCP (Model Context Protocol) E-commerce System that will orchestrate natural language interactions with the existing Museo platform APIs.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     External Clients                              │
│  (Web Admin, WhatsApp, Instagram, Mobile Apps, CLI)              │
└─────────────────────────────────┬───────────────────────────────┘
                                  │ WebSocket/HTTP
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│              MCP Orchestrator (Port 3300)                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ • WebSocket Server      • Request Router                │    │
│  │ • Process Manager       • Response Formatter            │    │
│  │ • Conversation Cache    • Client Type Handler           │    │
│  └─────────────────────────────────────────────────────────┘    │
└────────┬──────────┬──────────┬──────────┬──────────────────────┘
         │ stdio    │ stdio    │ stdio    │ stdio
         ▼          ▼          ▼          ▼
    ┌─────────┐┌─────────┐┌─────────┐┌─────────┐
    │ Intent  ││ Admin   ││Storefront││ Notif.  │
    │  MCP    ││  MCP    ││   MCP    ││  MCP    │
    └────┬────┘└────┬────┘└────┬────┘└────┬────┘
         │          │          │          │
         └──────────┴──────────┴──────────┘
                         │ HTTP
                         ▼
         ┌─────────────────────────────────┐
         │    Museo API (Port 8000)        │
         │  • Products  • Orders           │
         │  • Baskets   • Payments         │
         └─────────────────────────────────┘
```

## System Components

### 1. MCP Orchestrator

The main MCP server that acts as a gateway, implementing the standard MCP protocol while managing child MCP servers.

**MCP Implementation:**
- Implements standard MCP server protocol over stdio/WebSocket
- Lists all available tools from child MCP servers
- Routes tool calls to appropriate MCP servers
- Handles standard MCP methods: initialize, tools/list, tools/call
- Maintains MCP session state and capabilities

**Key Classes:**
```typescript
// src/orchestrator/index.ts
export class MCPOrchestrator {
  private transport: StdioServerTransport | WebSocketServerTransport;
  private server: Server;
  private mcpManager: MCPProcessManager;
  private conversationCache: ConversationCache;
  private responseFormatter: ResponseFormatter;
  
  // MCP Protocol Implementation
  async handleInitialize(params: InitializeParams): Promise<InitializeResult>;
  async handleToolsList(): Promise<ListToolsResult>;
  async handleToolCall(params: CallToolParams): Promise<CallToolResult>;
}

// MCP Server Configuration
const server = new Server({
  name: 'museo-mcp-orchestrator',
  version: '1.0.0',
  capabilities: {
    tools: {}
  }
});

// Register all tools from child MCPs
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: await this.aggregateToolsFromMCPs()
  };
});
```

### 2. Intent Analysis MCP

Analyzes user messages to determine intent and route to appropriate services.

**Tools:**
- `analyze_intent` - Main intent analysis
- `validate_mapping` - Validate intent mapping
- `get_suggestions` - Get intent suggestions

**Key Components:**
```typescript
// Pattern matching for common intents
export interface IntentPattern {
  pattern: RegExp;
  intent: string;
  confidence: number;
  requiredEntities?: string[];
}

// AI analysis for complex intents
export interface AIProvider {
  analyzeIntent(message: string, context: Message[]): Promise<IntentResult>;
}
```

### 3. E-commerce Admin MCP

Handles admin operations through natural language.

**Tools:**
- `create_product` - Create new products
- `create_from_file` - Import products from files
- `list_products` - List products with filters
- `update_product` - Update product information
- `delete_product` - Soft delete products

**API Integration:**
```typescript
// Calls to existing Admin API
POST /api/v1/admin/products/masters
PUT /api/v1/admin/products/masters/:id
DELETE /api/v1/admin/products/masters/:id
```

### 4. E-commerce Storefront MCP

Handles customer shopping operations.

**Tools:**
- `search_products` - Natural language product search
- `get_product_info` - Detailed product information
- `add_to_basket` - Add items to basket
- `get_basket` - View basket contents
- `get_payment_link` - Generate payment links
- `answer_question` - Product Q&A

**API Integration:**
```typescript
// Calls to existing Storefront API
GET /api/v1/catalog/products
GET /api/v1/search
POST /api/v1/basket/items
GET /api/v1/basket
```

### 5. Notifications MCP

Handles multi-channel notifications.

**Tools:**
- `send_email` - Email notifications
- `send_sms` - SMS notifications
- `send_whatsapp` - WhatsApp messages

## Data Models

### MCP Protocol Implementation
```typescript
// Standard MCP Protocol Types
interface MCPRequest {
  jsonrpc: '2.0';
  id: string | number;
  method: string;
  params?: any;
}

interface MCPResponse {
  jsonrpc: '2.0';
  id: string | number;
  result?: any;
  error?: MCPError;
}

// Tool Definition (MCP Standard)
interface Tool {
  name: string;
  description: string;
  inputSchema: {
    type: 'object';
    properties: Record<string, any>;
    required?: string[];
  };
}

// Orchestrator-specific Extensions
interface OrchestratorToolCall {
  tool: string;
  arguments: any;
  context: {
    userId: string;
    clientType: 'web-admin' | 'web-storefront' | 'whatsapp' | 'instagram' | 'mobile';
    locale: string;
    conversationId: string;
  };
}
```

### Conversation Management
```typescript
interface Conversation {
  userId: string;
  messages: Message[];
  context: ConversationContext;
  createdAt: Date;
  updatedAt: Date;
  expiresAt: Date;
}

interface Message {
  role: 'user' | 'assistant';
  content: string;
  timestamp: Date;
  intent?: IntentResult;
  locale?: string;
}
```

## Authentication Design

### 1. Client to Orchestrator
- WebSocket connections authenticated via connection params
- HTTP endpoints use Bearer tokens
- Session management for web clients

### 2. MCP to Museo API (M2M)
```typescript
interface MCPServiceAccount {
  id: string;
  name: string;
  apiKey: string;
  permissions: string[];
  createdAt: Date;
  expiresAt?: Date;
}

// Environment configuration
MCP_API_KEY=mcp_service_account_key_xxx
MCP_API_SECRET=secret_xxx
```

### 3. Authentication Flow
1. MCP servers use service account credentials
2. Obtain JWT token from Museo API auth endpoint
3. Include token in all API requests
4. Refresh token before expiration

## Database Schema

### Redis Structure
```
# Conversation Cache
conversation:{userId} -> JSON (Conversation object)
conversation:{userId}:lock -> Distributed lock

# MCP Process Status
mcp:process:{name}:status -> "running" | "stopped" | "error"
mcp:process:{name}:pid -> Process ID
mcp:process:{name}:stats -> JSON (usage stats)

# Request Cache
request:{hash} -> JSON (cached response)
```

## Communication Protocols

### 1. Client to Orchestrator (WebSocket)
```javascript
// Client connects
ws = new WebSocket('ws://localhost:3300');

// Send message (example from web storefront)
ws.send(JSON.stringify({
  type: 'message',
  userId: 'user123',
  message: 'Do you have iPhone 15?',
  clientType: 'web-storefront',
  locale: 'en'
}));

// Receive response
ws.on('message', (data) => {
  const response = JSON.parse(data);
  console.log(response.message);
});
```

### 2. Orchestrator to MCP (stdio)
```typescript
// Send request to MCP
mcpClient.send({
  jsonrpc: '2.0',
  id: 1,
  method: 'tools/call',
  params: {
    name: 'search_products',
    arguments: {
      query: 'iPhone 15',
      locale: 'en'
    }
  }
});

// Receive response
mcpClient.on('response', (response) => {
  // Process MCP response
});
```

## Error Handling

### Error Types
```typescript
enum MCPErrorCode {
  INVALID_REQUEST = -32600,
  METHOD_NOT_FOUND = -32601,
  INVALID_PARAMS = -32602,
  INTERNAL_ERROR = -32603,
  PARSE_ERROR = -32700,
  
  // Custom errors
  MCP_NOT_AVAILABLE = -32000,
  API_ERROR = -32001,
  AUTHENTICATION_ERROR = -32002,
  RATE_LIMIT_ERROR = -32003
}
```

### Error Handling Strategy
1. Graceful degradation when MCP servers are unavailable
2. Retry logic with exponential backoff
3. Circuit breaker pattern for API calls
4. User-friendly error messages per locale

## Security Considerations

1. **Input Validation**
   - Sanitize all user inputs
   - Validate message size limits
   - Rate limiting per user/IP

2. **Authentication**
   - Service accounts for M2M communication
   - JWT token rotation
   - Secure credential storage

3. **Data Privacy**
   - No sensitive data in logs
   - Conversation data encryption at rest
   - GDPR compliance for data retention

## Performance Optimizations

1. **Caching Strategy**
   - Cache intent analysis results
   - Cache API responses (with TTL)
   - Conversation context in Redis

2. **Connection Pooling**
   - Reuse HTTP connections to API
   - Redis connection pooling
   - MCP process pooling

3. **Async Processing**
   - Non-blocking I/O throughout
   - Parallel MCP tool calls when possible
   - Stream responses for large datasets

## Monitoring & Observability

### Metrics to Track
- Request latency by MCP server
- Intent recognition accuracy
- API call success rates
- WebSocket connection count
- Process memory/CPU usage

### Logging Structure
```typescript
interface LogEntry {
  timestamp: string;
  level: 'debug' | 'info' | 'warn' | 'error';
  correlationId: string;
  userId?: string;
  mcp?: string;
  message: string;
  metadata?: Record<string, any>;
}
```

## Deployment Architecture

### Container Structure
```yaml
services:
  mcp-orchestrator:
    image: museo/mcp-orchestrator:latest
    ports:
      - "3300:3300"
    environment:
      - REDIS_URL=redis://redis:6379
      - MUSEO_API_URL=http://api:8000
      - MCP_API_KEY=${MCP_API_KEY}
    depends_on:
      - redis
      - api

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

### Process Management
- Use PM2 or similar for process supervision
- Auto-restart on failure
- Graceful shutdown handling
- Health check endpoints

## Integration Points

### 1. Web Admin Updates
```typescript
// New WebSocket client in admin
const mcpClient = new MCPWebSocketClient({
  url: 'ws://localhost:3300',
  clientType: 'web-admin',
  locale: getUserLocale()
});

// Natural language input component
<NaturalLanguageInput 
  onSubmit={(message) => mcpClient.send(message)}
  onResponse={(response) => handleMCPResponse(response)}
/>
```

### 2. API Endpoints Used
- **Catalog**: `/api/v1/catalog/products`, `/api/v1/search`
- **Admin**: `/api/v1/admin/products/masters`
- **Basket**: `/api/v1/basket/items`, `/api/v1/basket`
- **Orders**: `/api/v1/orders/checkout`
- **Payments**: `/api/v1/payment/providers`

## Configuration Management

### Environment Variables
New environment variables needed in `.env`:
```bash
# MCP Configuration
MCP_ORCHESTRATOR_PORT=3300
MCP_API_KEY=mcp_service_account_key_xxx
MCP_API_SECRET=secret_xxx
MCP_JWT_SECRET=your_jwt_secret_here
MCP_REDIS_URL=redis://localhost:6379
MCP_MUSEO_API_URL=http://localhost:8000
MCP_LOG_LEVEL=info

# AI Provider Configuration  
MCP_ANTHROPIC_API_KEY=your_anthropic_key
MCP_GROQ_API_KEY=your_groq_key

# Loki Configuration (same as API)
MCP_LOKI_HOST=loki
MCP_LOKI_PORT=3100
```

### Logging Configuration
Use the exact same Winston logger configuration as the API:
```typescript
// src/shared/logging/WinstonLogger.ts
import winston from 'winston';
import LokiTransport from 'winston-loki';
// Copy exact implementation from api/src/infrastructure/logging/WinstonLogger.ts
```

## Local Development Scripts

### Start Scripts
Create platform-specific scripts to start all services:

**Linux/Mac** (`mcp/scripts/start-local.sh`):
```bash
#!/bin/bash
echo "Starting MCP Orchestrator..."
npm run dev
```

**Windows** (`mcp/scripts/start-local.bat`):
```batch
@echo off
echo Starting MCP Orchestrator...
npm run dev
```

## Docker Integration

Update `infrastructure/docker-compose.yml`:
```yaml
  # MCP Orchestrator Service
  mcp-orchestrator:
    build:
      context: ../mcp
    container_name: museo-mcp-orchestrator
    restart: unless-stopped
    ports:
      - "${MCP_ORCHESTRATOR_PORT:-3300}:3300"
    environment:
      NODE_ENV: ${NODE_ENV:-development}
      PORT: 3300
      REDIS_URL: redis://redis:6379
      MUSEO_API_URL: http://api:${API_PORT:-8000}
      MCP_API_KEY: ${MCP_API_KEY}
      MCP_API_SECRET: ${MCP_API_SECRET}
      MCP_JWT_SECRET: ${MCP_JWT_SECRET}
      ANTHROPIC_API_KEY: ${MCP_ANTHROPIC_API_KEY}
      GROQ_API_KEY: ${MCP_GROQ_API_KEY}
      LOKI_HOST: loki
      LOKI_PORT: 3100
      LOG_LEVEL: ${MCP_LOG_LEVEL:-info}
    depends_on:
      api:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ../mcp/src:/app/src
      - ${LOG_DIR}/mcp-orchestrator:/app/logs
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3300/health"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## Web Admin Integration

### 1. Create MCP Service
Create a new service following the web-admin architecture:

```typescript
// web-admin/src/shared/services/MCPService.ts
import { BaseAPIService } from './BaseAPIService'

export interface MCPMessage {
  userId: string
  message: string
  clientType: 'web-admin'
  locale: string
  metadata?: Record<string, any>
}

export interface MCPResponse {
  message: string
  data?: any
  metadata?: Record<string, any>
}

export class MCPService {
  private ws: WebSocket | null = null
  private reconnectAttempts = 0
  private maxReconnectAttempts = 5
  private reconnectDelay = 1000
  private messageHandlers: Map<string, (response: MCPResponse) => void> = new Map()
  
  constructor(private url: string = process.env.NEXT_PUBLIC_MCP_URL || 'ws://localhost:3300') {}
  
  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.ws = new WebSocket(this.url)
      
      this.ws.onopen = () => {
        console.log('Connected to MCP Orchestrator')
        this.reconnectAttempts = 0
        resolve()
      }
      
      this.ws.onmessage = (event) => {
        const response = JSON.parse(event.data) as MCPResponse
        this.handleMessage(response)
      }
      
      this.ws.onerror = (error) => {
        console.error('MCP WebSocket error:', error)
        reject(error)
      }
      
      this.ws.onclose = () => {
        console.log('Disconnected from MCP Orchestrator')
        this.attemptReconnect()
      }
    })
  }
  
  sendMessage(message: string, userId: string, locale: string): void {
    if (!this.ws || this.ws.readyState !== WebSocket.OPEN) {
      throw new Error('WebSocket is not connected')
    }
    
    const mcpMessage: MCPMessage = {
      userId,
      message,
      clientType: 'web-admin',
      locale,
      metadata: {
        timestamp: new Date().toISOString()
      }
    }
    
    this.ws.send(JSON.stringify(mcpMessage))
  }
  
  onMessage(handler: (response: MCPResponse) => void): void {
    const id = Date.now().toString()
    this.messageHandlers.set(id, handler)
  }
  
  private handleMessage(response: MCPResponse): void {
    this.messageHandlers.forEach(handler => handler(response))
  }
  
  private attemptReconnect(): void {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached')
      return
    }
    
    this.reconnectAttempts++
    setTimeout(() => {
      console.log(`Reconnection attempt ${this.reconnectAttempts}`)
      this.connect()
    }, this.reconnectDelay * this.reconnectAttempts)
  }
  
  disconnect(): void {
    if (this.ws) {
      this.ws.close()
      this.ws = null
    }
  }
}
```

### 2. Update AIChat Component
Update the existing AIChat component to use the MCP service instead of hardcoded implementation:

```typescript
// web-admin/src/shared/components/organisms/AIChat/AIChatPanel.tsx
import { useMCPService } from '@/shared/hooks/useMCPService'

// Update the component to use MCP service
const mcpService = useMCPService()

// Send messages through MCP
const handleSendMessage = async (message: string) => {
  try {
    await mcpService.sendMessage(message, userId, locale)
  } catch (error) {
    console.error('Failed to send message:', error)
  }
}
```

### 3. Add Environment Variable
```env
# web-admin/.env
NEXT_PUBLIC_MCP_URL=ws://localhost:3300
```

## Next Steps

1. Implement core orchestrator with WebSocket server
2. Create Intent MCP with pattern matching
3. Implement Storefront MCP for basic operations
4. Add Admin MCP with M2M authentication
5. Integrate with web admin interface using MCPService
6. Add comprehensive testing and documentation