# AI Agent Chat - Technical Design

## Overview
A copilot-style AI assistant panel integrated into the Museo admin interface, providing task automation capabilities through a collapsible chat interface with drag-and-drop support.

## Architecture

### Component Structure

```
src/shared/components/organisms/AIChat/
├── AIChat.tsx                    # Main container component
├── AIChatPanel.tsx              # Collapsible panel wrapper
├── AIChatHeader.tsx             # Header with controls
├── AIChatMessages.tsx           # Message list container
├── AIChatMessage.tsx            # Individual message component
├── AIChatInput.tsx              # Input area with drag-drop
├── AIChatModelSelector.tsx      # Model dropdown
├── AIChatModeToggle.tsx         # Ask/Agent mode toggle
├── AIChatDropZone.tsx           # Drag-drop overlay
├── AIChatProgress.tsx           # Task progress indicator
├── AIChatExamples.tsx           # Help/examples modal
└── index.ts                     # Barrel export

src/styles/components/
└── _ai-chat.scss                # AI Chat styles following 7-1 architecture
```

### State Management

#### Store Design - Create dedicated AIChatStore using Zustand

```typescript
// src/shared/store/AIChatStore.ts
import { create } from 'zustand'

interface AIChatState {
  // UI State
  chatOpen: boolean
  chatPosition: 'right' | 'full' // right panel or full screen on mobile
  
  // Current Conversation Only (cleared on new chat)
  messages: ChatMessage[]
  currentTaskId: string | null
  isProcessing: boolean
  processingSteps: ProcessingStep[]
  
  // Settings (persisted to localStorage)
  selectedModel: AIModel
  selectedMode: 'ask' | 'agent'
  
  // Drag and Drop
  isDragging: boolean
  draggedItems: DraggedItem[]
}

interface ChatMessage {
  id: string
  type: 'user' | 'assistant'
  content: string
  attachments: Attachment[]
  modelName?: string
  timestamp: Date
  status: 'sending' | 'sent' | 'error'
}

interface Attachment {
  type: 'file' | 'folder' | 'element'
  name: string
  path?: string
  icon: string // heroicon name
  metadata?: Record<string, any>
}

interface ProcessingStep {
  id: string
  label: string // localized
  status: 'pending' | 'active' | 'completed' | 'error'
}

interface DraggedItem {
  type: 'file' | 'folder' | 'element'
  data: any
}

type AIModel = 'claude-sonnet' | 'gpt-4' | 'kimi-k2' | 'llama-3'
```

#### Actions

```typescript
interface AIChatActions {
  // UI Actions
  toggleChat: () => void
  openChat: () => void
  closeChat: () => void
  setPosition: (position: 'right' | 'full') => void
  
  // Message Actions
  sendMessage: (content: string, attachments: Attachment[]) => void
  addMessage: (message: ChatMessage) => void
  updateMessageStatus: (id: string, status: MessageStatus) => void
  clearChat: () => void
  
  // Settings Actions
  setModel: (model: AIModel) => void
  setMode: (mode: 'ask' | 'agent') => void
  
  // Task Processing
  startProcessing: (taskId: string, steps: ProcessingStep[]) => void
  updateProcessingStep: (stepId: string, status: StepStatus) => void
  completeProcessing: () => void
  
  // Drag and Drop
  setDragging: (isDragging: boolean) => void
  addDraggedItem: (item: DraggedItem) => void
  clearDraggedItems: () => void
  
  // Persistence
  loadSettings: () => void
  saveSettings: () => void
}

// Create the store
export const useAIChatStore = create<AIChatState & AIChatActions>((set, get) => ({
  // Initial state...
  // Actions implementation...
}))
```

### Store Integration with AdminStore

```typescript
// Example of AI Chat interacting with AdminStore
import { useAdminStore } from '@/shared/store/AdminStore'
import { useAIChatStore } from '@/shared/store/AIChatStore'

// In AI response handlers, we can trigger admin actions
const handleAITask = async (task: string, attachments: Attachment[]) => {
  const adminStore = useAdminStore.getState()
  const chatStore = useAIChatStore.getState()
  
  // Example: AI creates products
  if (task.includes('create products')) {
    chatStore.startProcessing('task-1', [
      { id: '1', label: 'Reading file...', status: 'active' },
      { id: '2', label: 'Analyzing schema...', status: 'pending' },
      { id: '3', label: 'Creating products...', status: 'pending' }
    ])
    
    // Mock: In real implementation, this would call AI service
    setTimeout(() => {
      // Create products via AdminStore
      const mockProducts = generateMockProducts(attachments)
      mockProducts.forEach(product => {
        adminStore.addProduct(product)
      })
      
      chatStore.completeProcessing()
      chatStore.addMessage({
        type: 'assistant',
        content: `Created ${mockProducts.length} products successfully!`,
        // ...
      })
    }, 2000)
  }
}
```

