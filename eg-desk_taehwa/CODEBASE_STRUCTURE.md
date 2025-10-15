# EG-DESK Custom Codebase Structure

> **Auto-maintained by coding-agent**
> **Last updated**: [Never - this is the initial template]
> **CRITICAL**: Update after EVERY implementation to prevent conflicts

## Purpose

This document tracks all EG-DESK custom code (services, keybindings, commands, features) to prevent naming conflicts and duplicate implementations.

**Maintained by**: coding-agent (automatically updated during implementations)
**Used by**: coding-agent (conflict detection before implementing)

---

## 1. Services Registry

### Custom Services (EG-DESK specific)

> List all custom services/classes created for EG-DESK
> Format: `ServiceName`: file-path:line - Brief description

**Example**:
```
- `CustomThemeService`: eg-desk_taehwa/terminal/custom-theme-service.ts:23 - Custom theme management for EG-DESK
```

**Current Services**:
- (None yet - add services here as they're implemented)

### Modified Theia Services

> List Theia framework services that were extended/modified
> Format: `ServiceName`: Extended/modified at file-path:line

**Current Modifications**:
- (None yet - add modifications here)

---

## 2. Keybindings Registry

### Custom Keybindings (EG-DESK specific)

> List all keybindings bound to custom EG-DESK features
> Format: `KeyCombo`: Feature name (file-path:line)

**Example**:
```
- `Ctrl+K`: QuickSearch feature (eg-desk_taehwa/search/search-contribution.ts:45)
- `Ctrl+Shift+K`: CustomFeature (eg-desk_taehwa/keymaps/custom-keymaps.ts:123)
```

**Current Keybindings**:
- (None yet - add keybindings here as they're implemented)

### Available Keybindings

> Track which keybindings are confirmed available (checked but not used)

**Known Available** (check Theia defaults before assuming):
- `Ctrl+J`: Available (checked 2025-10-15)
- `Ctrl+Shift+J`: Available (checked 2025-10-15)
- `Ctrl+Alt+[A-Z]`: Generally available (verify before use)

**⚠️ IMPORTANT**: Always check Theia core keybindings before claiming a key is "available"

---

## 3. Commands Registry

### Custom Commands (EG-DESK specific)

> List all command IDs registered for EG-DESK features
> Format: `command.id`: Description (file-path:line)

**Example**:
```
- `egdesk.quickSearch`: Quick search functionality (eg-desk_taehwa/search/search-contribution.ts:67)
```

**Current Commands**:
- (None yet - add command IDs here as they're implemented)

---

## 4. Custom Features Timeline

> Chronological log of all custom features implemented
> **Purpose**: Track what was added when, for context and rollback

### Format
```
### YYYY-MM-DD HH:MM: Feature Name
- **Files**: [list of files created/modified]
- **Keybindings**: [if any]
- **Commands**: [if any]
- **Dependencies**: [what Theia services this relies on]
- **Description**: [brief description]
```

### Timeline

**(No features yet - entries will be added here as features are implemented)**

**Example Entry**:
```
### 2025-10-15 16:30: Quick Search Enhancement
- **Files**: search-contribution.ts (created), search-service.ts (created)
- **Keybindings**: Ctrl+K
- **Commands**: egdesk.quickSearch
- **Dependencies**: Theia SearchService (extended)
- **Description**: Custom quick search with fuzzy matching and recent history
```

---

## 5. Dependency Graph

> Map implementation dependencies: what depends on what
> **Purpose**: Understand implementation order and impact of changes

### Format
```
FeatureName
  ├─ depends on → TheiaService (Theia core)
  ├─ depends on → CustomService (EG-DESK)
  └─ registered in → module-file.ts
```

### Current Dependencies

**(No dependencies yet - graph will be built as features are implemented)**

**Example Entry**:
```
TimeBasedThemeSwitcher
  ├─ depends on → TerminalThemeService (Theia core)
  ├─ depends on → PreferenceService (Theia core)
  └─ registered in → terminal-frontend-module.ts

CustomThemeService
  └─ extends → ThemeService (Theia core)
```

---

## 6. Naming Conventions

> Track naming patterns to maintain consistency and avoid conflicts

### Used Naming Patterns (avoid conflicts)

**Service Names**:
- Services ending in "Service": (list used names)
- Services ending in "Switcher": (list used names)
- Services ending in "Manager": (list used names)

**Contribution Names**:
- Contributions ending in "Contribution": (list used names)

**Command Prefixes**:
- `egdesk.*`: Reserved for EG-DESK custom commands
- (add other prefixes as they're established)

### Available Naming Patterns

**Service Suffixes**: `*Handler`, `*Provider`, `*Controller`, `*Adapter` (available)
**Command Prefixes**: Define new prefixes as needed for feature domains

---

## 7. File Path Patterns

> Track where different types of files are located

### Current Structure

```
eg-desk_taehwa/
├─ (Feature directories will be added here as implemented)
└─ CODEBASE_STRUCTURE.md (this file)
```

### Conventions

**To be established** as features are implemented:
- Services: `eg-desk_taehwa/[feature]/[feature]-service.ts`
- Contributions: `eg-desk_taehwa/[feature]/[feature]-contribution.ts`
- Utilities: `eg-desk_taehwa/[feature]/utils/`

---

## Maintenance Instructions

### For coding-agent

**BEFORE Implementation**:
1. ✅ Read this file
2. ✅ Check for conflicts:
   - Service/class names in "Services Registry"
   - Keybindings in "Keybindings Registry"
   - Command IDs in "Commands Registry"
3. ✅ If conflict detected: **STOP**, report to Main Thread
4. ✅ If no conflicts: Proceed with implementation

**AFTER Implementation**:
1. ✅ Update relevant sections:
   - Add to Services Registry (if new service)
   - Add to Keybindings Registry (if new keybinding)
   - Add to Commands Registry (if new command)
   - Add to Timeline (always - with timestamp)
   - Update Dependency Graph (if dependencies changed)
2. ✅ Update "Last updated" timestamp at top
3. ✅ Keep formatting consistent
4. ✅ Report update in final implementation report

**When to Skip Update**:
- ⏭️ Bug fixes (no new entities)
- ⏭️ Modifying Theia framework code (packages/*)
- ⏭️ Documentation changes only
- ⏭️ Main Thread explicitly says "skip structure update"

### For Main Thread

This file is automatically maintained by coding-agent. Manual updates are rarely needed, but if you do:
- Keep format consistent
- Add timestamps to timeline entries
- Update "Last updated" at top

### For Human Developers

This file is your **conflict prevention system**. Check it before:
- Creating new services
- Adding keybindings
- Registering commands
- Naming new features

---

## Quick Reference

**Find EG-DESK codebase**: `Glob: eg-desk*/**/*.ts`
**Find this file**: `Glob: eg-desk*/CODEBASE_STRUCTURE.md`
**Search for conflicts**: Grep this file for service/keybinding/command names

---

*This file is part of the EG-DESK agent swarm system. See AGENT_SWARM_FLOW.md for workflow details.*
