# Process Designer - MCP Customer Journey Visual Editor

## Overview
Process Designer is a NextJS-based visual editor for creating and managing customer journeys in the MCP (Model Context Protocol) orchestrator system. It provides a drag-and-drop interface to visualize how intents trigger tools and other intents, allowing developers and business users to understand and modify customer interaction flows.

## Core Concept
The MCP orchestrator processes customer messages through this flow:
```
Client Message → Orchestrator → Intent Analyzer → Tools Execution → Composer → Response
```

Process Designer visualizes this flow, showing:
- How intents are detected from customer messages
- Which tools are executed for each intent
- How tools can trigger other intents (cascading effects)
- How data flows through the system
- The final response composition

## Key Features
1. **Visual Journey Editor**: Drag-and-drop interface to design customer journeys
2. **Intent Management**: Create, edit, and manage intents with their patterns
3. **Tool Management**: Define tools, their parameters, and connections
4. **Flow Visualization**: See how intents trigger tools and other intents
5. **Test Simulator**: Test messages to see intent detection and data flow
6. **Multi-Journey Support**: Create different journeys for different scenarios
7. **Real-time Preview**: See how changes affect the customer experience

## Architecture Alignment
Following the same practices as kivo-client:
- NextJS with App Router
- TypeScript with strict mode
- Atomic Design pattern for components
- Zustand for state management
- SCSS modules with BEM naming
- Mobile-first responsive design

## Data Model
Based on the MCP configuration files:
- **Tools**: Defined actions with parameters, providers, and data structures
- **Intents**: Patterns that trigger specific tools
- **Client Types**: Different channels (web, whatsapp, etc.) with available tools
- **Journeys**: Visual representations of intent-tool-response flows

## Use Cases
1. **Business Users**: Visualize and understand customer interaction flows
2. **Developers**: Design and test new intents and tools
3. **Product Managers**: Plan customer journeys and experiences
4. **QA Teams**: Test conversation flows and edge cases