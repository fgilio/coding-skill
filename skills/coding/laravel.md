# Laravel: Idiomatic PHP

## Fluent Chains and Typed Array Access

```php
$user->posts()->published()->latest()->take(5)->get();
$query->when($request->has('filter'), fn ($query) => $query->where('status', $request->filter));
Route::get(uri: '/users', action: [UserController::class, 'index']);

// Typed access to untyped arrays (API payloads, cached values, JSON)
$order = fluent($data);
$quantity = $order->integer('quantity');
$placedAt = $order->date('placed_at');

// One-shot: skip the local var when you only read one key
'total' => fluent($data)->integer('amount_cents'),
```

Reach for `fluent()` when you'd otherwise sprinkle `(int) $data['x']`, `$data['y'] ?? null`, or `Carbon::parse($data['z'])` across a method. It gives you `->integer()`, `->string()`, `->boolean()`, `->date()`, `->collect()` with coercion and null-safety built in.

## Framework APIs Over Raw PHP

When Laravel ships an API for the job, reach for it before raw PHP: collections over `foreach`/`array_map`/`array_filter`, `Str`/`Arr` over `preg_*` and manual string ops, `Http` over curl, `Storage` over raw filesystem calls, `throw_if`/`throw_unless` over guard-and-throw blocks, `Rule` objects over string-built validation rules.

A native function that does the whole job in one call (`mb_strtoupper($key)`, `mb_trim($line)`, `mb_substr($text, 0, 80)`) is not a manual string op, so keep it: the `Str` method only wraps it, and a formatter can write these calls itself, as Pint's `mb_str_functions` rule does. The rule targets work spread over several calls or a regex where one helper says it.

```php
// DON'T: guard-and-throw block
if (! $response->successful()) {
    throw new ReportDownloadException(
        "Failed to download report: HTTP {$response->status()}",
        $response->status(),
    );
}

// DO: the class-string form builds the exception only when it throws
throw_unless(
    $response->successful(),
    ReportDownloadException::class,
    "Failed to download report: HTTP {$response->status()}",
    $response->status(),
);

// DO: a Throwable instance is rethrown as-is
throw_if($e instanceof RuntimeException, $e);
```

```php
// DON'T: accumulate through a mutable array
$labels = [];
foreach ($lines as $line) {
    if (trim($line) !== '') {
        $labels[] = trim($line);
    }
}

// DO: the chain states the shape of the result
$labels = collect($lines)
    ->map(fn (string $line): string => trim($line))
    ->filter()
    ->values()
    ->all();
```

```php
// DON'T: build the rule string by hand
'status' => 'required|in:'.implode(',', Order::STATUSES),

// DO: array rules with Rule objects
'status' => ['required', Rule::in(Order::STATUSES)],
```

When the framework already hands you a resolved value, don't reach back to the raw and re-derive it: an Eloquent cast, a Nova field's resolved attribute, a FormRequest's validated data.

```php
// DON'T: reach back to the resource and re-cast in a closure
Text::make('Status', fn () => $this->resource->status->value),

// DO: the field resolves the cast enum; displayUsing gets it
Text::make('Status')->displayUsing(fn (?Status $status) => $status?->value),
```

Idiomatic conversion preserves semantics. The boundaries:

- A loop that exits early stays `foreach` unless a collection method expresses the exit directly (`first()`, `contains()`, `takeUntil()`).
- `each()` discards closure returns. When the results matter, that is `map()`.
- Helpers are not drop-in for the raw construct they resemble: `blank('0')` is false while `empty('0')` is true. Convert when the semantics match, not when the shape does.
