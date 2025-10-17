# EG-DESK Technology Stack Registry

> **Purpose**: Centralized registry of all technologies, frameworks, and libraries used in EG-DESK
> **Maintained by**: PM Agent (auto-update) + Human decisions (ideation/research phases)
> **Used by**: PM Agent for technology selection guidance during feature implementation
> **Last Updated**: 2025-10-15 (Initial template)

---

## How to Use This Document

### For PM Agent
1. **Always discover this file dynamically**: `Glob: ideas*/**/eg-desk*ideas*/*tech*.md`
2. **Read entire document** to understand available technologies
3. **Match user requirements** to technology capabilities
4. **Update this document** when new technology added (Write/Edit tools)

### For Developers
- Reference this before implementing features
- Propose new technologies during ideation/research
- Update when new framework/library integrated

---

## 1. IDE Framework

### Eclipse Theia
- **Category**: IDE Core Framework
- **Version**: (Check package.json)
- **Purpose**: Eclipse Theia IDE framework providing core IDE functionality
- **Capabilities**:
  - Terminal integration and management
  - Editor (Monaco) integration
  - Workspace and file system management
  - Command palette and keybinding system
  - Preference/settings system
  - Extension system (VS Code compatible)
  - Menu contributions
  - View containers and widgets
- **Use When**:
  - IDE 기능 구현 (editor, terminal, file system)
  - Command/keybinding 시스템
  - Workspace 관리
  - Extension 개발
- **Location**: `packages/*` (upstream Theia packages)
- **Documentation**:
  - Official: https://theia-ide.org/docs/
  - Local: See Theia source code in packages/
- **Integration Notes**: Base framework, most features integrate with Theia services

---

## 2. Desktop Application Framework

### Electron
- **Category**: Desktop Application Framework
- **Version**: (Check package.json)
- **Purpose**: Cross-platform desktop application with native OS integration
- **Capabilities**:
  - Native menus (Menu.buildFromTemplate)
  - Native dialogs (file picker, message box)
  - System tray integration
  - Notifications
  - Native file system access
  - IPC (Inter-Process Communication)
  - Window management
  - Auto-updater
- **Use When**:
  - OS native 기능 (native menus, dialogs)
  - System-level integration (tray, notifications)
  - Desktop-specific features
- **Location**: `packages/electron-*`
- **Documentation**:
  - Official: https://www.electronjs.org/docs/
  - Security: https://www.electronjs.org/docs/latest/tutorial/security
- **Integration Notes**:
  - Use with Theia for IDE features
  - Security: Enable contextIsolation, disable nodeIntegration

---

## 3. Canvas System

### Infinite Canvas
- **Category**: Canvas Rendering System
- **Status**: Research phase / To be integrated
- **Purpose**: Infinite scrollable canvas with viewport management for spatial UI
- **Capabilities**:
  - Viewport rendering (visible area optimization)
  - Camera control (zoom, pan)
  - Infinite scroll (no boundaries)
  - Coordinate transformation
  - Performance optimization (only render visible)
- **Use When**:
  - 무한 캔버스 UI
  - Spatial canvas interaction
  - Large-scale visualizations
  - Zoom/pan required
- **Location**: TBD (planned: eg-desk_taehwa/canvas/)
- **Documentation**: `ideas&external_references/infinite-canvas/`
- **Integration Notes**:
  - To be used with Konva.js for object manipulation
  - Consider performance implications

### Konva.js
- **Category**: Canvas Object Manipulation Library
- **Status**: Research phase / To be integrated
- **Purpose**: 2D canvas library for interactive shapes, drag-and-drop, transformations
- **Capabilities**:
  - Shapes (Rectangle, Circle, Path, Custom)
  - Drag-and-drop
  - Transformations (rotate, scale, translate)
  - Animations
  - Event handling (click, drag, hover)
  - Layer management
  - Hit detection
- **Use When**:
  - Canvas 객체 조작
  - 드래그앤드롭 interaction
  - Shape drawing and manipulation
  - Interactive canvas elements
- **Location**: TBD (planned: eg-desk_taehwa/canvas/)
- **Documentation**: https://konvajs.org/
- **Integration Notes**:
  - Use with Infinite Canvas for viewport
  - Performance: Use layers wisely

---

## 4. AI Integration

### Anthropic Claude API
- **Category**: AI/LLM Integration
- **Version**: (Check Theia AI packages)
- **Purpose**: AI chat and assistance functionality powered by Claude
- **Capabilities**:
  - Chat completion
  - Streaming responses
  - Context management
  - Prompt engineering
  - Vision (image understanding)
- **Use When**:
  - AI 채팅 기능
  - AI-powered assistance
  - Code generation/explanation
  - Content analysis
- **Location**: `packages/ai-anthropic/`
- **Documentation**: https://docs.anthropic.com/
- **Integration Notes**: Use Theia AI packages for integration

