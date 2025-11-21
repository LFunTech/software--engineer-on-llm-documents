## ADDED Requirements
### Requirement: Vibe coding foundations doc
The repository SHALL include a foundations tutorial that lets a new hire set up and safely use Codex CLI, Claude Code, Cursor (or similar AI coding tools) within the first day.

#### Scenario: Foundations doc available
- **WHEN** a teammate opens `docs/vc-foundations.md`
- **THEN** they see tool overviews, install/setup steps, sign-in/account notes, and workspace access checks

#### Scenario: Safety guidance present
- **WHEN** reading the doc
- **THEN** it lists safety practices for prompts, command execution, and diff review, including examples of safe/unsafe prompts

#### Scenario: Starter prompts and exercises
- **WHEN** following the doc
- **THEN** it provides day-one exercises and starter prompts for navigation, search, and small edits using the AI tools
