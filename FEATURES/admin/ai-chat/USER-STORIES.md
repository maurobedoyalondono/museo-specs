# AI Agent Chat - User Stories

## Overview
The AI Agent Chat is a task-oriented assistant integrated into the Museo admin panel, helping administrators complete complex tasks through intelligent automation. For example, converting non-standard product files into properly formatted products, analyzing data schemas, and automating repetitive admin tasks.

## User Stories

### 1. Chat Panel Access and Visibility
**As an** admin user  
**I want to** access an AI chat panel from any page in the admin interface  
**So that** I can get AI assistance without leaving my current workflow  

**Acceptance Criteria:**
- Chat panel is accessible from all admin pages
- Chat icon/button is visible in a fixed position (bottom-right or integrated into the layout)
- Panel opens as an overlay on the right side of the screen
- Panel state (open/closed) persists across page navigation using localStorage

### 2. Chat Panel Expand/Collapse
**As an** admin user  
**I want to** expand and collapse the AI chat panel  
**So that** I can maximize my workspace when not using the chat  

**Acceptance Criteria:**
- Panel has smooth slide-in/slide-out animation
- Collapsed state shows only a chat icon or narrow bar
- Expanded state shows full chat interface (takes up right portion of screen)
- State is saved to localStorage and persists across sessions
- Panel overlays content without pushing it

### 3. Model Selection
**As an** admin user  
**I want to** select from different AI models  
**So that** I can choose the appropriate model for my needs  

**Acceptance Criteria:**
- Dropdown selector shows available models: Claude Sonnet, GPT, Kimi K2, etc.
- Selected model is displayed in the chat interface
- Model selection persists in localStorage
- Each model's responses are prefixed with the model name

### 4. Mode Selection (Ask vs Agent)
**As an** admin user  
**I want to** switch between Ask mode and Agent mode  
**So that** I can get quick answers or deeper analysis as needed  

**Acceptance Criteria:**
- Toggle or radio buttons to switch between Ask and Agent mode
- Ask mode provides brief, concise responses
- Agent mode provides detailed, comprehensive responses
- Mode selection is clearly visible in the UI
- Selected mode persists in localStorage

### 5. Text Message Input
**As an** admin user  
**I want to** describe tasks for the AI to complete  
**So that** I can automate complex administrative operations  

**Acceptance Criteria:**
- Text input field at the bottom of the chat panel
- Support for multi-line input
- Send button and Enter key submission (Shift+Enter for new line)
- Input field clears after sending
- Loading indicator while waiting for response
- Placeholder text is localized (e.g., "Describe your task...")

### 6. Drag and Drop Files
**As an** admin user  
**I want to** drag and drop files into the chat  
**So that** the AI can process and transform them (e.g., convert product specs to match schema)  

**Acceptance Criteria:**
- Drag and drop zone covers the entire chat area
- Visual feedback when dragging over the drop zone
- File icon (@heroicons/react DocumentIcon) appears in the message with file name
- Multiple files can be dropped at once
- File information (name, path) is included in the message
- Drop zone text is localized

### 7. Drag and Drop Folders
**As an** admin user  
**I want to** drag and drop folders into the chat  
**So that** the AI can batch process multiple files or understand project structure  

**Acceptance Criteria:**
- Folders show folder icon (@heroicons/react FolderIcon) with folder name
- Only folder name/path is captured (not contents)
- Visual distinction between files and folders
- Multiple folders can be dropped at once

### 8. Drag and Drop HTML Elements
**As an** admin user  
**I want to** drag HTML elements from the admin interface into the chat  
**So that** the AI can help me manipulate or extract data from specific UI components  

**Acceptance Criteria:**
- Any HTML element on the page can be dragged to the chat
- HTML element icon (@heroicons/react CodeBracketIcon) appears in the message
- Element is identified by a meaningful label (class, id, or tag name)
- Visual feedback during drag operation
- Elements maintain their context information

### 9. Chat History Display
**As an** admin user  
**I want to** see my conversation history with the AI  
**So that** I can reference previous messages and responses  

