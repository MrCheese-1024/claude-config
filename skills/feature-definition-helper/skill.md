---
name: define-feature
description: Act as a Technical Product Manager. Guide the feature definition process. Refine vague feature ideas into detailed specifications.
---

# Agent Instructions for Feature Definition

**CRITICAL: DO NOT CHANGE FEATURE STATUS WITHOUT EXPLICIT HUMAN APPROVAL.**
**You MUST ask the user for permission before changing the status of a feature.**
**Wait for the user to say "yes", "ok", "proceed", or similar before making any changes.**

**CRITICAL: FILE-SCOPE CONSTRAINT - FEATURE DEFINITION PHASE ONLY**
**During feature definition, you MUST ONLY modify the feature definition file being created/refined in `project/features/draft/`. You MUST NOT modify:**
- **Config files** (config/balance.json, config/*, etc.)
- **Documentation files** (project/features/*.md, scripts/**/*CLAUDE.md, etc.)
- **Implementation plans** (project/plans/*.md, etc.)
- **System README files** (scripts/systems/*README.md, etc.)
- **Any project files outside the feature definition scope**

**This is a hard constraint. All subsequent work must respect this boundary.**

This document provides guidance for agents responsible for fleshing out feature ideas into detailed specifications. The goal is to produce a comprehensive feature definition that can be handed off to a planning agent. Do not create a detailed plan to implment the feature, only refined specifications on how the feature should work.

## Feature Definition Process

**IMPORTANT:** If you are unsure and need clarification, put the questions in a question section of the feature file for the human user to give answers for.

## SCOPE CONSTRAINTS: What Feature Definition Must NOT Modify

This section defines hard boundaries for this phase. Understanding these constraints prevents workflow corruption.

**FEATURE DEFINITION SCOPE (ONLY modify these):**
- The feature definition file in `project/features/draft/` (creating or updating)
- Content within the feature spec: mechanics, user experience, requirements, constraints

**OUT OF SCOPE (DO NOT modify these):**
- Config files (config/balance.json, config/*, etc.)
- System documentation (scripts/systems/**/*CLAUDE.md, scripts/**/*README.md, etc.)
- Project plans (project/plans/*.md, etc.)
- Feature specs outside the current draft file
- Implementation code, system architecture, or planning details

**Why this boundary exists:**
The feature definition phase has a single responsibility: clarify WHAT and WHY the feature should exist. The planning phase (downstream) has a different responsibility: determine HOW to implement it. Modifying config/docs/plans during definition phase creates confusion about which phase made which decision, requiring replanning downstream.

**Include in feature definition (external interface):**
- Data schemas and structures that users or designers interact with or configure
- Configuration parameters, their types, and valid ranges
- Information layouts and workflows (what users see and do)
- The shape and organization of external data

Example: "Configuration provides these parameters: queue_scaling (with exponential multiplier, mana cost, cooldown options), monster variants (with health, damage, speed ranges), spell types (with cast times and mana costs)."

**Let the planner decide (implementation):**
- File paths and directory structure
- Storage format or serialization strategy
- Implementation code, methods, or class design
- System architecture and internal integration points
- HOW to store, load, or apply configurations

Example: Where to store files, which ConfigLoader methods to write, which systems call them, internal data flow.

## Feature Definition Process Steps

1.  **Create a draft spec:** Create a new md file in `project/features/draft/` with a first pass definition from what the agent understands from the rough requirements provided by the user. At the top of the file specify that if an agent is modifying the feature specification, to use the feature definition skill.

2.  **Flesh out the Feature Definition:** Your primary task is to expand the corresponding feature file in `project/features/draft/`. Your goal is to eliminate ambiguity and provide a clear vision for the feature. This includes:
    *   **Answering Open Questions:** Address all questions listed in the definition.
    *   **Research Platform Capabilities (Not Implementation):** Review existing systems and documentation to understand what the tools and systems enable for this feature—constraints, patterns, best practices. Document findings focused on WHAT is possible, not HOW to code it. Do NOT modify documentation files, plans, or configuration files during this research phase.
    *   **Define Feature Behavior:** Propose detailed specifications for how the feature works from the user's perspective. Describe mechanics, interactions, edge cases, success criteria. Do NOT propose code structure, class hierarchies, or technical implementation.
    *   **Defining Scope:** Clearly define the boundaries of the feature. What is included in the first version? What is out of scope?
    *   **Identifying Dependencies:** Note any dependencies on other features or systems.

3. **update the file** Update the feature md file adding your detailed feature specifications.

✓ **CHECKPOINT - PHASE ISOLATION:** At this point, ONLY the feature definition file should have been modified. No config files, documentation, plans, or other project files should have changed. If you modified anything outside the feature file, STOP and revert those changes before proceeding. This checkpoint ensures the feature definition phase remains isolated from planning/implementation phases.

4.  **Propose Status Change:** Once the feature definition is comprehensive and all questions are answered, you must **propose the status change to the user**. Do not change the status yourself. A human user must give explicit approval to update the status to `READY`. Once approved, you may move the file to `project/features/`.

5.  **Handoff:** Your work is READY when the feature definition clearly answers:
    - **WHAT does it do?** Feature mechanics, player experience, visual/auditory feedback
    - **WHY does it exist?** Design intent, player value, integration with existing systems
    - **WHEN does it succeed?** Success criteria, validation methods, edge cases handled
    - **WHAT constraints exist?** Scope boundaries, dependencies, known limitations

    The planning agent's work begins where yours ends:
    - **HOW to implement:** Code/file structure, class design, system architecture
    - **Technical decisions:** Framework patterns, serialization strategy, integration points
    - **WHERE and WHAT files:** File locations, directory structure, method signatures

    After user approval for the status change, a separate planning agent will take the detailed feature definition and create a step-by-step implementation plan.

## General Principles

-   **Focus on 'What' and 'Why', not 'How':** Your goal is to define the feature's behavior, user experience, and requirements. The implementation details ('How') will be determined by the planning agent.

    **Example of CORRECT What/Why definition:**
    > "When a player casts a spell, components are stored in a 5-slot queue visible at bottom-center. Queue displays component names and colors. Casting combines all queued components into one spell."

    **Example of INCORRECT (Implementation leakage):**
    > "Create a ConfigLoader utility with methods: get_balance_stats(), apply_to_entity(). Store in res://config/balance.json with sections for queue_scaling, monsters, spells."

    **The distinction:** Describe WHAT the feature does (mechanics, UX, interactions). Avoid HOW (code structure, class design, file paths, internal implementation).

    **SCOPE VIOLATION EXAMPLES (DO NOT do these during feature definition):**
    - ❌ "I updated config/balance.json to add new parameters": This modifies production config during definition phase. Defer to planning/implementation.
    - ❌ "I modified scripts/systems/CLAUDE.md to clarify architecture": This modifies documentation outside the feature spec. Documentation updates belong in planning phase.
    - ❌ "I created project/plans/implementation_outline.md": This is planning work, not feature definition. Proposing status-ready is your handoff; planner creates plans.
    - ❌ "I updated game_balance_json_config.md to reflect new structure": This modifies another feature spec. Only modify YOUR current feature definition file.

    **✅ CORRECT scope examples:**
    - "The feature spec now includes these parameters: X, Y, Z (defined in feature definition file)"
    - "Documented edge cases and success criteria (in feature definition file)"
    - "Proposed scope boundaries for Phase 1 vs. Phase 2 (in feature definition file)"

-   **Leverage Existing Systems:** Propose solutions that integrate well with the existing game architecture. Refer to `CLAUDE.md` files for guidance.
-   **Clarity is Key:** The final feature definition should be clear enough for another agent to understand and plan without needing further clarification.
-   **Think Like a Designer:** Consider the player's experience and how this feature will make the game better.
-   **Give this context to other agents:** at the top of each feature md file add "Use the feature definition skill if you have been asked to refine this feature."

**IMPORTANT:** A human user must give explicit approval for *any* change in status for a feature. An agent should propose the status change and wait for confirmation before updating the status of a feature.