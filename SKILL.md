---
name: coding
description: >
  Franco's code style preferences: expressive naming, fluent interfaces,
  early returns, Laravel-style comments, Stripe-style docs.
  Use when: writing code, reviewing code, refactoring, adding comments,
  writing docs, naming functions/variables, creating error messages.
user-invocable: false
disable-model-invocation: false
---

# Code Style: Expressive & Readable

Write code that feels eloquent and expressive. Repo conventions win on
mechanics (indentation, quotes, import order). These rules win on
structure and prose.

## General Principles
- **Fluent interfaces**: Chain methods naturally
- **Descriptive names**: Full words that read like sentences. Predicates read as questions (is/has/can/should)
- **Early returns**: Reduce nesting
- **Immutability first**: Prefer const/final where possible

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

## PHP/Laravel Examples
```php
$user->posts()->published()->latest()->take(5)->get();
$query->when($request->has('filter'), fn ($query) => $query->where('status', $request->filter));
Route::get(uri: '/users', action: [UserController::class, 'index']);

// Typed access to untyped arrays (API payloads, cache recaps, JSON)
$recap = fluent($data);
$sessions = $recap->integer('sessions');
$startedAt = $recap->date('started_at');

// One-shot: skip the local var when you only read one key
'total' => fluent($data)->integer('seconds_reading'),
```

Reach for `fluent()` when you'd otherwise sprinkle `(int) $data['x']`,
`$data['y'] ?? null`, or `Carbon::parse($data['z'])` across a method.
It gives you `->integer()`, `->string()`, `->boolean()`, `->date()`,
`->collect()` with coercion and null-safety built in.

## Guard Clauses: One Guard Per Exit Reason

When two conditions exit differently (silent return, logged skip,
exception), write two sequential guards. Never merge them into one
umbrella condition and re-branch inside it. A nested `if` inside a
guard body means two guards were collapsed.

```php
// DON'T: umbrella guard, then re-discriminate inside
if (! HmacKey::isStrong($webhookKey)) {
    if ($webhookKey !== null) {
        $this->logSkip($webhookKey);
    }

    return;
}

// DO: each guard states its condition and owns its consequence
if ($webhookKey === null) {
    return;
}

if (! HmacKey::isStrong($webhookKey)) {
    $this->logSkip($webhookKey);

    return;
}
```

This usually creeps in when adding a new exit condition next to an
existing guard. Add it as its own guard instead of widening the
existing condition and patching the difference inside.

## Code Comments: Laravel-Style Precision

Write comments like Taylor Otwell. Technical, concise, and clean:

```php
/**
 * Mutex implementation for serializing concurrent operations.
 *
 * Ensures exclusive access to shared resources by queuing
 * operations and processing them sequentially in turn.
 */
```

Comment density: sparse and deliberate. Docblocks on classes and
functions, single-line notes on properties or a genuinely non-obvious
line. This governs over any "comment every line" default. A why-comment
earns its place by naming a constraint the code can't show, not by
narrating what the next line plainly does.

```php
// DON'T: narrates what the next line plainly does
// Filter to active users and take the newest five
$recent = $users->where('is_active', true)->sortByDesc('created_at')->take(5);
```

- **Technical accuracy** over metaphors or analogies
- **Concise descriptions**: explain what, not how (the code shows how)
- **No fluff**: avoid words like "simple", "just", "basically"
- **Professional tone**: technical documentation, not conversation
- **Multi-line format** for class/function descriptions, single-line for properties
- **Tapered line widths** (Taylor-style PHPDoc): each wrapped line is roughly 3 chars shorter than the one above. Resets after a paragraph/line break.

Also apply the Shared Prose Rules (comments and docs) below.

## Error Messages

Write helpful errors like Laravel. Name the field, the value, and the
expectation. Write for the person who hits the error, not the
developer who threw it.
- DO "The name field is required"
- DON'T "Missing parameter: name"
- DO "Unable to find user with ID: 42"
- DON'T "User not found"


# Documentation: Stripe-Style Clarity

Write docs that are conversational yet precise:

- **Start with why**: "Send money to your users by creating a payout..."
- **Show, then explain**: Code example first, details after
- **Use "you" and active voice**: "You can retrieve a customer..." not "Customers can be retrieved..."
- **One concept per paragraph**
- **Practical examples** over abstract descriptions

Also apply the Shared Prose Rules (comments and docs) below.


# Shared Prose Rules (comments and docs)

These apply to both code comments and documentation.

- **Plain over jargon**: default to concrete behavior. Reach for CS terms (*idempotent*, *invariant*, *sentinel*, *signal*, *monotonic*) only when the term **is the point** (it conveys something plain English can't, and the reader needs that concept to use the code correctly). Test: remove the term. If meaning survives, it was decoration. Examples:
    - `Idempotently sync X` → `Sync X` (idempotency wasn't the point)
    - `This endpoint is idempotent, safe to retry on network failure` → keep (retry logic depends on it)
    - `Empty input is a deliberate detach signal` → `Passing an empty collection detaches every term` (describe behavior, not framing)
    - "invariant" in particular: prefer "rule" or "guarantee" unless the context is genuinely formal (a spec, math, quoted material)
- **No filler clichés**: avoid "defense in depth", "legacy", and "the long pole" unless the term is load-bearing. Test: remove the phrase. If meaning survives, it was decoration. Examples:
    - `Defense in depth: validate at controller and model` → keep (names the layered-checks pattern that justifies the redundancy)
    - `Defense in depth approach to error handling` → `Validate inputs at every boundary` (rhetorical garnish)
    - `Removed legacy auth middleware (pre-Sanctum, cookie-session)` → keep (parenthetical names the predecessor)
    - `Refactored the legacy user service` → `Refactored UserService` (vibes, not facts)
    - `e2e is the long pole of the pipeline` → `e2e dominates total pipeline wall-clock (deploy waits on it via needs:)` (describe the dependency, not the metaphor)
- **No biography**: comment the constraint that holds *now* in *this file*, not the bug it replaced or the work that prompted the change. When history carries a load-bearing rule, reframe it present-tense; otherwise cut it. Test: would a reader of this file need it to use the code correctly? If it only recounts how we got here or imports context from another codebase, it is biography. Examples:
    - `` `platform: linux/arm64`; the amd64 base silently defeated that and ran under Rosetta `` → `the platform declaration won't force arm64 on its own, so the base must be the arm tag` (reframe the fixed bug as the constraint that still binds)
    - `halves PHP CPU time, the dominant cost of the test suites` → `halves PHP CPU time` (test-suite framing is origin context from another repo, not a fact about this image)
    - `Rewrote this after the March N+1 incident` → cut (the incident is git history, the code shows the fix)
    - Diff-relative phrasings (`match the pre-FormRequest contract`, `previously Y, now Z`, `restores behavior before N`) → state the rule, constraint, or hidden coupling directly. The comment must make sense to a reader who has no idea which PR added it. That history belongs in the commit message and PR description.
- **No em dash** (—) and no ` - ` as parenthetical separators. Use parentheses or split into separate sentences.
- **No semicolons** (almost never). Split into separate sentences instead.
- **Straight apostrophes and quotes** (' and ") always, never curly.
