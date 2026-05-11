---
name: repo-migration
description: Migrate code between two diverged Git repositories when simple git merge or cherry-pick is not feasible. Use when repositories have diverged significantly (hundreds of commits), when you need to port features from source to fork or vice versa, when repositories have incompatible histories, or when you want to selectively migrate features rather than entire branches. This skill handles commit analysis, feature grouping, migration planning, and code migration with verification. Apply this skill whenever the user mentions repository migration, diverged branches, feature porting, or syncing forked repositories.
---

# Repository Migration Skill

This skill migrates code between two diverged Git repositories when traditional methods (merge, cherry-pick, rebase) are not feasible due to significant divergence.

## Core Principles

1. **Artifacts over context**: Every step outputs files. Subsequent steps read these files, not conversation context.
2. **Human in the loop**: Key decisions require human verification before proceeding.
3. **Verifiable migrations**: Each feature migration is tested against existing or new test cases.
4. **Subagent execution**: Use subagents for heavy analytical work to keep context clean.

---

## Input Specification

The skill accepts either repository URLs or local paths:

```
source: <URL or local path>
source_branch: <branch name>
fork: <URL or local path>
fork_branch: <branch name>
direction: source->fork | fork->source
```

---

## Migration Strategy Selection

Before starting, analyze the divergence and recommend a strategy:

### Strategy A: Commit-Based (when commits are well-structured)
1. Analyze commit history
2. Group commits into features
3. Analyze migration impact per feature
4. Generate migration plan
5. Execute migration feature by feature

### Strategy B: Diff-Based (when history is messy or commits are squashed)
1. Generate diff between repositories
2. Analyze diff to identify features
3. Group changes into migration units
4. Generate migration plan
5. Execute migration feature by feature

**Decision criteria**:
- Clean commit history with meaningful messages → Strategy A
- Squashed/auto-generated commits, complex history → Strategy B
- For Go repositories: check for go.mod changes, significant API surface changes

---

## Step 1: Repository Analysis

Create `artifacts/step1_repository_analysis.json`:

```json
{
  "source": {
    "url": "<source repo URL or path>",
    "branch": "<branch>",
    "commit_count": <number>,
    "contributors": ["<list>"],
    "file_types": {"go": <count>, "md": <count>, ...},
    "key_directories": ["<list>"]
  },
  "fork": {
    "url": "<fork repo URL or path>",
    "branch": "<branch>",
    "commit_count": <number>,
    "contributors": ["<list>"],
    "file_types": {"go": <count>, "md": <count>, ...},
    "key_directories": ["<list>"]
  },
  "divergence_analysis": {
    "last_common_commit": "<hash>",
    "commits_ahead_source": <number>,
    "commits_ahead_fork": <number>,
    "strategy_recommended": "commit_based | diff_based",
    "rationale": "<explanation>"
  },
  "recommendation": "Strategy A | Strategy B"
}
```

### Analysis Steps

1. **Clone/fetch repositories** if remote URLs
2. **Analyze commit history**: `git log --oneline`, contributor stats
3. **Identify file type distribution**: `.go`, `.mod`, configuration files
4. **Find divergence point**: `git merge-base` or analyze ref logs
5. **Count diverged commits** in each direction
6. **Assess Go-specific changes**: `go.mod` diffs, API breaking changes

### Commit-Based Per-Commit Analysis

When using Strategy A (commit-based), for each diverged commit analyze and output to `artifacts/step1_commits_detail.json`:

```json
{
  "strategy": "commit_based",
  "commits_analyzed": [
    {
      "hash": "<commit hash>",
      "message": "<original commit message>",
      "author": "<author>",
      "date": "<ISO date>",
      "type": "feature | bugfix | refactor | perf | test | docs | misc",
      "problem": "<description of the problem this commit solves>",
      "code_change": {
        "summary": "<one-line summary of what changed>",
        "logic_changes": ["<list of specific logic modifications>"],
        "before_logic": "<description of logic before this commit>",
        "after_logic": "<description of logic after this commit>"
      },
      "affected_components": [
        {
          "type": "struct | function | module | package | interface",
          "name": "<fully qualified name>",
          "file_path": "<file path>",
          "change_type": "created | modified | deleted"
        }
      ],
      "depends_on_previous": true,
      "dependency_notes": "<which commits this depends on and why>",
      "migration_priority": "high | medium | low",
      "risk_level": "low | medium | high",
      "risk_reasons": ["<list of specific risk factors>"]
    }
  ],
  "commit_count": <total analyzed>,
  "type_distribution": {
    "feature": <count>,
    "bugfix": <count>,
    "refactor": <count>,
    "perf": <count>,
    "test": <count>,
    "docs": <count>,
    "misc": <count>
  },
  "high_risk_commits": ["<hashes requiring careful handling>"]
}
```

