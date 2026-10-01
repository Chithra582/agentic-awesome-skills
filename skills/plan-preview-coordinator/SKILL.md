---
name: plan-preview-coordinator
description: Generates deterministic plan diffs before applying workspace changes.
---

# Plan Preview Coordinator

## Overview
Implements the Preview-Before-Apply pattern, creating structured diffs that allow human operators to review proposed additions, edits, or removals before modifying project files.

## Key Capabilities
- Unified diff generation for proposed file installations.
- Conflict detection between existing workspace skills and incoming playbooks.
- Rollback plan serialization for experimental recovery procedures.
