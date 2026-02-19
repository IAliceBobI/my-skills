---
argument-hint: [--style=simple|full] [--type=feat|fix|docs|style|refactor|perf|test|chore|ci|build|revert] [--no-push] [--force]
description: Create well-formatted commits with conventional commit messages and automatic push
---

# Claude Command: Commit

This command helps you create well-formatted commits following the Conventional Commits specification, with automatic push to remote repository by default.

## Usage

Basic usage (commit and push):
```
/commit
```

Commit without pushing:
```
/commit --no-push
```

With force push (for amended commits):
```
/commit --force
```

With other options:
```
/commit --style=simple
/commit --style=full
/commit --style=full --type=fix
/commit --no-push --style=simple
```

## Command Options

- `--style=simple|full`:
  - `full` (default): Creates detailed commit messages with body and footer sections
  - `simple`: Creates concise single-line commit messages
- `--type=<type>`: Specify the commit type (default: `feat`, overrides automatic detection)
- `--no-push`: Skip automatic push to remote repository after committing
- `--force`: Use force push when pushing (use with caution, mainly for amended commits)

## What This Command Does

1. **Pre-commit checks**: Always skipped (noverify)
   - No pre-commit hooks run

2. **File staging**:
   - Check staged files with `git status`
   - If no files staged, automatically add all modified/new files with `git add`

3. **Change analysis**:
   - Run `git diff` to understand changes
   - Detect if multiple logical changes should be split
   - Suggest atomic commits when appropriate

4. **Commit message creation**:
   - Generate messages following Conventional Commits specification
   - Apply appropriate emoji prefixes
   - Add detailed body/footer in full style mode

5. **Automatic push to remote** (default behavior):
   - Push commit to remote repository after successful commit
   - Uses `git push` by default, or `git push --force` with `--force` flag
   - Skip push if `--no-push` flag is provided
   - Verifies push success and provides feedback

## Conventional Commits Format

### Simple Style
```
<emoji> <type>[optional scope]: <description>
```
Example: `✨ feat(auth): add JWT token validation`

### Full Style  
```
<emoji> <type>[optional scope]: <description>

<body>

<footer>
```

Example:
```
✨ feat(auth): add JWT token validation

Implement JWT token validation middleware that:
- Validates token signature and expiration
- Extracts user claims from payload
- Adds user context to request object
- Handles refresh token rotation

This change improves security by ensuring all protected 
routes validate authentication tokens properly.

BREAKING CHANGE: API now requires Bearer token for all authenticated endpoints
Closes: #123
```

## Commit Types & Emojis

| Type | Emoji | Description | When to Use |
|------|-------|-------------|-------------|
| `feat` | ✨ | New feature | Adding new functionality |
| `fix` | 🐛 | Bug fix | Fixing an issue |
| `docs` | 📝 | Documentation | Documentation only changes |
| `style` | 🎨 | Code style | Formatting, missing semi-colons, etc |
| `refactor` | ♻️ | Code refactoring | Neither fixes bug nor adds feature |
| `perf` | ⚡️ | Performance | Performance improvements |
| `test` | ✅ | Testing | Adding missing tests |
| `chore` | 🔧 | Maintenance | Changes to build process or tools |
| `ci` | 👷 | CI/CD | Changes to CI configuration |
| `build` | 📦 | Build system | Changes affecting build system |
| `revert` | ⏪ | Revert | Reverting previous commit |

## Body Section Guidelines (Full Style)

The body should:
- Explain **what** changed and **why** (not how)
- Use bullet points for multiple changes
- Include motivation for the change
- Contrast behavior with previous behavior
- Reference related issues or decisions
- Be wrapped at 72 characters per line

Good body example:
```
Previously, the application allowed unauthenticated access to
user profile endpoints, creating a security vulnerability.

This commit adds comprehensive authentication middleware that:
- Validates JWT tokens on all protected routes
- Implements proper token refresh logic
- Adds rate limiting to prevent brute force attacks
- Logs authentication failures for monitoring

The change follows OAuth 2.0 best practices and improves
overall application security posture.
```

## Footer Section Guidelines (Full Style)

Footer contains:
- **Breaking changes**: Start with `BREAKING CHANGE:`
- **Issue references**: `Closes:`, `Fixes:`, `Refs:`
- **Co-authors**: `Co-authored-by: name <email>`
- **Review references**: `Reviewed-by:`, `Approved-by:`

Example footers:
```
BREAKING CHANGE: rename config.auth to config.authentication
Closes: #123, #124
Co-authored-by: Jane Doe <jane@example.com>
```

## Scope Guidelines

Scope should be:
- A noun describing the section of codebase
- Consistent across the project
- Brief and meaningful

