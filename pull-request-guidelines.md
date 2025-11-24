# Pull Request Guidelines

## Before Opening a PR

### ✅ Checklist

- [ ] Code works and has been tested locally
- [ ] All tests pass (Optional for now till you get used to the flow...)
- [ ] Code follows project style guidelines
- [ ] Documentation is updated
- [ ] Commit messages follow standards
- [ ] Branch is up to date with main
- [ ] No merge conflicts
- [ ] Secrets/credentials removed
- [ ] Self-review completed

## PR Title Format

Follow the same convention as commit messages:

```
<type>(<scope>): <description>
```

### Examples

```
feat(cli): add JSON output format option
fix(terraform): resolve provider version conflict
docs: update Kubernetes deployment guide
refactor(k8s): simplify ingress configuration
```

## PR Description Template

Use this template for all PRs:

```markdown
## Description
Brief description of what this PR does.

## Changes
- List of specific changes made
- Another change
- One more change

## Why
Explain the reasoning behind these changes.

## Testing
How did you test this?
- [ ] Tested locally
- [ ] Added unit tests
- [ ] Tested in real environment
- [ ] Tested edge cases

## Screenshots (if applicable)
Add screenshots or terminal output.

## Related Issues
Closes #123
Related to #456

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] Tests added/updated
- [ ] No breaking changes (or documented if present)
```

## PR Size Guidelines

### Small PRs Are Better

**Ideal**: 50-200 lines changed
**Maximum**: 400 lines changed

### Why Small PRs?

- Faster reviews
- Easier to understand
- Less likely to introduce bugs
- Easier to revert if needed
- Better for reviewers

**Solution**: Split into multiple PRs

### How to Split Large PRs
1. Identify independent changes
2. Create separate branches
3. Submit PRs in logical order
4. Mark dependencies clearly

Example:
```
PR 1: Add database schema
PR 2: Add API endpoints (depends on PR 1)
PR 3: Add frontend UI (depends on PR 2)
```

## What Makes a Good PR

### Clear Title & Description
- Explains **what** changed
- Explains **why** it changed
- Includes context for reviewers

### Single Responsibility
- One feature/fix per PR
- Related changes grouped together
- No unrelated modifications

### Well-Structured Code
- Clean, readable code
- Follows project conventions
- Includes comments where needed
- No commented-out code
- No debugging statements

### Comprehensive Testing
- Tests included
- Edge cases covered
- Manual testing documented

### Updated Documentation
- README updated if needed
- Code comments added
- API docs updated
- Examples provided

## Draft vs Ready PRs

### Use Draft PRs When
- Work is in progress
- You want early feedback
- You're blocked and need help
- You want to share approach

**Mark as Draft** and add `[WIP]` or `[Draft]` to title

### Ready PRs
- Code is complete
- Tests pass
- Ready for review
- Ready to merge

## Requesting Reviews

### Who to Request
- Your pod members (always)
- Subject matter experts (if needed)
- Code owners (automatic)

### When to Request
- After self-review
- When PR is "ready"
- Not for WIP/Draft PRs (unless asking for feedback)

### How Many Reviewers
- **Minimum**: 1 approval
- **Recommended**: 2 approvals
- **Critical changes**: 2+ approvals

## Responding to Review Feedback

### Be Receptive
- Reviews make code better
- Don't take feedback personally
- Ask questions if unclear
- Thank reviewers for their time

### Try to respond to Every Comment
- ✅ "Fixed in commit abc123"
- ✅ "Good catch, updated"
- ✅ "I disagree because X, what do you think?"
- ✅ "Will address in follow-up PR"

### Don't

- ❌ Ignore comments
- ❌ Get defensive or take it personally.
- ❌ Make changes without replying
- ❌ Mark as resolved without addressing

### When You Disagree

Explain your reasoning:

```
I kept the current approach because:
- [technical reason]
- [performance consideration]
- [consistency with existing code]

Open to changing if you feel strongly, but wanted
to explain the tradeoff.
```

## Making Changes After Review

### Small Changes

```bash
# Make the changes
git add <files>
git commit -m "refactor: address PR feedback"
git push origin feature/your-branch
```

### Many Changes

Consider squashing commits before merge (if allowed).

### Force Push (Use Carefully)
Only force push if:
- You've rebased with main
- You're fixing commit history
- No one else is working on the branch

```bash
git push --force-with-lease origin feature/your-branch
```

Never use `--force` without `--force-with-lease`

## CI/CD Checks

### Must Pass Before Merge

- ✅ All tests pass
- ✅ Linting passes
- ✅ Security scans pass
- ✅ Build succeeds

### If Checks Fail

1. Click on the failed check
2. Read the error message
3. Fix the issue locally
4. Push the fix
5. Wait for checks to re-run

### Common CI Failures

- Linting errors → Fix formatting
- Test failures → Fix tests or code
- Build failures → Check dependencies
- Security issues → Remove vulnerable deps

## Merging Your PR

### Merge Requirements

- ✅ Minimum approvals met
- ✅ All CI checks pass
- ✅ No unresolved comments
- ✅ Up to date with main branch
- ✅ Merge conflicts resolved

### Merge Methods

#### Squash and Merge (Recommended)

- Combines all commits into one
- Clean history on main
- Use for feature branches

#### Merge Commit

- Preserves all commits
- Shows complete history
- Use for important features

#### Rebase and Merge

- Replays commits on main
- Linear history
- Use if commits are clean

**Default**: Squash and Merge for most PRs

### After Merging

1. Delete the feature branch
2. Close related issues
3. Update project board
4. Celebrate 🎉

## PR Labels (if applicable)

Common labels to use:
- `bug` - Bug fixes
- `enhancement` - New features
- `documentation` - Docs changes
- `breaking-change` - Breaking changes
- `needs-review` - Ready for review
- `work-in-progress` - WIP
- `blocked` - Blocked by something
- `good-first-issue` - For new contributors

## PR Examples

### Good PR Example

```markdown
Title: feat(cli): add --dry-run flag for safe testing

## Description
Adds a --dry-run flag that shows what would be deployed
without actually making changes. Useful for validating
configurations before real deployments.

## Changes
- Added --dry-run flag to CLI parser
- Added validation-only mode to deployment logic
- Updated help text and documentation
- Added tests for dry-run mode

## Why
Users requested a way to validate configurations without
risk. This is standard in tools like kubectl and terraform.

## Testing
- [x] Tested locally with various configs
- [x] Added unit tests (95% coverage)
- [x] Tested error cases
- [x] Updated integration tests

## Screenshots
```
$ ./cli deploy --dry-run --config prod.yaml
[DRY RUN] Would deploy:
  - Service: api-server (replicas: 3)
  - Ingress: api.example.com
  ✓ Configuration is valid
```

Closes #234
```

### Bad PR Example

```markdown
Title: updates

## Description
made some changes

## Changes
lots of stuff
```

## Quick Tips

1. **Small PRs get merged faster** - Break work into chunks
2. **Good descriptions save time** - Reviewers understand context immediately
3. **Self-review first** - Catch issues before others do
4. **Be responsive** - Reply to comments within 24 hours
5. **Keep it updated** - Rebase regularly to avoid conflicts
6. **Test thoroughly** - Don't rely on CI to find bugs
7. **Document changes** - Update docs in the same PR
8. **Link issues** - Use "Closes #123" to auto-close issues

## Resources

- [About Pull Requests - GitHub](https://docs.github.com/en/pull-requests)
- [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)
- [Effective Code Reviews](https://google.github.io/eng-practices/review/)

---

**Remember**: A good PR is a gift to your reviewers and your future self!