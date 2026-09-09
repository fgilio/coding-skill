---
name: coding
description: >
  Franco's code style and change-landing preferences.
  Use when: writing code, reviewing code, refactoring, converting raw
  PHP to framework idioms, adding comments, writing docs, naming
  functions/variables, creating error messages, rebasing or updating
  branches, structuring PRs.
user-invocable: true
disable-model-invocation: false
---

# Code Style: Expressive & Readable

Write code that feels eloquent and expressive. Repo conventions win on mechanics (indentation, quotes, import order). These rules win on structure and prose.

## General Principles

- **Fluent interfaces**: Chain methods naturally
- **Descriptive names**: Full words that read like sentences. Predicates read as questions (is/has/can/should)
- **Early returns**: Reduce nesting
- **Immutability first**: Prefer const/final where possible

## Coherent Interfaces

Design from the call site. Read the producer, operation, returned value, stored representation, and consumers as one contract. Apply these rules when writing and reviewing code in the changed flow:

- **Names promise behavior**: A caller should be able to predict what an operation returns, changes, or refuses. Name the actual behavior, including relevant units and representation. A method that only reports invalid orders should be `reportInvalidOrders()`, not `rejectInvalidOrders()`.
- **Name related concepts together**: Use one vocabulary for the same concept across methods, variables, properties, bindings, and tests. Choose the names as a set before proposing a rename. Keep established domain terms unless they misstate the contract.
- **Symmetry follows meaning**: Similar contracts should look similar. Preserve names that distinguish different behavior. Do not merge two outputs with different meanings just to make their callers uniform.
- **Ownership follows responsibility**: Put an operation where its context and callers live. Request-specific work stays at the request boundary. Reusable transformations take no request or UI context they do not need. A helper or wrapper earns its place by marking that boundary, not by shortening a call.
- **One shared rule, one derivation**: Centralize repeated calculations or conversions when they express the same rule. Do not merge similar expressions that have different failure behavior or reasons to change. A base value can be shared while its variants remain explicit.
- **Expose only the intended interface**: Keep implementation details private or protected.
- **Change interfaces as a set**: Check callers, framework hooks, and serialized or external contracts before changing a name, owner, or visibility. Move consumers and tests with the change.

```php
// Avoid: the consumer's name hides the amount's unit.
$total = $order->totalInCents();

// Prefer: the producer and consumer describe the same value.
$totalInCents = $order->totalInCents();

// Keep: only the total includes tax.
$subtotalInCents = $order->subtotalInCents();
$totalInCents = $order->totalInCents();
```

Prefer a local change that makes a real caller easier to understand. Comments can explain a hidden constraint, but should not have to correct a misleading name.

## JavaScript/TypeScript Examples

```js
// Expressive method chaining
users
  .filter(user => user.isActive)
  .sort((a, b) => b.createdAt - a.createdAt)
  .slice(0, 5)
  .map(user => user.profile);

// Descriptive function names
function canUserEditPost(user, post) { ... }  // Not: checkEdit()
await fetchUserWithPosts(userId);             // Not: getUser(userId, true)

// Object methods that read naturally
post.belongsToUser(user);
order.hasStatus('pending');
```

## Laravel Codebases

Working in PHP/Laravel? Read [laravel.md](laravel.md) before writing code.

## Guard Clauses: One Guard Per Exit Reason

When two conditions exit differently (silent return, logged skip, exception), write two sequential guards. Never merge them into one umbrella condition and re-branch inside it. A nested `if` inside a guard body means two guards were collapsed.

```php
// DON'T: umbrella guard, then re-discriminate inside
if (! $user->hasVerifiedEmail()) {
    if ($user->email !== null) {
        $this->logSkip($user);
    }

    return;
}

// DO: each guard states its condition and owns its consequence
if ($user->email === null) {
    return;
}

if (! $user->hasVerifiedEmail()) {
    $this->logSkip($user);

    return;
}
```

This usually creeps in when adding a new exit condition next to an existing guard. Add it as its own guard instead of widening the existing condition and patching the difference inside.

## Code Comments: Laravel-Style Precision

Write comments like Taylor Otwell. Technical, concise, and clean:

```php
/**
 * Builds the monthly statement for a customer account.
 *
 * Amounts stay in cents until the final formatting step,
 * so rounding never drifts from line items to total.
 */
```

Comment density: a why-comment is a rare event. The default for any line, property, or well-named function is no comment at all. A docblock earns its place by carrying something the name and signature cannot; most functions need none, and "docblocks on classes and functions" is never a mandate to write one. A why-comment names a constraint the code can't show, not what the next line plainly does. Budget check on a finished diff: if it reads as commentary with code interleaved, cut until only the constraints the code cannot show remain. One or two short lines is the ceiling for any single why; a paragraph-sized rationale belongs in the PR description.

```php
// DON'T: narrates what the next line plainly does
// Filter to active users and take the newest five
$recent = $users->where('is_active', true)->sortByDesc('created_at')->take(5);
```