### Theia AI Packages
- **Category**: AI Integration Framework
- **Purpose**: Theia's built-in AI integration framework
- **Capabilities**:
  - ai-core: Core AI abstractions
  - ai-core-ui: AI UI components
  - ai-chat: Chat interface
  - ai-mcp: Model Context Protocol
- **Use When**: Integrating AI features into Theia IDE
- **Location**: `packages/ai-*/`
- **Integration Notes**: Provides standardized AI integration patterns

---

## 5. (Future Technologies)

_New frameworks and libraries will be added here during ideation/research phases_

### Template for Adding New Technology

When proposing/adding a new technology, use this format:

```markdown
### [Technology Name]
- **Category**: [Framework / Library / Service / Tool]
- **Version**: [Version if known]
- **Status**: [Research / Integration / Production]
- **Purpose**: [One-line description of what it does]
- **Capabilities**:
  - [Capability 1]
  - [Capability 2]
  - [Capability 3]
- **Use When**:
  - [Use case 1]
  - [Use case 2]
- **Location**: [Where in codebase or TBD]
- **Documentation**: [Links or local paths]
- **Integration Notes**: [Important considerations, dependencies, performance notes]
```

**Example: Adding Three.js**
```markdown
### Three.js
- **Category**: 3D Rendering Library
- **Version**: TBD
- **Status**: Proposed (2025-10-15)
- **Purpose**: 3D visualization and rendering for spatial canvas
- **Capabilities**:
  - 3D object rendering
  - Camera control
  - Lighting and materials
  - Animation
  - WebGL optimization
- **Use When**:
  - 3D visualization needs
  - Spatial representation in 3D
  - Complex geometric rendering
- **Location**: TBD (proposed: eg-desk_taehwa/3d-renderer/)
- **Documentation**: https://threejs.org/docs/
- **Integration Notes**:
  - Works with Infinite Canvas for viewport
  - Performance: WebGL context management
  - Bundle size: Consider tree-shaking
```

---

## Technology Selection Guidelines

### Decision Process (for PM Agent)

When user requests a feature:

1. **Analyze User Requirements**:
   - What is the user trying to achieve?
   - What are the key technical characteristics?
   - What existing features does it relate to?

2. **Read This Document**:
   - Discover available technologies dynamically
   - Understand capabilities of each technology
   - Check integration notes

3. **Match Requirements to Technologies**:
   - Which technology's capabilities match the requirements?
   - Are multiple technologies needed? (common!)
   - Is custom implementation needed?

4. **Make Selection**:
   - **Primary Technology**: Main technology for the feature
   - **Secondary Technologies**: Supporting technologies (if needed)
   - **Custom Implementation**: Areas requiring custom code

5. **Check Implementation Status**:
   - Does this technology exist in codebase? (Glob for it)
   - Is it in research phase? Integration phase? Production?

6. **Document Decision**:
   - Include technology selection in Strategic Guide
   - Provide rationale for choice
   - Note integration points

### Multi-Technology Features

Many features require **multiple technologies working together**:

**Examples**:
- **Canvas Node Editor**:
  - Primary: Infinite Canvas (viewport/rendering)
  - Secondary: Konva.js (node manipulation)
  - Custom: Node data model, state management

- **AI Chat UI**:
  - Primary: Theia AI packages (integration framework)
  - Secondary: Anthropic Claude API (LLM provider)
  - Custom: Chat UI components

- **Native Menu with Commands**:
  - Primary: Electron (native menu API)
  - Secondary: Theia (command system)
  - Integration: Menu items trigger Theia commands

### When No Technology Fits

If no existing technology matches requirements:

1. **Consider Custom Implementation**:
   - Pure TypeScript/React
   - Use existing tech as foundation
   - Document rationale in PRD

2. **Propose New Technology**:
   - Present to user: "This requires [NewTech]. Should we add to stack?"
   - If approved: Research phase → Integration → Update this document

3. **Never Force-Fit**:
   - Don't use wrong technology just because it exists
   - Wrong choice = technical debt
   - Better to add right technology or go custom

---

## Technology Status Tracking

### Research Phase
- Proposed, being evaluated
- Documentation review ongoing
- Proof-of-concept needed

### Integration Phase
- Approved, being integrated into codebase
- Package installation, configuration
- Example implementation

### Production
- Fully integrated and ready for use
- Examples available in codebase
- Documented patterns established

**Current Status**:
- ✅ **Production**: Theia, Electron, Theia AI packages
- 🔬 **Research**: Infinite Canvas, Konva.js
- 📋 **Proposed**: (None currently)

---

## Maintenance Log

### 2025-10-15: Initial Template
- Created technology stack registry
- Documented existing technologies: Theia, Electron, AI packages
- Added research-phase technologies: Infinite Canvas, Konva.js
- Established template for new technologies

### (Future updates will be logged here by PM Agent)

---

**Note**: This document is **dynamically referenced** by PM Agent. Never hardcode technology lists in agent prompts. Always read this file to discover current technology stack.