### Per-Commit Analysis Process

For each diverged commit, use subagent analysis to:

1. **Classify type**: Analyze commit message and diff to categorize
2. **Extract problem**: What issue does this commit address?
3. **Analyze code changes**: Use `git show <hash> --stat` and `git show <hash> -p` for detailed diff
4. **Identify affected components**: Parse diff for Go structs, functions, packages affected
5. **Detect dependencies**: Check if commit modifies files that previous commits also modified, indicating dependency
6. **Assess risk**: Identify potential migration difficulties (conflicts, breaking changes, etc.)

Use multiple parallel subagents to analyze commits in batches for efficiency.

---

## Step 2: Feature Grouping

Create `artifacts/step2_features.json`:

```json
{
  "strategy_used": "commit_based | diff_based",
  "features": [
    {
      "id": "feat-001",
      "name": "<feature name>",
      "description": "<what this feature does>",
      "source": "source | fork | both",
      "migration_direction": "to_fork | to_source | bidirectional",
      "priority": "high | medium | low",
      "risk_level": "low | medium | high",
      "commits": ["<commit hashes>"] | null,
      "files_affected": ["<file paths>"],
      "dependencies": ["<feature IDs>"],
      "verification_method": "test | manual | both"
    }
  ],
  "feature_count": <number>,
  "high_priority_features": ["<IDs>"],
  "risky_features": ["<IDs>"]
}
```

### Commit-Based Feature Grouping

When using commit-based strategy, reference per-commit analysis from `step1_commits_detail.json`:

1. **Analyze commit classification**: Use the `type` field to group by category
2. **Identify related file changes**: Commits that modify same packages are likely related
3. **Detect cross-cutting concerns**: Refactoring commits often affect multiple features
4. **Group by feature area**: authentication, API handlers, data models, utilities
5. **Use dependency information**: Commits with `depends_on_previous: true` must be migrated together

### Diff-Based Feature Grouping

1. **Generate diff**: `git diff <last-common-commit>...HEAD` for each repo
2. **Analyze file path patterns** (group by package/directory)
3. **Identify semantic change clusters** (related functionality)
4. **Detect refactoring vs functional changes**

---

## Step 3: Migration Impact Analysis

Create `artifacts/step3_impact_analysis.json`:

```json
{
  "features": [
    {
      "id": "feat-001",
      "migration_impact": {
        "lines_added": <number>,
        "lines_removed": <number>,
        "files_created": <number>,
        "files_modified": <number>,
        "files_deleted": <number>,
        "api_changes": {
          "breaking": ["<list>"],
          "additions": ["<list>"],
          "deprecations": ["<list>"]
        },
        "go_mod_changes": ["<module versions changed>"],
        "test_coverage": {
          "existing_tests": ["<test files>"],
          "coverage_gap": "<description>"
        }
      },
      "risks": [
        {
          "risk": "<description>",
          "severity": "high | medium | low",
          "mitigation": "<how to address>"
        }
      ],
      "estimated_effort": "hours | days | weeks"
    }
  ],
  "total_features": <number>,
  "high_risk_count": <number>,
  "estimated_total_effort": "<overall estimate>"
}
```

### Impact Analysis Steps

1. **For each feature, analyze**:
   - File-level diff statistics
   - Go module dependency changes (`go.mod` diff)
   - API signature changes (function renames, parameter changes)
   - Test file existence and coverage
2. **Identify breaking changes** that require migration adaptation
3. **Detect missing tests** that need to be created
4. **Estimate effort** based on complexity and risk

### Impact Analysis Notes

- **Breaking changes**: If a feature does NOT introduce breaking changes, use `"breaking": []` (empty array, not null). This is valid and common for new features. The presence of the `breaking` field with an empty array indicates analysis was performed.
- **API additions**: Document new API functions even when they're additions (not breaking changes). These are important for understanding what the migration introduces.
- **Risk assessment**: Always provide at least one risk entry even for low-risk features. This shows thorough analysis.

---

## Step 4: Migration Plan Generation with User Approval

**This step requires active human involvement.** Do NOT proceed to Step 5 without explicit user approval.

### Phase A: Present Analysis for Review

Before generating the migration plan, present all artifacts to the user in a readable format:

**1. Present Repository Summary**
Show `step1_repository_analysis.json` in human-readable format:
- Repository URLs and branches
- Commit counts and divergence metrics
- Recommended strategy and rationale

**2. Present Commit Analysis (if commit-based)**
Show `step1_commits_detail.json` summary:
- Type distribution (feature: X, bugfix: Y, refactor: Z, etc.)
- List of high-risk commits requiring careful handling
- Commits with dependencies that must be migrated together

