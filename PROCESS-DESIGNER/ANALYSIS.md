# Process Designer - Analysis

## Current System Analysis

### MCP Orchestrator Flow
Based on the orchestrator code analysis, the system works as follows:

1. **Message Reception** (WebSocket)
   - Client sends message with userId, message, clientType, locale, metadata
   - Two types: 'message' (conversational) or 'action' (direct tool call)

2. **Intent Analysis**
   - Message goes to Intent MCP
   - Analyzes message with available tools for client type
   - Returns intents[] with type 'tool' or 'conversational'
   - Detects language and entities

3. **Tool Execution**
   - Tools executed in dependency order (search → product_info → add_to_basket)
   - Each tool can access results from previous tools via executionContext
   - Tools can have mandatoryContextData requirements
   - Tools can trigger other tools (e.g., add_to_basket → get_product_suggestions)

4. **Response Composition**
   - Conversation MCP composes final response
   - Uses tool results and conversational intents
   - Returns message text and metadata

### Key Data Structures

#### Tool Definition
```json
{
  "tool_name": {
    "mcp": "ecommerce-storefront",
    "description": "Tool description",
    "provides": ["capability1", "capability2"],
    "requires": ["dependency1"],
    "mandatoryContextData": ["basketId", "locale"],
    "triggers": [{
      "tool": "another_tool",
      "description": "Why it triggers",
      "parameters": {}
    }],
    "data": {
      "field1": "type",
      "field2": "type"
    },
    "summary": {
      "field1": "type"
    },
    "parameters": {
      "param1": "type"
    }
  }
}
```

#### Client Configuration
```json
{
  "client_type": {
    "name": "Display Name",
    "description": "Client description",
    "availableTools": ["tool1", "tool2", "tool3"]
  }
}
```

### Intent Patterns
- Stored in locale-specific files (en.ts, es.ts)
- Regular expressions to match user messages
- Examples for training/testing

### Process Designer Requirements

1. **Visual Representation**
   - Nodes: Intents, Tools, Responses
   - Edges: Triggers, Data flow
   - Swimlanes: Intent Analysis, Tool Execution, Response Composition

2. **Data Management**
   - Import/Export tool definitions
   - Manage intent patterns
   - Configure client-tool mappings
   - Define tool triggers and dependencies

3. **Testing Capabilities**
   - Input test message
   - Show intent detection
   - Visualize tool execution order
   - Display data flow between tools
   - Show final response

4. **Journey Management**
   - Default journey from existing configs
   - Create custom journeys
   - Version control for journeys
   - A/B testing scenarios

### Technical Considerations

1. **State Management**
   - Journey configuration
   - Visual graph state
   - Test execution state
   - Tool/Intent definitions

2. **Real-time Updates**
   - WebSocket connection to test against live MCP
   - Dry-run mode for testing without execution

3. **Export Capabilities**
   - Generate tool-definitions.json
   - Generate client-tools.json
   - Export intent patterns
   - Create journey documentation

### Integration Points

1. **MCP Orchestrator**
   - Read configuration files
   - Test message processing
   - Monitor live conversations (future)

2. **Development Workflow**
   - Version control integration
   - CI/CD pipeline updates
   - Configuration validation

### Security Considerations
- Read-only mode for production
- Role-based access (viewer, editor, admin)
- Audit trail for changes
- Sandbox environment for testing