# Summary: 17 — Settings System: 5+1 Layer Priority, MDM, and Hot-Reload

## Overview
Describes the Claude Code settings system — its six-layer priority stack (Plugin → User → Project → Local → Flag → Policy), MDM-based enterprise policy enforcement, and hot-reload mechanism for settings changes during a live session.

## Key Points
- **6-layer priority**: Settings are resolved in order: Plugin (lowest) → User-global (`~/.claude/settings.json`) → Project (`.claude/settings.json`) → Local (`.claude/settings.local.json`) → CLI Flag → Policy/MDM (highest, cannot be overridden).
- **MDM policy enforcement**: Organizations can deploy a managed policy layer via Mobile Device Management (MDM) or a policy file; settings at this layer are read-only and override all user/project settings.
- **Zod schema validation**: Every settings file is parsed and validated against a Zod schema on load; unknown keys are warned but not fatal; invalid values reject loudly with field-level error messages.
- **Hot-reload**: A filesystem watcher monitors all active settings files; when a file changes, the affected settings layer is re-parsed and merged, and subscribers are notified — no restart required.
- **Settings composition**: Lower-priority layers provide defaults; higher-priority layers override specific keys; array-valued settings (e.g., allowed tools) can use `+` prefix to merge rather than replace.
- **Type-safe access**: A generated TypeScript type covers all known settings keys; accessing an unknown key is a compile-time error, preventing typos in settings reads.

## Transferable Patterns
1. **Model settings as a priority-ordered merge stack**: Define each layer's scope (user / project / local / policy) explicitly and merge them in a documented order; this makes override behavior predictable and debuggable.
2. **Reserve a top-priority policy layer**: Always include a highest-priority, read-only policy layer so that enterprise/compliance requirements can be enforced without touching user or project files.
3. **Hot-reload settings with filesystem watchers**: Watch settings files for changes and re-merge in the background; users should be able to edit settings without restarting the tool.
