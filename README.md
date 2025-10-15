
This file provides comprehensive guidance to Claude Code and other AI agents when working with code in this repository.

---

## 🤖 AI Agent Guidance

### AI Agent Operating Principles

**Critical Instructions for AI Agents:**

1. **Tool Result Reflection**: After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding. Use your thinking to plan and iterate based on this new information, and then take the best next action.

2. **Parallel Execution**: For maximum efficiency, whenever you need to perform multiple independent operations, invoke all relevant tools simultaneously rather than sequentially.

3. **Temporary File Management**: If you create any temporary new files, scripts, or helper files for iteration, clean up these files by removing them at the end of the task.

4. **High-Quality Solutions**: Please write a high quality, general purpose solution. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Instead, implement the actual logic that solves the problem generally.

5. **Problem Understanding**: Focus on understanding the problem requirements and implementing the correct algorithm. Tests are there to verify correctness, not to define the solution. Provide a principled implementation that follows best practices and software design principles.

6. **Feasibility Assessment**: If the task is unreasonable or infeasible, or if any of the tests are incorrect, please tell me. The solution should be robust, maintainable, and extendable.


### Important Context Notes

- Each Terraform workspace corresponds to a specific client environment
- Production environments use standard GKE clusters with dedicated node pools
- Development environments use Spot VMs for cost optimization
- The workspace name must match the corresponding `.tfvars` filename in `projects/`
- Cloud SQL and GKE clusters are automatically configured for both production and development
- Development environments have automated shutdown capabilities via Cloud Functions

---

## 🧘‍♂️ Zen Principles of This Repo

### The Zen of Infrastructure Code

