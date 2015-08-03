# purescript-safely

A combinator for making any monadic control operator stack-safe (using `MonadRec`).

For example, `purescript-lists` defines `replicateM`, which is not stack-safe. It gives a stack overflow when applied to large inputs using, for example, the `Eff` monad.

```text
> replicateM 100000 (log "Testing...")

RangeError: Maximum call stack size exceeded
```

However, we can make it stack-safe by using the `safely` combinator:

```text
replicateS :: forall m a. (MonadRec m) => Int -> m a -> m (List a)
replicateS n m = safely \up down -> down (replicateM n (up m))
```

`safely` provides us two natural transformations: one to lift our `MonadRec` into a full `Monad`, and one to lower it back down again.

We lift the action we want to `replicate(M)` with `up`, and afterwards, lower it again with `down`. The result is a stack-safe function which works with any `MonadRec`.

```text
> replicateM 100000 (log "Testing...")

Testing...
Testing...
Testing...
...
unit
```

We can apply `safely` to lift all sorts of control operators like `iterateM`, `traverse`, `filterM`, etc.