### Components Design

#### 1. AIChat (Main Container)
- Manages overall chat state
- Handles keyboard shortcuts (Escape to close)
- Provides context for child components
- Renders floating action button when collapsed

#### 2. AIChatPanel
- Sliding panel animation (transform: translateX)
- Overlay backdrop on mobile
- Responsive width (400px desktop, full screen mobile)
- Z-index management (above sidebar)
- Box shadow for depth

#### 3. AIChatHeader
- Model selector dropdown
- Mode toggle (Ask/Agent)
- Minimize/close buttons
- Title with localization

#### 4. AIChatMessages
- Virtual scrolling for performance
- Auto-scroll to latest message
- Message grouping by time
- Loading states between messages

#### 5. AIChatMessage
- User messages (right-aligned, primary color)
- Assistant messages (left-aligned, secondary color)
- Attachment pills with icons
- Markdown rendering with syntax highlighting
- Copy code button for code blocks
- Timestamp on hover

#### 6. AIChatInput
- Textarea with auto-resize
- Attachment preview area
- Send button (disabled when empty)
- Character/line limit indicators
- Placeholder text changes based on mode

#### 7. AIChatDropZone
- Full panel overlay when dragging
- Visual feedback (dashed border, background)
- Drop instructions (localized)
- File type indicators

#### 8. AIChatProgress
- Step-by-step progress display
- Animated transitions between steps
- Time estimates (mock)
- Cancel option

### Styling Design

#### SCSS Structure

```scss
// In styles/components/_ai-chat.scss

.ai-chat {
  // Floating action button
  &-trigger {
    position: fixed;
    bottom: map-get($spacing, 6);
    right: map-get($spacing, 6);
    z-index: map-get($z-index, sticky);
    // Use existing button mixin
    @include button-primary;
  }
  
  // Panel container
  &-panel {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    width: 400px;
    background: map-get(map-get($colors, admin), card-bg);
    box-shadow: map-get($shadows, xl);
    z-index: map-get($z-index, modal);
    transform: translateX(100%);
    transition: transform map-get($animation-duration, 300) ease-out;
    
    &--open {
      transform: translateX(0);
    }
    
    // Mobile full screen
    @include respond-to-max(md) {
      width: 100%;
    }
  }
  
  // Header styles
  &-header {
    @include flex-between;
    padding: map-get($spacing, 4);
    border-bottom: 1px solid map-get(map-get($colors, admin), border);
    background: map-get(map-get($colors, neutral), 50);
  }
  
  // Messages container
  &-messages {
    flex: 1;
    overflow-y: auto;
    padding: map-get($spacing, 4);
    
    // Custom scrollbar
    @include custom-scrollbar;
  }
  
  // Individual message
  &-message {
    margin-bottom: map-get($spacing, 4);
    
    &--user {
      @include flex-end;
      .ai-chat-message-content {
        background: map-get(map-get($colors, primary), 500);
        color: white;
        margin-left: map-get($spacing, 8);
      }
    }
    
    &--assistant {
      @include flex-start;
      .ai-chat-message-content {
        background: map-get(map-get($colors, neutral), 100);
        margin-right: map-get($spacing, 8);
      }
    }
    
    &-content {
      padding: map-get($spacing, 3) map-get($spacing, 4);
      border-radius: map-get($border-radius, lg);
      max-width: 80%;
      
      // Markdown styles
      @include prose-styles;
    }
  }
}
```

### Drag and Drop Implementation

```typescript
// HTML5 Drag and Drop API
interface DragDropHandlers {
  onDragEnter: (e: DragEvent) => void
  onDragOver: (e: DragEvent) => void
  onDragLeave: (e: DragEvent) => void
  onDrop: (e: DragEvent) => void
}

// File/Folder detection
const getDroppedItems = (dataTransfer: DataTransfer): Attachment[] => {
  const items: Attachment[] = []
  
  // Files
  if (dataTransfer.files.length > 0) {
    Array.from(dataTransfer.files).forEach(file => {
      items.push({
        type: 'file',
        name: file.name,
        path: file.path || file.name,
        icon: 'DocumentIcon'
      })
    })
  }
  
  // HTML Elements (custom data transfer)
  const elementData = dataTransfer.getData('text/html-element')
  if (elementData) {
    const parsed = JSON.parse(elementData)
    items.push({
      type: 'element',
      name: parsed.selector || 'HTML Element',
      icon: 'CodeBracketIcon',
      metadata: parsed
    })
  }
  
  return items
}
```

### Mock Response Generation