Inspired by [PEP 20 - The Zen of Python](https://peps.python.org/pep-0020/), we apply these principles to infrastructure code:

- **Beautiful is better than ugly** - Clean, readable Terraform over complex nested expressions
- **Explicit is better than implicit** - Clear variable names and documented intentions
- **Simple is better than complex** - Straightforward logic over clever abstractions
- **Complex is better than complicated** - When complexity is needed, make it organized not chaotic
- **Readability counts** - Code is read more often than written
- **Special cases aren't special enough to break the rules** - Consistency over exceptions
- **Errors should never pass silently** - Fail loud and early with clear messages
- **In the face of ambiguity, refuse the temptation to guess** - Test and verify, don't assume
- **If the implementation is hard to explain, it's a bad idea** - Complex patterns need clear documentation
- **If the implementation is easy to explain, it may be a good idea** - Simple solutions are often best
- **If you need a decoder ring to understand the code, rewrite it simpler** - No hieroglyphs!
- **There should be one obvious way to do it** - Establish patterns and stick to them
- **Be humble enough to build systems that are better than you** - Create safeguards that protect against human error, forgetfulness, and AI session resets

### Core Philosophical Principles

- **KISS (Keep It Simple, Stupid)** - The fundamental principle guiding ALL decisions in this repository
- Keep it simple and don't over-engineer solutions
- **No hieroglyphs - code should be readable by humans, not just compilers**
  - Avoid complex regex patterns when simple logic works
  - Replace nested function calls with clear step-by-step operations
  - Use descriptive comments for complex validation logic
  - If you need a decoder ring to understand the code, rewrite it simpler

### The "Be Humble" Principle

Create safeguards that protect against:

- Human error and oversight
- AI session resets and context loss
- Complex edge cases that might be forgotten
- Future developers who may not understand the original intent

---

## ✅ CRITICAL: Validation Architecture

**FUNDAMENTAL RULE**: Every module must have BOTH components for validations to work:

### Mandatory Pattern

**Two components required:**

1. **Validation Resource** - `terraform_data` in `validations.tf` with lifecycle preconditions
2. **Dependency Enforcement** - Main resources with `depends_on = [terraform_data.{module}_business_validations]`

**Without BOTH pieces, validations are DEAD CODE.**

*See [code-examples.md](code-examples.md#✅-validation-architecture-examples) for complete code patterns*

### Key Learning

**Infrastructure code requires ACTIVE validation enforcement, not just validation existence.**

- Validations must be INTEGRATED into resource creation flow
- Having validation logic that never executes is worse than no validation
- Validations must block resource creation on constraint violations

### Module Completion Requirements

Before marking ANY module as complete, verify:

#### Infrastructure Safety Checklist

- [ ] **Validation Existence**: `terraform_data` resource exists in validations.tf
- [ ] **Validation Enforcement**: Main resources have `depends_on = [terraform_data.validations]`
- [ ] **Validation Activation**: Run audit commands to confirm validations execute
- [ ] **No Dead Code**: No validation logic exists without enforcement mechanism
- [ ] **Dependency Matrix**: Module appears in validation dependency matrix

#### Validation Architecture Verification

### MANDATORY: Run before marking any module complete

Use the current Terraform workspace to pick the matching tfvars file:

- `terraform plan -input=false -var-file=projects/$(terraform workspace show).tfvars`
- Verify validation preconditions are evaluated
- Confirm no validation-related errors with `terraform validate`
- Ensure `projects/<workspace>.tfvars` exists for the selected workspace

---

## 🎯 PR Quality Guide - Preventing Review Comments

**Goal**: Write infrastructure code that passes review on first submission

**Target**: <5 review comments per PR (down from 40-comment baseline)

### Critical Discovery

80% of review comments come from just 7 recurring patterns:

- 30% Null safety violations
- 25% Poor error messages
- 15% Architecture inconsistencies
- 15% Validation placement issues
- 10% DRY violations
- 10% Incomplete defensive programming
- 15% Optional field safety issues

### ✅ DO's (Prevents 80% of Review Comments)

1. **Null Safety (30% of issues)**
   - Always use `coalesce(var.x, default)`
   - Use `try()` for object access
   - Use `lookup()` for nested values

2. **Error Messages (25% of issues)**
   - Template: `"${what} (${current}) must be ${constraint} for ${why}; set ${var} = '${example}' in .tfvars"`
   - Include current values
   - Provide actionable fixes

3. **Validation Location (15% of issues)**
   - ALL validations in `validations.tf` (NOT in `variables.tf`)
   - Use `terraform_data` with `depends_on` enforcement
   - Exception: `modules/metadata/variables.tf` and `modules/buckets/variables.tf` (documented exceptions)

4. **Input Normalization**
   - Pattern: `trimspace(coalesce(var.x, ""))`
   - Normalize once in locals, reuse everywhere
   - Apply before regex/comparisons

5. **Security Defaults**
   - Mark PII outputs: `sensitive = true`
   - Wrap external APIs: `try(data.external_api.field, null)`

6. **DRY Principle**
   - Single source of truth in locals
   - Reference `local.*_validation.condition` in preconditions
   - No duplicate regex patterns

### ❌ DON'Ts (Common Anti-Patterns)

1. **No Complex Regex**
   - Cloud providers validate formats server-side
   - Focus on business rules (length limits), not format validation

2. **No Over-Normalization**
   - Don't normalize controlled/hardcoded values
   - Trust our controlled .tfvars files

3. **No Validation Duplication**
   - Same validation in multiple places
   - Single source of truth: module-specific `validations.tf`

4. **No Hieroglyphic Code**
   - If it needs extensive comments to understand, simplify it
   - Code should be self-explanatory

5. **No Blind Review Implementation**
   - Evaluate suggestions against official documentation
   - Consider our controlled environment
   - Push back on complexity without benefit

### Module Structure That Passes Review

- **main.tf** - Resources with depends_on validations
- **validations.tf** - ALL validations centralized here  
- **variables.tf** - Variable definitions only - NO validation blocks
- **outputs.tf** - Mark PII as sensitive = true
- **versions.tf** - Pinned provider versions (not "~>")

*See [code-examples.md](code-examples.md#📁-module-structure-examples) for complete structure*

---

## 📝 Code Guidelines

### Core Principles - CRITICAL Rules

- **CRITICAL: NEVER run `terraform apply` or any state-changing commands without explicit user approval**
- **CRITICAL: Before starting any new task, always pull/fetch from master and create a new branch with the latest changes from master**
- **CRITICAL: ALWAYS check current branch with `git branch --show-current` BEFORE making any changes. If in master, stop immediately and create a new branch first.**

### Infrastructure Safety Guidelines

**CRITICAL: Infrastructure changes must be handled with extreme care to prevent downtime and service disruption.**

#### Review Suggestion Evaluation

Before implementing any review suggestion, ask:

1. **Verify Against Official Documentation First**
   - Check official Google Cloud documentation for actual requirements
   - Verify Terraform Google provider official patterns and recommendations
   - Research if the suggestion contradicts official best practices

2. **Is this applicable to our controlled environment?**
   - Do we have controlled vs dynamic inputs?
   - Are values hardcoded vs user-provided?
   - Is this defensive programming vs practical necessity?

3. **Cost vs Benefit Analysis**
   - **High benefit**: Security fixes, actual bugs, performance improvements
   - **Medium benefit**: Code clarity, maintainability improvements  
   - **Low benefit**: Theoretical edge cases that won't occur in our controlled environment

4. **When to push back with explanations**
   - Normalization for hardcoded values we control
   - Validation for constants that can't be wrong
   - Complex defensive patterns for simple, controlled scenarios
   - Suggestions that contradict official documentation

**Comment template for overkill suggestions:**

```markdown
This suggestion contradicts official documentation and is not applicable to our controlled infrastructure environment:

**Official Documentation Source:**
- [Link to GCP/Terraform official docs that contradicts this suggestion]
- [Specific quote or rule from official source]

**Why it's not needed:**
- [Specific reason - controlled input, hardcoded values, etc.]
- [Context about our environment]
- [Official guidance supports our approach]

**When this would be valuable:**
- [Scenarios where this makes sense]

**Our approach:**
- [Why our current approach follows official recommendations]
- [Reference to official best practices we follow]
```

### Defensive Engineering Principles

**Goal**: Write code that survives hostile environments and unexpected conditions.

**CodeRabbit Advantage**: Trained on production failures - knows what breaks in the real world.

#### Core Defensive Engineering Rules

1. **Engineer for Hostile Environments**
   - Use absolute paths with `ROOT="$(git rev-parse --show-toplevel)"`
   - Handle CWD changes in pre-commit hooks
   - Plan for simultaneous file staging scenarios

2. **Validate Everything**
   - Check file existence before operations
   - Verify directory structure before writes
   - Confirm expected state before proceeding

3. **Apply the "What If" Method**
   - Test edge cases: CWD changes, missing files, network failures
   - Consider user errors: typos, wrong staging, unexpected inputs
   - Simulate production conditions: CI environments, permission limits

4. **Design Recovery-First**
   - Include error handling from the start
   - Provide specific commands for error recovery
   - Make error messages actionable with exact steps

#### Implementation Success Patterns

- ✅ **Use robust paths**: `ROOT="$(git rev-parse --show-toplevel)"`
- ✅ **Validate inputs**: `[[ -f "$file" ]] || { echo "File not found"; exit 1; }`  
- ✅ **Handle conflicts**: Check for double-staging before operations
- ✅ **Guide recovery**: "To fix: 1. git restore --staged file 2. Edit source 3. git add source"
- ✅ **Test environments**: Verify in CI conditions, not just local

**Principle**: "Assume Murphy's Law - if something can go wrong, it will"

**Success Metric**: Code works the same locally, in CI, and when things go wrong.

### Terraform Validation Best Practices

1. **Validation File Separation** - ALWAYS use dedicated `validations.tf` files
2. **Security-by-Design Defaults** - Apply security patterns proactively
3. **Fail-Fast Validation Dependencies** - Enforce validation order
4. **Clear Error Message Standards** - Use what/why/how pattern
5. **Input Normalization** - Always normalize user inputs
6. **Null Safety** - Handle null values defensively

### No Half-Measures Rule

**MANDATORY: When implementing infrastructure changes, fixes must be comprehensive across the entire codebase.**

We won't do half-finished work:

- When fixing an issue in one module, ALL modules must be checked and updated
- Never leave some modules using old patterns while others use new patterns
- Use grep/search to find ALL instances of what you're changing

### Definition of Done

Before marking ANY task as completed:

1. ✅ **Code Changes**: All required modifications implemented
2. ✅ **Validation**: `terraform validate` passes
3. ✅ **Plan Test**: `terraform plan -var-file=projects/$(terraform workspace show).tfvars` successful
4. ✅ **No Regressions**: Plan shows expected changes only
5. ✅ **Validation Architecture**: Active `terraform_data` with `depends_on` enforcement
6. ✅ **Security Compliance**: PII marked sensitive, external APIs wrapped
7. ✅ **Error Message Quality**: What/why/how format
8. ✅ **Null Safety Compliance**: All `merge()` calls use `coalesce()`

---

## 🔄 GitHub Workflow Guidelines

### MANDATORY Workflow Checklist

**EVERY task must start with this checklist - NO EXCEPTIONS:**

#### Option A: Automated (Recommended)

- Run `./scripts/safe-start.sh [optional-task-name]` - handles all checks automatically

#### Option B: Manual Steps

1. Check current branch with `git branch --show-current`
2. If in master - create feature branch immediately
3. Pull latest changes from master
4. Set appropriate testing workspace: `terraform workspace select <workspace>`
5. Verify branch protection is active

*See [code-examples.md](code-examples.md#git-workflow-examples) for complete commands*

### Git Merge Rules

#### CRITICAL: Prefer merge over rebase for safety

- Use `git pull --no-rebase` (merge strategy) instead of rebase
- Preserves complete commit history and reduces conflict complexity
- Safer for infrastructure code where history matters

### Git Commit Amendment Rules

#### CRITICAL: Avoid creating conflicts with commit amendments

#### When to use `git commit --amend`

- ✅ **BEFORE pushing**: Safe to amend local commits
- ✅ **Fixing typos**: Only if commit is still local

#### When NOT to use `git commit --amend`

- ❌ **AFTER pushing**: Never amend pushed commits
- ❌ **On shared branches**: Others may have based work on your commits

### Pre-commit Hooks Setup

- Install: `brew install pre-commit`
- Setup: `pre-commit install`  
- Manual run: `pre-commit run --all-files`

**Configured Hooks**: terraform fmt, terraform validate, terraform docs, checkov, file checks

*See [code-examples.md - Pre-commit Hooks](code-examples.md#pre-commit-hooks) for detailed commands and hook sequencing*

### Critical Notes

- **Repository requires GPG-signed commits. NEVER use --no-gpg-sign flag**
- **NEVER use `git commit --amend` after a commit has been pushed**
- **NEVER begin changes that affect Kubernetes cluster operations without explicit approval**

---

## ⚙️ Terraform Best Practices

### Quick Rules (Condensed)

- Centralize validations in `validations.tf`; enforce via `terraform_data.validations` and `depends_on`
- Normalize once via locals; use standardized null‑safe patterns (`try`, `coalesce`, `trimspace`)
- Prefer ID‑based existence checks in outputs; mark PII outputs sensitive
- Cloud providers validate formats - focus on business rules, not format validation
- Keep error messages short and actionable
- Use `try()/coalesce()` length checks with `trimspace()` for compatibility - these are the preferred, consistent patterns

### Industry Standards

Based on HashiCorp, Google Cloud, and Terrateam recommendations.

### Validation & Normalization Rules

- **Validation Architecture**: Place all module validations in `validations.tf` using `terraform_data`
- **Normalization Rules**: Normalize inputs once via locals and reuse
- **Null‑Safety Conventions**:
  - Strings: `length(trimspace(coalesce(var.s, ""))) > 0`
  - Lists/maps: `length(coalesce(var.l, []))`
  - Loops: `for x in try(var.l, [])`
- **Error Message Style**: Use what/why/how with current value

### Module Creation Checklist

1. Create structure: `main.tf`, `variables.tf`, `validations.tf`, `outputs.tf`
2. Variables.tf: Definitions ONLY - no validation blocks
3. Validations.tf: ALL validations centralized with `terraform_data`
4. Main.tf: Resources with `depends_on = [terraform_data.validations]`
5. Outputs.tf: Mark PII as `sensitive = true`

### Essential Commands

**Workspace Operations:**

- List workspaces: `terraform workspace list`
- Select workspace: `terraform workspace select <client>`
- Plan with workspace: `terraform plan -var-file=projects/$(terraform workspace show).tfvars`

**Pre-PR Audits:**

- `make audit-validations` - Check terraform_data enforcement
- `make audit-errors` - Verify error message quality  
- `make audit-optional-fields` - Ensure null safety

*See [code-examples.md](code-examples.md#terraform-commands) for complete command reference*

### Workspace Rules

- **ALWAYS use dynamic workspace reference**: `$(terraform workspace show).tfvars`
- **ALWAYS use `spot` instead of `preemptible`** in node pools

---

## 📚 Additional Resources

### Reference Documentation

- [HashiCorp Custom Conditions](https://developer.hashicorp.com/terraform/tutorials/configuration-language/custom-conditions)
- [Google Cloud Module Best Practices](https://cloud.google.com/docs/terraform/best-practices/reusable-modules)
- [Terrateam Code Organization](https://terrateam.io/blog/terraform-code-organization)

---

## Important Instruction Reminders

- **KISS (Keep It Simple, Stupid)** is our fundamental principle guiding ALL decisions
- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User
- **Simplicity > Complexity** in ALL decisions - when in doubt, choose the simpler solution
- Trust our controlled environment - .tfvars files are carefully managed by experienced engineers
- Cloud providers validate formats server-side - focus on business logic, not format validation

### Make Targets (Automation)

**Audits:** `make audit-validations`, `make audit-errors`, `make audit-optional-fields`, `make audit-ci`

All targets are non-destructive and fail on findings to prevent issues before they reach PR stage.

---

## 📋 References

### Related Documentation

- **[code-examples.md](code-examples.md)** - All code examples, patterns, and commands
- **[terraform-best-practices.md](terraform-best-practices.md)** - Industry standards from HashiCorp, Google Cloud, and Terrateam
- **[coderabbit-review-insights.md](coderabbit-review-insights.md)** - Detailed review patterns analysis and prevention strategies

### External References

- **[HashiCorp Custom Conditions](https://developer.hashicorp.com/terraform/tutorials/configuration-language/custom-conditions)** - Official validation patterns
- **[Google Cloud Module Best Practices](https://cloud.google.com/docs/terraform/best-practices/reusable-modules)** - Infrastructure standards
- **[Terrateam Code Organization](https://terrateam.io/blog/terraform-code-organization)** - Enterprise patterns

### Quick Navigation

- **Need code examples?** → [code-examples.md](code-examples.md)
- **Want industry best practices?** → [terraform-best-practices.md](terraform-best-practices.md)
- **Want to prevent PR comments?** → [coderabbit-review-insights.md](coderabbit-review-insights.md)
- **Looking for specific patterns?** → Use the search function in the examples file
- **Understanding review feedback?** → Check the insights file for context and solutions