**Acceptance Criteria:**
- Messages are displayed in chronological order
- User messages are visually distinct from AI responses
- Timestamps are shown for each message
- Chat scrolls to latest message automatically
- Smooth scrolling behavior

### 10. AI Response Display
**As an** admin user  
**I want to** receive formatted AI responses  
**So that** I can easily read and understand the information  

**Acceptance Criteria:**
- AI responses show the model name as prefix
- Support for markdown formatting (headers, lists, bold, italic)
- Code blocks with syntax highlighting
- Proper spacing and typography
- Loading animation while generating response

### 11. Mock Response Generation
**As an** admin user  
**I want to** receive contextual mock responses that simulate task completion  
**So that** I can test the chat interface functionality  

**Acceptance Criteria:**
- Responses acknowledge the task and echo the user's message
- Responses list all dragged items (files, folders, elements)
- Ask mode: Brief task analysis (e.g., "I can help convert your product file to match the schema")
- Agent mode: Detailed task execution mock (e.g., "Analyzing file structure... Converting fields... Created 5 products successfully")
- Responses include the selected model name
- Mock responses demonstrate markdown formatting and code blocks
- All response text is localized

### 12. New Conversation
**As an** admin user  
**I want to** start a new conversation  
**So that** I can clear the chat history and begin fresh  

**Acceptance Criteria:**
- "New Conversation" button/option in the chat interface
- Confirmation dialog to prevent accidental clearing
- Chat history is cleared from the UI
- Scroll position resets to top
- Input field is focused for new input

### 13. Responsive Design
**As an** admin user  
**I want to** use the AI chat on different screen sizes  
**So that** I can access AI assistance on any device  

**Acceptance Criteria:**
- Chat panel adapts to tablet and mobile screens
- On mobile: panel takes full screen when expanded
- Touch-friendly interface elements
- Proper spacing for touch interactions
- Maintains functionality across all breakpoints

### 14. Visual Consistency
**As an** admin user  
**I want to** have a consistent visual experience  
**So that** the chat feels integrated with the admin interface  

**Acceptance Criteria:**
- Follows admin color scheme and typography
- Uses established SCSS component patterns
- Consistent with existing admin UI elements
- Professional appearance matching admin theme
- Smooth animations and transitions

### 15. Keyboard Navigation
**As an** admin user  
**I want to** navigate the chat using keyboard shortcuts  
**So that** I can interact efficiently  

**Acceptance Criteria:**
- Tab navigation through interactive elements
- Escape key closes expanded panel
- Arrow keys navigate through chat history
- Focus management for accessibility
- Keyboard shortcuts don't conflict with admin shortcuts

### 16. Task Examples Display
**As an** admin user  
**I want to** see example tasks the AI can perform  
**So that** I understand the agent's capabilities  

**Acceptance Criteria:**
- Help button shows example tasks
- Examples include: "Convert CSV to products", "Analyze product schema", "Batch update categories"
- Examples are localized
- Clicking an example populates the input field
- Examples vary based on selected mode (Ask vs Agent)

### 17. Task Progress Indication
**As an** admin user  
**I want to** see progress updates while the AI processes my task  
**So that** I know what the agent is doing  

**Acceptance Criteria:**
- Multi-step progress indicators for complex tasks
- Mock steps like: "Reading file...", "Analyzing schema...", "Converting data...", "Creating products..."
- Progress messages are localized
- Visual progress indicator (spinner or progress bar)
- Different progress flows for Ask mode (quick) vs Agent mode (detailed)

### 18. Localized Interface
**As an** admin user  
**I want to** use the AI chat in my preferred language  
**So that** I can work comfortably in my native language  

**Acceptance Criteria:**
- All UI text uses the i18n translation system
- Chat interface respects the admin panel's current language setting
- Translations provided for: buttons, labels, placeholders, help text, and mock responses
- New translation keys added to both en.json and es.json
- Date/time formatting follows locale conventions

## Technical Constraints
- Implementation is UI-only with hardcoded responses
- No actual AI integration in this phase
- No message persistence beyond page reload
- Focus on UX concept and visual design
- Built using Next.js, TypeScript, and SCSS
- Follows existing admin architecture patterns
- All UI text must be localized using the i18n system
- Uses @heroicons/react for all icons