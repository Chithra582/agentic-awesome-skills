---
name: skill-manifest-compiler
description: Validates and lints YAML frontmatter and token budgets.
---

# Skill Manifest Compiler

## Overview
Parses and validates `SKILL.md` playbooks against the OpenGAP and Agent Skills specification standards, enforcing strict kebab-case naming, description limits, and metadata hygiene.

## Key Capabilities
- Frontmatter schema validation using JSON Schema / Ajv standards.
- Token budget estimation ensuring instructions remain within optimal context sizes.
- Automatic Unix newline normalization and UTF-8 encoding verification.
