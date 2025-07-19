# MCP E-commerce System - Implementation Plan

## Overview
Step-by-step plan to implement the MCP (Model Context Protocol) E-commerce System for Museo. This plan follows a phased approach, starting with the orchestrator and progressively adding MCP servers.

## Key Design Principles
- **Incremental Development**: Build and test one component at a time
- **Integration First**: Ensure each component integrates properly before moving on
- **Mock Implementation**: Start with mocks, then implement real functionality
- **Testing at Each Step**: Verify builds and functionality after each step
- **Documentation**: Update docs as we implement

## Implementation Steps

### Phase 1: MCP Orchestrator Setup

#### Step 1: Initialize MCP Project Structure ⬜
- [ ] Create `/mcp` directory structure
- [ ] Initialize npm project with TypeScript
- [ ] Create folder structure:
  ```
  mcp/
  ├── src/
  │   ├── orchestrator/
  │   ├── mcps/
  │   ├── shared/
  │   └── config/
  ├── scripts/
  ├── tests/
  └── package.json
  ```
- [ ] Install core dependencies
- [ ] Setup TypeScript configuration
- [ ] Create .env.example file
- [ ] Run `npm run build` to verify setup

#### Step 2: Implement Core MCP Orchestrator Server ⬜
- [ ] Create MCP server using @modelcontextprotocol/sdk
- [ ] Implement WebSocket transport on port 3300
- [ ] Setup standard MCP protocol handlers (initialize, tools/list, tools/call)
- [ ] Setup Winston logger (copy from API)
- [ ] Configure Loki transport
- [ ] Register orchestrator capabilities
- [ ] Run `npm run build` to verify

#### Step 3: Setup Redis Connection ⬜
- [ ] Implement Redis client configuration
- [ ] Create ConversationCache class
- [ ] Test Redis connection
- [ ] Implement conversation storage/retrieval
- [ ] Add TTL for conversations
- [ ] Run `npm run build` to verify

#### Step 4: Create MCP Process Manager ⬜
- [ ] Implement MCPProcessManager class
- [ ] Setup stdio communication with child processes
- [ ] Create MCPClient wrapper
- [ ] Implement process lifecycle methods
- [ ] Add process health monitoring
- [ ] Run `npm run build` to verify

#### Step 5: Implement Response Formatter ⬜
- [ ] Create ResponseFormatter class
- [ ] Handle different client types
- [ ] Implement locale-based formatting
- [ ] Add emoji/plain text formatting logic
- [ ] Test with mock responses
- [ ] Run `npm run build` to verify

#### Step 6: Setup Docker Configuration ⬜
- [ ] Create Dockerfile for MCP
- [ ] Update infrastructure/docker-compose.yml
- [ ] Add environment variables to .env
- [ ] Create .dockerignore file
- [ ] Document Docker configuration
- [ ] (Testing will be done later in integration phase)

#### Step 7: Create Local Development Scripts ⬜
- [ ] Create scripts/start-local.sh (Linux/Mac)
- [ ] Create scripts/start-local.bat (Windows)
- [ ] Make scripts executable
- [ ] Test local startup
- [ ] Document usage in README
- [ ] Verify all services start

### Phase 2: Intent Analysis MCP

#### Step 8: Create Intent MCP Structure ⬜
- [ ] Create src/mcps/intent/ directory
- [ ] Setup MCP server boilerplate
- [ ] Define intent types and interfaces
- [ ] Create pattern files for languages
- [ ] Setup tool definitions
- [ ] Run `npm run build` to verify

#### Step 9: Implement Pattern Matching ⬜
- [ ] Create PatternMatcher service
- [ ] Define patterns for common intents
- [ ] Implement multi-language support
- [ ] Add confidence scoring
- [ ] Test pattern matching
- [ ] Run `npm run build` to verify

#### Step 10: Add AI Provider Integration ⬜
- [ ] Create AI provider interface
- [ ] Implement mock AI provider
- [ ] Add Claude provider (using API key)
- [ ] Add Groq provider (optional)
- [ ] Test AI analysis fallback
- [ ] Run `npm run build` to verify

#### Step 11: Connect Intent MCP to Orchestrator ⬜
- [ ] Register Intent MCP in orchestrator
- [ ] Test stdio communication
- [ ] Implement analyze_intent tool
- [ ] Test end-to-end flow
- [ ] Add error handling
- [ ] Run integration tests

### Phase 3: Storefront MCP

#### Step 12: Create Storefront MCP Structure ⬜
- [ ] Create src/mcps/ecommerce-storefront/
- [ ] Setup MCP server boilerplate
- [ ] Define storefront types
- [ ] Create API client for Museo API
- [ ] Setup tool definitions
- [ ] Run `npm run build` to verify

#### Step 13: Implement Product Search ⬜
- [ ] Create search_products tool
- [ ] Integrate with /api/v1/search endpoint
- [ ] Handle natural language queries
- [ ] Format product results
- [ ] Test search functionality
- [ ] Run `npm run build` to verify