**3. Present Feature Grouping**
Show `step2_features.json`:
- Feature list with names and descriptions
- Priority and risk levels per feature
- Dependencies between features

**4. Present Impact Analysis**
Show `step3_impact_analysis.json`:
- Per-feature impact summary
- Breaking changes and risks
- Effort estimates

### Phase B: Collect User Decisions

Ask the user to provide explicit decisions on each of the following:

**Decision 1: Feature Approval**
```
Review the features list above. For each feature, indicate:
- APPROVE: Include in migration
- SKIP: Do not migrate
- DEFER: Migrate later

Format: [APPROVE|SKIP|DEFER] feature-id: reason (optional)
```

**Decision 2: Migration Order**
```
Based on dependencies, what is the desired migration order?
- Use suggested order (based on dependencies)
- Custom order: [specify your preferred order]

Are there any features that MUST migrate together?
```

**Decision 3: High-Risk Commit Handling**
```
For high-risk commits identified:
- Handle automatically with best-effort (may need manual intervention)
- Skip entirely
- Defer to end of migration

For each high-risk commit: [AUTO|SKIP|DEFER] commit-hash
```

**Decision 4: Verification Approach**
```
For each approved feature, verify via:
- Existing test suites only
- Manual code review only
- Both tests and manual review
- Custom verification steps: [specify]

Default for all: [tests|manual|both]
```

**Decision 5: Conflict Resolution Strategy**
```
When conflicts occur during migration:
- Always ask me (human in loop)
- Auto-resolve if straightforward, otherwise ask
- Never auto-resolve, always ask

Default: [ask|auto-then-ask|never-auto]
```

**Decision 6: On Failure Handling**
```
If a feature migration fails:
- Roll back and stop (fail-fast)
- Roll back and continue with next feature
- Skip failed, continue with remaining features

Default: [stop|rollback-continue|skip-continue]
```

### Phase C: Record Decisions

Create `artifacts/step4_user_decisions.json`:

```json
{
  "review_timestamp": "<ISO timestamp>",
  "user_approved": true,
  "feature_decisions": {
    "feat-001": "approved",
    "feat-002": "deferred",
    "feat-003": "skipped"
  },
  "migration_order": ["<approved feature IDs in order>"],
  "high_risk_handling": {
    "<commit-hash>": "auto | skip | defer"
  },
  "verification_approach": {
    "<feature_id>": "tests | manual | both | custom"
  },
  "conflict_resolution": "ask | auto-then-ask | never-auto",
  "on_failure": "stop | rollback-continue | skip-continue",
  "user_notes": "<any additional requirements or modifications>"
}
```

**IMPORTANT**: Do NOT proceed to Step 5 until this file is created with explicit user approval.

### Phase D: Generate Migration Plan

After user approval, create `artifacts/step4_migration_plan.json`:

```json
{
  "user_approved": true,
  "approval_reference": "step4_user_decisions.json",
  "migration_order": ["<feature IDs in execution order>"],
  "features": [
    {
      "id": "feat-001",
      "execution_order": <number>,
      "user_decision": "approved",
      "actions": [
        {
          "step": 1,
          "action": "checkout target branch",
          "command": "git checkout fork-branch",
          "verify": "git status shows clean"
        },
        {
          "step": 2,
          "action": "apply feature changes",
          "command": "git cherry-pick <commits> | patch apply",
          "verify": "git diff --stat"
        },
        {
          "step": 3,
          "action": "resolve conflicts if any",
          "action_type": "manual | automated",
          "expected_conflicts": ["<files>"]
        },
        {
          "step": 4,
          "action": "run verification",
          "command": "go test ./...",
          "verify": "all tests pass"
        }
      ],
      "rollback_plan": "<how to undo if something fails>",
      "verification_commands": ["<test commands to run>"]
    }
  ],
  "conflict_resolution_mode": "<from user decisions>",
  "failure_handling": "<from user decisions>",
  "rollback_plan": "<overall rollback strategy>",
  "estimated_total_time": "<time estimate>"
}
```

```json
{
  "migration_order": ["<feature IDs in execution order>"],
  "features": [
    {
      "id": "feat-001",
      "execution_order": <number>,
      "actions": [
        {
          "step": 1,
          "action": "checkout target branch",
          "command": "git checkout fork-branch",
          "verify": "git status shows clean"
        },
        {
          "step": 2,
          "action": "apply feature changes",
          "command": "git cherry-pick <commits> | patch apply",
          "verify": "git diff --stat"
        },
        {
          "step": 3,
          "action": "resolve conflicts if any",
          "action_type": "manual | automated",
          "expected_conflicts": ["<files>"]
        },
        {
          "step": 4,
          "action": "run verification",
          "command": "go test ./...",
          "verify": "all tests pass"
        }
      ],
      "rollback_plan": "<how to undo if something fails>",
      "verification_commands": ["<test commands to run>"]
    }
  ],
  "rollback_plan": "<overall rollback strategy>",
  "estimated_total_time": "<time estimate>"
}
```

