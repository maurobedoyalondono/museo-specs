# Process Designer - Technical Design

## Architecture Overview

### Technology Stack
Following kivo-client patterns:
- **Framework**: Next.js 15+ with App Router
- **Language**: TypeScript (strict mode)
- **State Management**: Zustand with persistence
- **Styling**: SCSS modules with BEM naming
- **UI Components**: Atomic Design Pattern
- **Graph Library**: React Flow (for journey visualization)
- **Drag & Drop**: React DnD
- **Icons**: @heroicons/react
- **Testing**: Jest + React Testing Library

## Data Models

### Core Entities

#### Journey
```typescript
interface Journey {
  id: string;
  name: string;
  description: string;
  version: string;
  clientType: ClientType;
  nodes: JourneyNode[];
  edges: JourneyEdge[];
  metadata: {
    createdAt: Date;
    updatedAt: Date;
    createdBy: string;
    lastModifiedBy: string;
    tags: string[];
  };
}
```

#### JourneyNode
```typescript
interface JourneyNode {
  id: string;
  type: 'intent' | 'tool' | 'response' | 'trigger';
  position: { x: number; y: number };
  data: IntentNode | ToolNode | ResponseNode | TriggerNode;
}

interface IntentNode {
  name: string;
  patterns: RegexPattern[];
  examples: string[];
  locale: string;
}

interface ToolNode {
  name: string;
  mcp?: string;
  description: string;
  parameters: Record<string, ParameterDef>;
  provides: string[];
  requires: string[];
  mandatoryContextData?: string[];
  data?: Record<string, string>;
  summary?: Record<string, string>;
}

interface ResponseNode {
  template: string;
  variables: string[];
}

interface TriggerNode {
  fromTool: string;
  toTool: string;
  condition?: string;
  parameters: Record<string, any>;
}
```

#### JourneyEdge
```typescript
interface JourneyEdge {
  id: string;
  source: string;
  target: string;
  type: 'triggers' | 'dataFlow' | 'sequence';
  label?: string;
  data?: {
    condition?: string;
    dataMapping?: Record<string, string>;
  };
}
```

#### TestExecution
```typescript
interface TestExecution {
  id: string;
  journeyId: string;
  message: string;
  clientType: string;
  locale: string;
  steps: ExecutionStep[];
  result: {
    response: string;
    metadata: Record<string, any>;
    data: Record<string, any>;
  };
}

interface ExecutionStep {
  type: 'intent' | 'tool' | 'compose';
  name: string;
  input: any;
  output: any;
  duration: number;
  status: 'success' | 'error' | 'skipped';
  error?: string;
}
```

## Component Architecture

### Atomic Design Structure
```
src/components/
├── atoms/
│   ├── NodeTypes/
│   │   ├── IntentNode/
│   │   ├── ToolNode/
│   │   └── ResponseNode/
│   ├── Button/
│   ├── Input/
│   └── Badge/
├── molecules/
│   ├── NodeEditor/
│   ├── EdgeEditor/
│   ├── Toolbar/
│   ├── PropertyPanel/
│   └── TestRunner/
├── organisms/
│   ├── JourneyCanvas/
│   ├── NodeLibrary/
│   ├── JourneyExplorer/
│   └── TestSimulator/
└── templates/
    ├── DesignerLayout/
    └── TestLayout/
```

### Key Components

#### JourneyCanvas
Main canvas for visual journey design
- React Flow integration
- Custom node/edge types
- Zoom/pan controls
- Auto-layout algorithm
- Export to image/PDF

#### NodeLibrary
Draggable elements sidebar
- Categorized nodes (Intents, Tools, Responses)
- Search/filter functionality
- Drag preview
- Template nodes

#### PropertyPanel
Context-sensitive property editor
- Node/edge properties
- Form validation
- Real-time preview
- Undo/redo support

#### TestSimulator
Interactive test environment
- Message input
- Step-by-step execution
- Data flow visualization
- Timeline view
- Export test results

## State Management

### Zustand Stores

#### JourneyStore
```typescript
interface JourneyStore {
  // State
  currentJourney: Journey | null;
  journeys: Journey[];
  selectedNode: string | null;
  selectedEdge: string | null;
  
  // Actions
  loadJourney: (id: string) => Promise<void>;
  saveJourney: () => Promise<void>;
  addNode: (node: JourneyNode) => void;
  updateNode: (id: string, data: Partial<JourneyNode>) => void;
  deleteNode: (id: string) => void;
  addEdge: (edge: JourneyEdge) => void;
  updateEdge: (id: string, data: Partial<JourneyEdge>) => void;
  deleteEdge: (id: string) => void;
}
```