Common scopes:
- `api`, `auth`, `ui`, `db`, `config`, `deps`
- Component names: `button`, `modal`, `header`
- Module names: `parser`, `compiler`, `validator`

## Commit Splitting Strategy

Automatically suggest splitting when detecting:
1. **Mixed types**: Features + fixes in same commit
2. **Multiple concerns**: Unrelated changes
3. **Large scope**: Changes across many modules
4. **File patterns**: Source + test + docs together
5. **Dependencies**: Dependency updates mixed with features

## Best Practices

### DO:
- ✅ Write in present tense, imperative mood ("add" not "added")
- ✅ Keep first line under 50 characters (72 max)
- ✅ Capitalize first letter of description
- ✅ No period at end of subject line
- ✅ Separate subject from body with blank line
- ✅ Use body to explain what and why vs. how
- ✅ Reference issues and breaking changes

### DON'T:
- ❌ Mix multiple logical changes in one commit
- ❌ Include implementation details in subject
- ❌ Use past tense ("added" instead of "add")
- ❌ Make commits too large to review
- ❌ Commit broken code (unless WIP)
- ❌ Include sensitive information

## Examples

### Simple Style Examples
```bash
✨ feat: add user registration flow
🐛 fix: resolve memory leak in event handler
📝 docs: update API endpoints documentation
♻️ refactor: simplify authentication logic
⚡️ perf: optimize database query performance
🔧 chore: update build dependencies
```

### Full Style Example
```bash
✨ feat(auth): implement OAuth2 authentication flow

Add complete OAuth2 authentication system supporting multiple
providers (Google, GitHub, Microsoft). The implementation 
follows RFC 6749 specification and includes:

- Authorization code flow with PKCE
- Refresh token rotation
- Scope-based permissions
- Session management with Redis
- Rate limiting per client

This provides users with secure single sign-on capabilities
while maintaining backwards compatibility with existing
JWT authentication.

BREAKING CHANGE: /api/auth endpoints now require client_id parameter
Closes: #456, #457
Refs: RFC-6749, RFC-7636
```

## Workflow

1. Analyze changes to determine commit type and scope
2. Check if changes should be split into multiple commits
3. For each commit:
   - Stage appropriate files
   - Generate commit message based on style setting
   - If full style, create detailed body and footer
   - Execute git commit with generated message
4. Push to remote (default behavior, unless `--no-push` is specified):
   - Determine push mode (normal or force)
   - Execute git push with appropriate flags
   - Verify push success and provide feedback
5. Provide summary of committed changes and push status

## Push Behavior

### Automatic Push (Default)
- Automatically pushes commits to remote branch after committing
- Safe for most workflow scenarios
- Keeps remote repository in sync with local changes
- Skip with `--no-push` flag if you only want to commit locally

### Force Push (`--force`)
- Overwrites remote history
- Use with caution!
- Appropriate for:
  - Amended commits that haven't been pushed yet
  - Squashed commits
  - Corrected commit messages
- NOT appropriate if:
  - Others have pulled the commits
  - Working on shared branch with active collaboration
  - Commit history is already published

### Push Safety Checks
Before pushing, the tool will:
1. Check if branch has upstream configured
2. Verify commit is successful
3. Warn if using force push on potentially shared branches
4. Confirm push completed successfully

## Important Notes

- Default style is `full --type=feat` for detailed commits
- Use `simple` style for quick, everyday commits
- The tool will intelligently detect when full style might be beneficial and suggest it
- Always review the generated message before confirming
- **Push is automatic by default** - use `--no-push` if you only want to commit locally
- Force push should only be used when you're certain it's safe

## Examples

### Commit with simple style (automatic push)
```bash
/commit --style=simple
# Creates: ✨ feat: add user profile page
# Automatically pushes to remote
```

### Commit without pushing
```bash
/commit --no-push --style=simple
# Creates commit but does not push
# Useful for local work or WIP commits
```

### Commit with full style and automatic push
```bash
/commit --style=full --type=fix
# Creates detailed commit with body and footer
# Automatically pushes to remote
```

### Commit and force push (for amended commit)
```bash
/commit --force
# Commits and force pushes to update remote
# Useful when you amended the last commit
```

### Typical workflow
```bash
# Make some changes
# Commit (automatically pushes)
/commit --style=simple

# Output:
# ✓ Staged 3 files
# ✓ Created commit: ✨ feat(auth): add JWT token validation
# ✓ Pushed to origin/main
```

### Local development workflow
```bash
# Work on feature locally, don't push yet
/commit --no-push --style=simple

# Later, push manually when ready
git push
```

### Amend and force push workflow
```bash
# Make small change to last commit
/commit --force

# Output:
# ✓ Amended commit: ✨ feat(auth): add JWT token validation
# ⚠️  Force pushed to origin/main (branch was behind by 1 commit)
```