- **Don't restate documented conventions**: if a rule already lives at the project or directory level (a `CLAUDE.md`, a framework convention every reader of this code knows), a comment re-explaining it is noise that drifts. Cut it, keep only what is specific to _this_ code. In a SingleStore migration, `// no foreign keys` restates the repo-wide rule (cut), but `// a unique key must contain the shard key, so id drops its primary key` explains this table's own choice (keep).
- **Put the why on the line it governs**: attach a why-comment to the specific declaration or statement it explains, not in a header block above the whole unit. When a header paragraph explains one column's key choice or one line's guard, move that sentence down onto that column or line. Reserve the header block for what is genuinely about the whole function or class.
- **Say a why once**: one constraint gets one comment, at the single place it binds. Never repeat the same rationale across a docblock, a call site, a test, and a doc. Everywhere else the code points at the place that carries it.
- **Test names carry the scenario**: a comment that restates the test name, or narrates the assertions under it, is noise. When a fixture's shape is the only non-obvious part, one short line on the fixture is the ceiling.

- **Technical accuracy** over metaphors or analogies
- **Concise descriptions**: explain what, not how (the code shows how)
- **No fluff**: avoid words like "simple", "just", "basically"
- **Professional tone**: technical documentation, not conversation
- **Multi-line format** for class/function descriptions, single-line for properties
- **Tapered line widths**: in a wrapped docblock paragraph, each line runs roughly 3 chars shorter than the one above. Resets after a paragraph break.

Also apply the Shared Prose Rules (comments and docs) below.

## Error Messages

Write helpful errors like Laravel. Name the field, the value, and the expectation. Write for the person who hits the error, not the developer who threw it.

- DO "The name field is required"
- DON'T "Missing parameter: name"
- DO "Unable to find user with ID: 42"
- DON'T "User not found"

## Documentation: Stripe-Style Clarity

Write docs that are conversational yet precise:

- **Start with why**: "Send money to your users by creating a payout..."
- **Show, then explain**: Code example first, details after
- **Use "you" and active voice**: "You can retrieve a customer..." not "Customers can be retrieved..."
- **One concept per paragraph**
- **Practical examples** over abstract descriptions

Also apply the Shared Prose Rules (comments and docs) below.

## Landing Changes

- **Make the change easy, then make the easy change**: when a change is hard, refactor first so it becomes trivial (as its own commit or PR), then land it
- **Rebase, don't merge**: update a branch by rebasing onto main, not by merging main into it
- **Stack dependent PRs**: multi-part dependent work ships as stacked GitHub PRs, each branch based on the previous. Independent changes get independent PRs, never stacked
- **Side work gets its own worktree**: anything beyond the task at hand runs in a git worktree with its own branch and PR, never in Franco's active checkout

## PR Descriptions

A reviewer should absorb the body in under a minute. Follow the repo's PR template when one exists. Describe the change in its final shape: never narrate the review process, superseded revisions of the same PR, or designs that did not ship, and never argue with objections nobody raised. Do not read test names back in prose, the test file already carries them. A rejected alternative earns at most one sentence, and only when the next maintainer would otherwise retry it.

## Shared Prose Rules (comments and docs)

These apply to both code comments and documentation.

- **Plain over jargon**: default to concrete behavior. Reach for CS terms (_idempotent_, _invariant_, _sentinel_, _signal_, _monotonic_) only when the term **is the point** (it conveys something plain English can't, and the reader needs that concept to use the code correctly). Test: remove the term. If meaning survives, it was decoration. Examples:
  - `Idempotently sync X` → `Sync X` (idempotency wasn't the point)
  - `This endpoint is idempotent, safe to retry on network failure` → keep (retry logic depends on it)
  - `Empty input is a deliberate detach signal` → `Passing an empty collection detaches every tag` (describe behavior, not framing)
  - "invariant" in particular: prefer "rule" or "guarantee" unless the context is genuinely formal (a spec, math, quoted material)
- **No filler clichés**: avoid "defense in depth", "legacy", and "the long pole" unless the term is load-bearing. Test: remove the phrase. If meaning survives, it was decoration. Examples:
  - `Defense in depth: validate at controller and model` → keep (names the layered-checks pattern that justifies the redundancy)
  - `Defense in depth approach to error handling` → `Validate inputs at every boundary` (rhetorical garnish)
  - `Removed legacy auth middleware (pre-Sanctum, cookie-session)` → keep (parenthetical names the predecessor)
  - `Refactored the legacy user service` → `Refactored UserService` (vibes, not facts)
  - `e2e is the long pole of the pipeline` → `e2e dominates total pipeline wall-clock (deploy waits on it)` (describe the dependency, not the metaphor)
- **No biography**: comment the constraint that holds _now_ in _this file_, not the bug it replaced or the work that prompted the change. When history carries a load-bearing rule, reframe it present-tense. Otherwise cut it. Test: would a reader of this file need it to use the code correctly? If it only recounts how we got here or imports context from another codebase, it is biography. Examples:
  - `Cast to int, the API returned strings and broke the totals` → `The API returns amounts as strings, so cast before summing` (reframe the fixed bug as the constraint that still binds)
  - `halves PHP CPU time, the dominant cost of the test suites` → `halves PHP CPU time` (test-suite framing is origin context from another repo, not a fact about this image)
  - `Rewrote this after the March N+1 incident` → cut (the incident is git history, the code shows the fix)
  - Diff-relative phrasings (`match the pre-FormRequest contract`, `previously Y, now Z`, `restores behavior before N`) → state the rule, constraint, or hidden coupling directly. The comment must make sense to a reader who has no idea which PR added it. That history belongs in the commit message and PR description.
- **No em dash** (—) and no `-` as parenthetical separators. Use parentheses or split into separate sentences.
- **No semicolons** (almost never). Split into separate sentences instead.
- **Straight apostrophes and quotes** (' and ") always, never curly.