---

## Step 5: Code Migration

**Execute migrations one feature at a time**. Use subagents for isolation.

### Per-Feature Migration Workflow

1. **Create feature migration workspace**
2. **Apply changes** using appropriate method:
   - **Cherry-pick**: For clean, linear commits
   - **Patch apply**: For selected changes
   - **Manual port**: For complex refactoring
3. **Resolve conflicts** with human guidance for non-trivial conflicts
4. **Verify** using planned test commands
5. **Commit** with descriptive message linking to original commits

### Migration Methods

**Cherry-pick** (preferred for intact commit history):
```bash
git cherry-pick <commit-hash>
```

**Patch-based** (for selective changes):
```bash
git diff <source-commit> -- <path> > feature.patch
git apply feature.patch
```

**Manual port** (for complex cross-cutting changes):
- Read source implementation
- Understand intent and behavior
- Reimplement in target repository
- Add tests

### Conflict Resolution

For each conflict:
1. **Present conflict** to human
2. **Explain** what changed and why
3. **Propose resolution** options
4. **Apply selected resolution**
5. **Verify** the resolution works

---

## Step 6: Verification & Completion

Create `artifacts/step6_verification_report.json`:

```json
{
  "migration_summary": {
    "features_planned": <number>,
    "features_completed": <number>,
    "features_deferred": <number>,
    "features_skipped": <number>
  },
  "feature_results": [
    {
      "id": "feat-001",
      "status": "completed | deferred | failed",
      "verification_results": {
        "tests_passed": true,
        "manual_review": "approved | pending | issues_found",
        "test_output": "<summary>"
      },
      "conflicts_resolved": <number>,
      "files_migrated": <number>
    }
  ],
  "issues_encountered": [
    {
      "feature": "<ID>",
      "issue": "<description>",
      "resolution": "<how it was resolved>"
    }
  ],
  "recommendations": ["<any follow-up actions>"]
}
```

---

## Artifact Naming Conventions

**CRITICAL**: Always use the exact artifact filenames as specified. Do NOT use alternative naming schemes.

### Required Filename Format
All artifacts MUST use the `step<number>_<description>.json` format:
- `step1_repository_analysis.json` ✓
- `step1_commits_detail.json` ✓
- `step2_features.json` ✓
- `step3_impact_analysis.json` ✓
- `step4_user_decisions.json` ✓
- `step4_migration_plan.json` ✓
- `step6_verification_report.json` ✓

### Incorrect Filenames That Will Break Integrations
- `001_migration_plan.json` ✗
- `migration_summary.json` ✗
- `analysis.json` ✗
- `plan.json` ✗

### Why This Matters
- Downstream tools and scripts expect exact filenames
- Human reviewers know where to find each artifact
- Traceability is maintained across steps

---

## Artifact Files Reference

| Step | Output File | Purpose |
|------|-------------|---------|
| 1 | `step1_repository_analysis.json` | Repository metadata and divergence analysis |
| 1 | `step1_commits_detail.json` | Per-commit analysis (commit-based strategy only) |
| 2 | `step2_features.json` | Feature grouping with metadata |
| 3 | `step3_impact_analysis.json` | Migration impact per feature |
| 4 | `step4_user_decisions.json` | **Required** user approvals and decisions before proceeding |
| 4 | `step4_migration_plan.json` | Detailed execution plan (after user approval) |
| 5 | `step5_execution_logs/` | Per-feature execution logs |
| 6 | `step6_verification_report.json` | Final verification results |

**IMPORTANT**: Step 4 produces TWO artifacts. `step4_user_decisions.json` MUST be created and approved before `step4_migration_plan.json` is generated. Step 5 cannot begin without user approval recorded in `step4_user_decisions.json`.

---

## Execution Guidelines

1. **Always read previous step artifacts** before starting a new step
2. **Never assume context** — if an artifact is missing, regenerate it
3. **Use subagents** for isolated work (commit analysis, diff parsing, etc.)
4. **Keep human in the loop** for decisions and conflict resolution
5. **Verify at every step** before proceeding to next
6. **Document everything** in artifacts for auditability

---

## Go-Specific Considerations

- Check `go.mod` changes for module version updates
- Identify API breaking changes using `go breaking` or manual review
- Ensure migrated code maintains Go idioms and conventions
- Verify dependency compatibility between repositories
- Run `go mod tidy` after any module changes