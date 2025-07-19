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

#### Step 1: Initialize MCP Project Structure ✅
- [x] Create `/mcp` directory structure
- [x] Initialize npm project with TypeScript
- [x] Create folder structure:
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
- [x] Install core dependencies
- [x] Setup TypeScript configuration
- [x] Create .env.example file
- [x] Run `npm run build` to verify setup

#### Step 2: Implement Core MCP Orchestrator Server ✅
- [x] Create MCP server using @modelcontextprotocol/sdk
- [x] Implement WebSocket transport on port 3300
- [x] Setup standard MCP protocol handlers (initialize, tools/list, tools/call)
- [x] Setup Winston logger (copy from API)
- [x] Configure Loki transport
- [x] Register orchestrator capabilities
- [x] Run `npm run build` to verify

#### Step 3: Setup Redis Connection ✅
- [x] Implement Redis client configuration
- [x] Create ConversationCache class
- [x] Test Redis connection
- [x] Implement conversation storage/retrieval
- [x] Add TTL for conversations
- [x] Run `npm run build` to verify

#### Step 4: Create MCP Process Manager ✅
- [x] Implement MCPProcessManager class
- [x] Setup stdio communication with child processes
- [x] Create MCPClient wrapper
- [x] Implement process lifecycle methods
- [x] Add process health monitoring
- [x] Run `npm run build` to verify

#### Step 5: Implement Response Formatter ✅
- [x] Create ResponseFormatter class
- [x] Handle different client types
- [x] Implement locale-based formatting
- [x] Add emoji/plain text formatting logic
- [x] Test with mock responses
- [x] Run `npm run build` to verify

#### Step 6: Setup Docker Configuration ✅
- [x] Create Dockerfile for MCP
- [x] Update infrastructure/docker-compose.yml
- [x] Add environment variables to .env
- [x] Create .dockerignore file
- [x] Document Docker configuration
- [x] (Testing will be done later in integration phase)

#### Step 7: Create Local Development Scripts ✅
- [x] Create scripts/start-local.sh (Linux/Mac)
- [x] Create scripts/start-local.bat (Windows)
- [x] Make scripts executable
- [x] Test local startup
- [x] Document usage in README
- [x] Verify all services start

### Phase 2: Intent Analysis MCP

#### Step 8: Create Intent MCP Structure ✅
- [x] Create src/mcps/intent/ directory
- [x] Setup MCP server boilerplate
- [x] Define intent types and interfaces
- [x] Create pattern files for languages
- [x] Setup tool definitions
- [x] Run `npm run build` to verify

#### Step 9: Implement Pattern Matching ✅
- [x] Create PatternMatcher service
- [x] Define patterns for common intents
- [x] Implement multi-language support
- [x] Add confidence scoring
- [x] Test pattern matching
- [x] Run `npm run build` to verify

#### Step 10: Add AI Provider Integration ✅
- [x] Create AI provider interface
- [x] Implement mock AI provider
- [x] Add Claude provider (using API key)
- [x] Add OpenAI provider (using API key)
- [x] Test AI analysis fallback
- [x] Run `npm run build` to verify

#### Step 11: Connect Intent MCP to Orchestrator ✅
- [x] Register Intent MCP in orchestrator
- [x] Test stdio communication
- [x] Implement analyze_intent tool
- [x] Test end-to-end flow
- [x] Add error handling
- [x] Run integration tests

### Phase 3: Storefront MCP

#### Step 12: Create Storefront MCP Structure ✅
- [x] Create src/mcps/storefront/
- [x] Setup MCP server boilerplate
- [x] Define storefront types
- [x] Create API client for Museo API
- [x] Setup tool definitions
- [x] Run `npm run build` to verify

#### Step 13: Implement Product Search ✅
- [x] Create search_products tool
- [x] Integrate with /api/v1/search endpoint
- [x] Handle natural language queries
- [x] Format product results
- [x] Test search functionality
- [x] Run `npm run build` to verify

#### Step 14: Implement Basket Operations ✅
- [x] Create add_to_basket tool
- [x] Create get_basket tool
- [x] Integrate with basket API endpoints
- [x] Handle session management
- [x] Test basket operations
- [x] Run `npm run build` to verify

#### Step 15: Test Storefront Integration ✅
- [x] Create test client script
- [x] Test complete shopping flow
- [x] Verify intent routing
- [x] Test error scenarios
- [x] Document example usage
- [x] Run end-to-end tests

### Phase 4: Admin MCP & Authentication

#### Step 16: Setup M2M Authentication ⬜
- [ ] Design service account system
- [ ] Create API key generation
- [ ] Implement JWT service for M2M
- [ ] Add to Museo API
- [ ] Test authentication flow
- [ ] Run `npm run build` in API

#### Step 17: Create Admin MCP Structure ✅
- [x] Create src/mcps/admin/
- [x] Setup authenticated API client
- [x] Define admin types
- [x] Create tool definitions
- [x] Setup file parsers
- [x] Run `npm run build` to verify

#### Step 18: Implement Product Management ✅
- [x] Create create_product tool
- [x] Create update_product tool
- [x] Create delete_product tool
- [x] Add validation logic
- [x] Test CRUD operations
- [x] Run `npm run build` to verify

#### Step 19: Implement File Import ✅
- [x] Create import_products tool
- [x] Implement CSV parser
- [x] Implement Excel parser
- [x] Implement JSON parser
- [x] Test file imports
- [x] Run `npm run build` to verify

### Phase 5: Web Admin Integration

#### Step 20: Create MCP Service for Web Admin ✅
- [x] Create MCPService.ts in web-admin
- [x] Implement WebSocket connection
- [x] Add reconnection logic
- [x] Create response handler
- [x] Test connection
- [x] Run `npm run build` in web-admin

#### Step 21: Update AIChat Component ✅
- [x] Remove hardcoded implementation
- [x] Integrate MCPService
- [x] Update message handling
- [x] Add error handling
- [x] Test chat functionality
- [x] Run `npm run build` to verify

#### Step 22: Add Environment Configuration ✅
- [x] Add NEXT_PUBLIC_MCP_ORCHESTRATOR_URL to .env
- [x] Update web-admin build config
- [x] Test in development
- [x] Integration verified
- [x] Document configuration
- [x] Verify all connections work

### Phase 6: Testing & Documentation

#### Step 23: Create Integration Tests ⬜
- [ ] Setup test framework
- [ ] Create orchestrator tests
- [ ] Create MCP server tests
- [ ] Test complete flows
- [ ] Add performance tests
- [ ] Run full test suite

#### Step 24: Create Client Examples ✅
- [x] WhatsApp integration example
- [x] Web client integration example
- [x] Mobile app integration example
- [x] CLI testing tool (test-mcp.js)
- [x] Integration examples with documentation
- [x] Document all examples

#### Step 25: Final Documentation ✅
- [x] Update README.md
- [x] Create API documentation
- [x] Document deployment process
- [x] Create comprehensive guides
- [x] Add architecture diagrams
- [x] Integration examples completed

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