```typescript
// Mock response patterns based on mode and context
const generateMockResponse = (
  message: string, 
  attachments: Attachment[],
  model: AIModel,
  mode: 'ask' | 'agent'
): string => {
  const prefix = `[${model}]:`
  
  if (mode === 'ask') {
    // Brief analysis
    return `${prefix} I can help you with "${message}". ${
      attachments.length > 0 
        ? `I see you've provided ${attachments.length} item(s): ${
            attachments.map(a => a.name).join(', ')
          }. I would analyze these and provide recommendations.`
        : 'Please provide the files or data you need help with.'
    }`
  } else {
    // Detailed agent response with markdown
    return `${prefix} Processing your request: "${message}"

## Analysis
${attachments.length > 0 ? `
### Provided Items:
${attachments.map(a => `- **${a.type}**: ${a.name}`).join('\n')}

### Execution Plan:
1. Reading and analyzing provided files...
2. Checking product schema compatibility...
3. Converting data format...
4. Validating transformed data...
5. Creating product records...

### Results:
\`\`\`json
{
  "processed": ${attachments.length},
  "created": ${Math.floor(Math.random() * 10) + 1},
  "errors": 0,
  "status": "success"
}
\`\`\`

### Summary:
✅ Successfully converted and created products from your file.
` : `
I'm ready to help automate your task. Please drag and drop files or describe what you need.
`}`
  }
}
```

### Localization Keys

```typescript
// New keys to add to i18n locales
const aiChatTranslations = {
  "aiChat": {
    "title": "AI Assistant",
    "toggle": "Toggle AI Assistant",
    "close": "Close",
    "minimize": "Minimize",
    "newChat": "New Conversation",
    "clearChat": "Clear conversation?",
    "selectModel": "Select Model",
    "selectMode": "Select Mode",
    "askMode": "Ask",
    "agentMode": "Agent",
    "inputPlaceholder": {
      "ask": "Ask a question...",
      "agent": "Describe your task..."
    },
    "send": "Send",
    "dragDrop": {
      "instructions": "Drop files, folders, or page elements here",
      "dragging": "Drop to add to message"
    },
    "examples": {
      "title": "What can I help you with?",
      "askExamples": [
        "What is the product schema?",
        "How do I bulk update categories?",
        "Explain the order workflow"
      ],
      "agentExamples": [
        "Convert this CSV to products",
        "Update all products in category X",
        "Generate SKUs for new products"
      ]
    },
    "progress": {
      "processing": "Processing...",
      "analyzing": "Analyzing files...",
      "converting": "Converting data...",
      "validating": "Validating...",
      "creating": "Creating records...",
      "complete": "Task completed"
    },
    "models": {
      "claude-sonnet": "Claude Sonnet",
      "gpt-4": "GPT-4",
      "kimi-k2": "Kimi K2",
      "llama-3": "Llama 3"
    }
  }
}
```

### Integration Points

#### 1. AdminLayout Integration
```typescript
// In AdminLayout.tsx
import { AIChat } from '@/components/organisms/AIChat'

export const AdminLayout = ({ children }) => {
  return (
    <div className="admin-layout">
      {/* Existing layout */}
      <AIChat /> {/* Rendered at root level */}
    </div>
  )
}
```

#### 2. localStorage Persistence
```typescript
const STORAGE_KEYS = {
  CHAT_SETTINGS: 'museo_ai_chat_settings', // Only settings, not history
  CHAT_OPEN_STATE: 'museo_ai_chat_open'
}

// Save only settings and UI state
useEffect(() => {
  localStorage.setItem(STORAGE_KEYS.CHAT_SETTINGS, JSON.stringify({
    selectedModel,
    selectedMode
  }))
}, [selectedModel, selectedMode])

// Current conversation is kept in memory only
// Cleared when user clicks "New Conversation"
```

#### 3. Responsive Breakpoints
- Desktop (≥1024px): 400px wide right panel
- Tablet (768-1023px): 350px wide right panel  
- Mobile (<768px): Full screen overlay

### Security Considerations
- Sanitize HTML in markdown rendering
- Validate file types on drop
- Limit message length
- Rate limiting for mock responses
- XSS prevention in element metadata

### Performance Optimizations
- Lazy load chat component
- Virtual scrolling for long conversations
- Debounce input changes
- Memoize expensive renders
- Code splitting for markdown renderer

### Accessibility
- ARIA labels for all interactive elements
- Keyboard navigation support
- Screen reader announcements
- Focus management
- High contrast mode support

### Browser Compatibility
- Modern browsers with ES6+ support
- HTML5 Drag and Drop API
- CSS Grid and Flexbox
- localStorage availability check
- Graceful degradation for older browsers