#### ToolStore
```typescript
interface ToolStore {
  // State
  tools: Record<string, ToolDefinition>;
  intents: Record<string, IntentPattern[]>;
  clientTools: Record<string, ClientConfig>;
  
  // Actions
  loadTools: () => Promise<void>;
  updateTool: (name: string, tool: ToolDefinition) => void;
  updateIntent: (name: string, patterns: IntentPattern[]) => void;
  updateClientTools: (client: string, tools: string[]) => void;
}
```

#### TestStore
```typescript
interface TestStore {
  // State
  currentTest: TestExecution | null;
  testHistory: TestExecution[];
  isRunning: boolean;
  currentStep: number;
  
  // Actions
  runTest: (message: string, options: TestOptions) => Promise<void>;
  stepForward: () => void;
  stepBackward: () => void;
  clearTest: () => void;
}
```

## Services Layer

### JourneyService
```typescript
class JourneyService {
  async listJourneys(): Promise<Journey[]>;
  async getJourney(id: string): Promise<Journey>;
  async createJourney(journey: Partial<Journey>): Promise<Journey>;
  async updateJourney(id: string, updates: Partial<Journey>): Promise<Journey>;
  async deleteJourney(id: string): Promise<void>;
  async exportJourney(id: string, format: 'json' | 'pdf' | 'md'): Promise<Blob>;
  async importJourney(file: File): Promise<Journey>;
}
```

### ConfigService
```typescript
class ConfigService {
  async loadToolDefinitions(): Promise<ToolDefinitions>;
  async loadClientTools(): Promise<ClientTools>;
  async loadIntentPatterns(locale: string): Promise<IntentPatterns>;
  async saveToolDefinitions(tools: ToolDefinitions): Promise<void>;
  async saveClientTools(clients: ClientTools): Promise<void>;
  async validateConfiguration(): Promise<ValidationResult>;
}
```

### TestService
```typescript
class TestService {
  async testMessage(
    message: string,
    journey: Journey,
    options: TestOptions
  ): Promise<TestExecution>;
  
  async testWithMCP(
    message: string,
    options: TestOptions
  ): Promise<TestExecution>;
  
  async compareJourneys(
    journey1: Journey,
    journey2: Journey,
    testCases: TestCase[]
  ): Promise<ComparisonResult>;
}
```

## API Routes

### Journey Management
```
GET    /api/journeys              - List all journeys
POST   /api/journeys              - Create new journey
GET    /api/journeys/:id          - Get journey details
PUT    /api/journeys/:id          - Update journey
DELETE /api/journeys/:id          - Delete journey
POST   /api/journeys/:id/export   - Export journey
POST   /api/journeys/import       - Import journey
```

### Configuration
```
GET    /api/config/tools          - Get tool definitions
PUT    /api/config/tools          - Update tool definitions
GET    /api/config/clients        - Get client configurations
PUT    /api/config/clients        - Update client configurations
GET    /api/config/intents/:locale - Get intent patterns
PUT    /api/config/intents/:locale - Update intent patterns
POST   /api/config/validate       - Validate configuration
```

### Testing
```
POST   /api/test/run              - Run test simulation
POST   /api/test/mcp              - Test against live MCP
GET    /api/test/history          - Get test history
POST   /api/test/compare          - Compare journey outputs
```

## File Structure
```
process-designer/
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── journeys/
│   │   │   ├── config/
│   │   │   └── test/
│   │   ├── journeys/
│   │   │   ├── [id]/
│   │   │   └── page.tsx
│   │   ├── test/
│   │   └── layout.tsx
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   ├── services/
│   ├── store/
│   ├── styles/
│   ├── types/
│   └── utils/
├── public/
└── package.json
```

## Security Considerations

### Authentication & Authorization
- JWT-based authentication
- Role-based access control
- API key for service-to-service

### Data Protection
- Encrypt sensitive configuration
- Audit trail for all changes
- Backup/restore functionality

### Input Validation
- Sanitize regex patterns
- Validate tool parameters
- Prevent XSS in test messages

## Performance Optimization

### Canvas Rendering
- Virtualization for large journeys
- LOD (Level of Detail) for zoom levels
- Debounced updates
- Web Workers for layout calculation

### Data Management
- Lazy loading for journey list
- Incremental save/autosave
- Optimistic UI updates
- Client-side caching

## Development Workflow

### Local Development
```bash
npm run dev      # Start dev server
npm run test     # Run tests
npm run lint     # Lint code
npm run build    # Build for production
```

### Environment Variables
```env
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_MCP_URL=ws://localhost:3300
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
JWT_SECRET=...
```

## Future Enhancements

1. **Real-time Collaboration**
   - WebSocket-based sync
   - Conflict resolution
   - Presence indicators

2. **Analytics Integration**
   - Journey performance metrics
   - A/B test results
   - User behavior tracking

3. **AI Assistant**
   - Suggest improvements
   - Auto-generate test cases
   - Pattern detection

4. **Version Control**
   - Git integration
   - Branching/merging
   - PR previews