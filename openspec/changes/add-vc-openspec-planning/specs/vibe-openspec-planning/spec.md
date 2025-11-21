## ADDED Requirements
### Requirement: OpenSpec planning guide
The repository SHALL include a guide that teaches how to use AI coding tools to create and refine OpenSpec proposals, designs, and spec deltas correctly.

#### Scenario: Guide available with scaffolding steps
- **WHEN** a teammate opens `docs/vc-openspec-planning.md`
- **THEN** they find commands and folder examples for scaffolding `openspec/changes/<id>/` with proposal, tasks, and spec deltas

#### Scenario: Prompt patterns documented
- **WHEN** reading the guide
- **THEN** it shows prompt recipes for proposal drafting, delta authoring with `## ADDED|MODIFIED|REMOVED Requirements`, and ambiguity checks before coding

#### Scenario: Worked example provided
- **WHEN** following the guide
- **THEN** it demonstrates a concrete idea taken through proposal, spec deltas, and validation (`openspec validate <id> --strict`), showing the prompts and commands used
