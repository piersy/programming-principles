# Neutralized functions

A neutralized function is a function that in certain cases does nothing.
Neutralized functions are a special case of violations of the "Single
responsibility principle".

Example neutralized function with return value:

```go
// build builds bar from foo.
func build(f *foo) (*bar, error) {
	if f == nil {
		return nil, nil
	}
	...
```

Example neutralized function without return value:
```go
// do changes some state based on foo.
func do(f *foo) error {
	if f == nil {
		return nil
	}
	...
```

The condition in a neutralized function does not need to be on a function
parameter it could also be a check on some global state, which is arguably
worse, since because the global state is accessible everywhere it's hard to pin
down what the value will be at runtime.
E.g:
```go
// build builds bar from foo.
func build(f *foo) (*bar, error) {
	if !shouldBuild {
		return nil, nil
	}
```

# Problems with neutralized functions

### Promotes programming errors

If `f` is nil, as we can see from the implementation of `build` and `do`, calling
`build` or `do` doesn't result in any work, so at the best calling them was
a waste of resources.

But in fact there is never any good reason to call `build` or `do` with nil
because nothing useful happens, either we  get a nil returned from `build` or
nothing happens in the case of `do`. So in fact calling `build` or `do` with
nil is a programming error. 

### Obscures the problem source

In the case of `build`, by returning nil when passed nil, instead of alerting
the caller to their mistake we are effectively delaying the point at which the
caller will discover their error.

Consider this use of `build` in a method on a struct `str` referenced by `s`:
```go
b, err := build(f)
if err != nil {
	return err
}
s.store[time.Now().String()] = b
```

And then, imagine a later call ...
```go
func (s *str) logBars() {
	for k, v := range s.store {
		println(k,v.String())
	}
}
```

If at some point `build` was called with nil `logBars` will crash when
subsequently called, but at this point we have lost the context showing how nil
was passed to `build` and hence lost any hope of easily tracking down the true
source of the crash.

Note: I could call this smell problem propagation ( the returning the nil )

In the case of `do` by taking no action when passed nil we are also delaying
the point at which the caller will discover their mistake.

Consider this use of `do`:
```go
err := do(f)
if err != nil {
	return err
}
```

Looking at it it would be reasonable to expect that `do` updated the relevant
state. If however `do` was unwittingly called with nil, subsequent operations
that rely on the state changes that `do` was meant to enact will fail in
mysterious ways. And again we have lost the context that would lead us to the
source of the problem.

### Harms readability

Consider this use of `build`:
```go
b, err := build(f)
...
```

To the reader it looks as if work is being done in build, without digging one
layer deeper the reader cannot know that maybe nothing is happening in `build`.

Now Consider this use of `build`:
```go
if f != nil {
	b, err := build(f)
	...
}

```

It is immediately obvious to the reader that if `f` is nil, we do not call
`build`. Which allows them to simply eradicate `build` from their mental
context when considering flows where `f` is nil. No need to apply mental energy
to understanding what build does, how it behaves in different contexts and what
side effects it might have.

The same reasoning can be applied to `do`.

### Violates SRP

`build` and `do` also violate the "single responsibility principle" because
they either execute their intended action or do nothing. Having a function that
behaves completely differently depending on its parameters invites misuse.

Callers of `build` are liable to forget to add the check for the returned nil (the
fact that nil is returned is not in the function documentation), this will lead
to their program crashing when they try to access the value. As we saw above
the access may not happen "close" to the call to `build`, so when the crash
occurs, the user may be left without a clue as to how the problem arose.

Callers of `do` have no easy way to detect if `do` actually did anything;
because it doesn't return a value. Making it easy to assume that `do` took
action when in fact it did not.

# Resolution

This problem is easy to resolve: by simply removing the check for a nil the
functions will panic when called incorrectly. In the case that nil input values
are a possibility this forces the caller to put a check around the function at
a higher level which aids readability and allows the caller to be flexible
about how they handle those situations.

# Exceptions

In the case of neutralized functions that return a value there are no
exceptions, building a nil value is pointless.

In the case of neutralized functions that do not return a value, if they are
called in many places, which causes the outer check for nil to be repeated in
those many places. It can make sense to move the check inside the function.  In
this case the function documentation should make it very clear that the passing
of invalid or nil values results in a noop.