#### Step 14: Implement Basket Operations ⬜
- [ ] Create add_to_basket tool
- [ ] Create get_basket tool
- [ ] Integrate with basket API endpoints
- [ ] Handle session management
- [ ] Test basket operations
- [ ] Run `npm run build` to verify

#### Step 15: Test Storefront Integration ⬜
- [ ] Create test client script
- [ ] Test complete shopping flow
- [ ] Verify intent routing
- [ ] Test error scenarios
- [ ] Document example usage
- [ ] Run end-to-end tests

### Phase 4: Admin MCP & Authentication

#### Step 16: Setup M2M Authentication ⬜
- [ ] Design service account system
- [ ] Create API key generation
- [ ] Implement JWT service for M2M
- [ ] Add to Museo API
- [ ] Test authentication flow
- [ ] Run `npm run build` in API

#### Step 17: Create Admin MCP Structure ⬜
- [ ] Create src/mcps/ecommerce-admin/
- [ ] Setup authenticated API client
- [ ] Define admin types
- [ ] Create tool definitions
- [ ] Setup file parsers
- [ ] Run `npm run build` to verify

#### Step 18: Implement Product Management ⬜
- [ ] Create create_product tool
- [ ] Create update_product tool
- [ ] Create delete_product tool
- [ ] Add validation logic
- [ ] Test CRUD operations
- [ ] Run `npm run build` to verify

#### Step 19: Implement File Import ⬜
- [ ] Create create_from_file tool
- [ ] Implement CSV parser
- [ ] Implement Excel parser
- [ ] Implement JSON parser
- [ ] Test file imports
- [ ] Run `npm run build` to verify

### Phase 5: Web Admin Integration

#### Step 20: Create MCP Service for Web Admin ⬜
- [ ] Create MCPService.ts in web-admin
- [ ] Implement WebSocket connection
- [ ] Add reconnection logic
- [ ] Create useMCPService hook
- [ ] Test connection
- [ ] Run `npm run build` in web-admin

#### Step 21: Update AIChat Component ⬜
- [ ] Remove hardcoded implementation
- [ ] Integrate MCPService
- [ ] Update message handling
- [ ] Add error handling
- [ ] Test chat functionality
- [ ] Run `npm run build` to verify

#### Step 22: Add Environment Configuration ⬜
- [ ] Add NEXT_PUBLIC_MCP_URL to .env
- [ ] Update web-admin build config
- [ ] Test in development
- [ ] Test in Docker
- [ ] Document configuration
- [ ] Verify all connections work

### Phase 6: Testing & Documentation

#### Step 23: Create Integration Tests ⬜
- [ ] Setup test framework
- [ ] Create orchestrator tests
- [ ] Create MCP server tests
- [ ] Test complete flows
- [ ] Add performance tests
- [ ] Run full test suite

#### Step 24: Create Client Examples ⬜
- [ ] WhatsApp integration example
- [ ] Web client example
- [ ] Mobile app example
- [ ] CLI testing tool
- [ ] Postman collection
- [ ] Document all examples

#### Step 25: Final Documentation ⬜
- [ ] Update README.md
- [ ] Create API documentation
- [ ] Document deployment process
- [ ] Create troubleshooting guide
- [ ] Add architecture diagrams
- [ ] Create video demo

## Testing Checklist

After each major step:
- [ ] Run `npm run build`
- [ ] Run `npm run type-check` 
- [ ] Run `npm run lint`
- [ ] Test in local environment
- [ ] Test in Docker environment
- [ ] Verify logs in Loki

## File Structure Summary

```
mcp/
├── src/
│   ├── orchestrator/
│   │   ├── index.ts
│   │   ├── mcp-client.ts
│   │   ├── conversation-cache.ts
│   │   ├── response-formatter.ts
│   │   ├── client-router.ts
│   │   └── process-manager.ts
│   ├── mcps/
│   │   ├── intent/
│   │   ├── ecommerce-admin/
│   │   ├── ecommerce-storefront/
│   │   └── notifications/
│   ├── shared/
│   │   ├── types/
│   │   ├── utils/
│   │   ├── constants/
│   │   └── logging/
│   └── config/
├── scripts/
│   ├── start-local.sh
│   └── start-local.bat
├── tests/
├── package.json
├── tsconfig.json
├── Dockerfile
├── .env.example
└── README.md
```

## Progress Tracking
- ⬜ Not Started
- 🔄 In Progress  
- ✅ Completed

## Estimated Timeline
- Phase 1 (Orchestrator): 2-3 days
- Phase 2 (Intent MCP): 2 days
- Phase 3 (Storefront MCP): 2 days
- Phase 4 (Admin MCP): 2-3 days
- Phase 5 (Web Integration): 1-2 days
- Phase 6 (Testing & Docs): 2 days

Total: ~2 weeks for complete implementation

## Success Criteria
1. All builds pass without errors
2. Docker compose runs all services
3. Web admin can communicate with MCP
4. Natural language shopping works
5. Admin operations work via chat
6. All tests pass
7. Documentation is complete

Last Updated: 2025-07-18