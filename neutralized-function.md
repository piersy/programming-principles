# Neutralized functions

A neutralized function is a function that in certain cases does nothing.
Neutralized functions are a special case of violations of the "Single
responsibility principle".

Example neutralized function

```go
// build builds bar from foo.
func build(f *foo) (*bar, error) {
	if f == nil {
		return nil, nil
	}
	...
```

There are several problems here.

## Promotes programmng errors

If f is nil, as we can see from the implementation of `build`, calling `build`
doesn't do any work, so at the best calling `build` was a waste of resources.

But in fact there is never any good reason to call `build` with nil because we
know we will get a nil back, and that is not helpful, so in fact calling
`build` with nil is a programming error. 

## Obscures the problem source

By returning nil here instead of alerting the caller to their mistake we are
effectively delaying the point at which the caller will discover their error.

Consider this use of `build` in a method on a struct `str` referenced by `s`
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

# Harms readability

Consider this use of `build`
```go
b, err := build(f)
...
```

To the reader it looks as if work is being done in build, without digging one
layer deeper the reader cannot know that maybe nothing is happening in `build`.

Now Consider this use of `build`
```go
if f != nil {
	b, err := build(f)
	...
}

```

It is immediately obvious to the reader that if f is nil, we do not call
`build`. Which allows them to simply eradicate `build` from their mental
context when considering flows where f is nil. No need to apply mental energy
to understanding what build does, how it behaves in different contexts and what
side effects it might have.

## Violates SRP

`build` also violates the "single responsibility principle" because it either
does nothing and returns nil or builds bar. Having a function that behaves
completely differently depending on its parameters invites misuse. Callers of
the function are liable to forget to add the check for the nil (the fact that
nil is returned is not in the function documentation), this will lead to their
program crashing when they try to access the value. As we saw above the access
may not happen "close" to the call to `build`, so when the crash occurs, the
user may be left without a clue as to how the problem arose.


## Resolution

This problem is easy to resolve: by simply removing the check for a nil the
function will panic when called incorrectly.
