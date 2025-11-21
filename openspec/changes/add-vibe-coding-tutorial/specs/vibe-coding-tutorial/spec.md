## ADDED Requirements
### Requirement: Vibe coding tutorial availability
The repository SHALL include a tutorial that introduces AI coding assistants (Codex CLI, Claude Code, Cursor, and similar tools) with setup and safety guidance, and explains how to use them alongside OpenSpec for end-to-end software delivery.

#### Scenario: Tutorial accessible in docs/vibe-coding-tutorial.md
- **WHEN** a teammate opens `docs/vibe-coding-tutorial.md`
- **THEN** they find sections covering tool overviews, setup, safeguards, and comparative notes

#### Scenario: Lifecycle guidance provided
- **WHEN** reading the tutorial
- **THEN** it explains how to pair OpenSpec with AI tools across planning/spec authoring, frontend/backend implementation, testing, and DevOps workflows using concrete prompts and instructions

#### Scenario: Role navigation
- **WHEN** a teammate opens the tutorial
- **THEN** they see clear links to role-specific guides (BA、实现、测试、运维) and tool deep-dives needed to complete the workflows

### Requirement: Practical case walkthroughs
The tutorial SHALL include at least one step-by-step example that shows prompts and resulting outputs for a sample feature implemented with OpenSpec and AI tools, including testing and deployment checks.

#### Scenario: Example traces end-to-end
- **WHEN** following the example
- **THEN** the reader sees prompts and responses that cover spec drafting, code edits (frontend and backend), test creation/execution, and operational follow-ups
