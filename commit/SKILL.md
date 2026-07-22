---
name: commit
description: "Create a conventional commit message for staged changes. Usage: /commit [optional context] [--push] [--pr]"
disable-model-invocation: false
---

**User-provided context:** $ARGUMENTS

**Parse flags from the context before anything else:**
   - `--push`: after committing, push the commit to the remote.
   - `--pr`: after committing, push the commit and open a pull request. `--pr` implies `--push`.
   - Strip any recognised flags out of the context. Only the remaining text is the
     "user-provided context" used to shape the commit message — do NOT let `--push` or
     `--pr` leak into the commit message itself.

Please create a commit for the staged changes following these guidelines:

1. Run these git commands in parallel to understand the changes and the current branch:
   - `git status` to see all staged files
   - `git diff --staged` to see the actual changes
   - `git log --oneline -10` to understand the commit message style
   - `git rev-parse --abbrev-ref HEAD` to determine the current branch
   - `git symbolic-ref --short refs/remotes/origin/HEAD` to determine the default branch
     (fall back to `main`/`master` if this is not set). This returns a remote-qualified
     ref such as `origin/main`; strip the `origin/` prefix before comparing it with the
     current branch (e.g. `origin/main` → `main`).

2. **Guard against committing on a detached HEAD or the default branch — do this before anything else:**
   - If `git rev-parse --abbrev-ref HEAD` returned `HEAD`, you are on a detached HEAD. STOP
     immediately — do NOT commit, push, or open a PR. Tell the user they are on a detached
     HEAD and must create a branch first, e.g. `git switch -c <type>/<short-description>`.
   - If the current branch IS the default branch (after stripping the `origin/` prefix, e.g.
     `main`/`master`), STOP immediately.
   - Do NOT commit, push, or open a PR.
   - Print a message telling the user they are on the default branch and must fork into a
     new branch before continuing, and suggest a command, e.g.
     `git switch -c <type>/<short-description>`.
   - Only continue with the steps below once the user is on a non-default branch.

3. **Incorporate user-provided context into the commit message:**
   - If context is provided above, it MUST influence the final commit message
   - Use the context to determine the appropriate commit type (feat, fix, refactor, etc.)
   - Reflect the context in the short description on the first line
   - Include relevant details from the context in the bullet points
   - The context explains the "why" - use it to make the message more meaningful
   - Example: if context is "performance optimization for search", the message should mention performance/search, not just generic "update code"
   - If no context is provided, generate the message purely from the staged changes

4. Analyze the staged changes and create a conventional commit message following this format:
   ```
   <type>(<scope>): <short description>

   - Bullet point summary of changes
   - Additional context if needed
   ```

   **CRITICAL LINE LENGTH REQUIREMENT:**
   - The commit message body MUST NOT exceed 100 characters per line
   - If a bullet point is longer than 100 characters, wrap it across multiple lines
   - Use proper indentation (2 spaces) for wrapped lines to maintain readability

5. Commit types to use:
   - `feat`: New feature
   - `fix`: Bug fix
   - `chore`: Maintenance tasks, configuration, dependencies
   - `refactor`: Code restructuring without changing behavior
   - `docs`: Documentation changes
   - `test`: Adding or updating tests
   - `style`: Code style/formatting changes
   - `perf`: Performance improvements
   - `ci`: CI/CD changes

6. Create the commit using a heredoc for proper formatting:
   ```bash
   git commit -m "$(cat <<'EOF'
   <type>(<scope>): <description>

   - First bullet point with concise details
   - Second bullet point, wrapped to multiple lines if needed to stay
     under 100 characters per line
   - Third bullet point
   EOF
   )"
   ```

   **Example of proper line wrapping:**
   ```bash
   git commit -m "$(cat <<'EOF'
   feat(app): add user authentication with JWT tokens

   - Implement JWT-based authentication with refresh token rotation
   - Add login, logout, and token refresh endpoints to API routes
   - Create authentication middleware for protected route validation
   - Update user schema to include hashed password and refresh tokens
   EOF
   )"
   ```

7. **If `--push` or `--pr` was passed, push the commit to the remote:**
   - The default-branch guard in step 2 guarantees you are on a non-default branch by now.
   - Push the current branch with `git push -u origin <branch>` so upstream tracking is set.
   - Report the push result to the user. If the push fails (e.g. non-fast-forward, auth,
     protected branch), report the failure message as-is and stop — do NOT force-push.

8. **If `--pr` was passed, open a pull request after a successful push:**
   - Use the `gh` CLI: `gh pr create`.
   - Base the PR title on the commit's short description and write a concise body
     summarising the "why" of the change (reuse the commit bullet points as a starting
     point).
   - Create the PR with a heredoc for the body so formatting is preserved:
     ```bash
     gh pr create --title "<type>(<scope>): <description>" --body "$(cat <<'EOF'
     ## Summary
     - Bullet point summary of the change

     ## Why
     - The motivation / context behind the change

     🤖 Generated with [Claude Code](https://claude.com/claude-code)
     EOF
     )"
     ```
   - Report the resulting PR URL to the user. If `gh` is not installed or authenticated,
     or PR creation fails, report the error as-is and stop.

## IMPORTANT NOTES - DO NOT IGNORE THE FOLLOWING:
   - Do NOT add any co-author to the commit message
   - Keep the commit message concise and focused on the "why" rather than the "what"
   - Under any circumstances do not circumvent the commit signing process. All commits should be signed as per the local configuration.

## Handling Git Lifecycle Hooks
   - Git hooks (e.g. pre-commit, commit-msg) may run automatically during commits
   - If a hook fails, report the failure message to the user as-is
   - Do NOT bypass hooks using flags like `--no-verify` or `--no-gpg-sign`
   - Do NOT attempt to fix hook failures automatically or retry the commit
   - Present the failure and let the user decide how to proceed
