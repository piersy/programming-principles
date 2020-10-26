# Overfitting tests

I'm borrowing the term 'Overfitting' that is traditionally used in statistics
but seems to fit quite nicely with the issue I am about to explain.

In statistics overfitting is described as ["the production of an analysis that
corresponds too closely or exactly to a particular set of data, and may
therefore fail to fit additional data or predict future observations
reliably"](https://en.oxforddictionaries.com/definition/overfitting)

I will use overfitting here to describe tests that assert more than is
required to verify the correct functioning of the object under test.

Example of overfitting:
```go
func TestBarFailsWhenInputIsLessThanZero(t *testing.T){
	f := NewFoo()	
	err := f.Bar(-1)
	assert.Equal(t, LessThanZeroError, err)
}
```
or:
```go

var LessThanZeroError = errors.New("input was less than zero")

func TestBarFailsWhenInputIsLessThanZero(t *testing.T){
	f := NewFoo()	
	err := f.Bar(-1)
	assert.Contains(t, err.String(), "less than zero")
}
```

Why are these cases of overfitting? We are checking for a specific error or
error string, which is unnecessary to ascertain whether or not `Bar` failed. To
know if `Bar` failed all we need to know is that it returned a non nil error.

Often people may say something along the lines of "but this is how we know we
are hitting all the edge cases in our tests", this however is a fallacy,
because you can only rely on hitting all the edge cases in your tests if you
know all your code and tests are correct, and if that was the case there would
be no need for testing.

If you want to know that you are hitting all the edge cases in your tests use a
tool to measure coverage and check to see that all paths are covered.

# Problems with overfitting in tests

### Brittle tests

Overfitting makes tests brittle and consequently leads to increased overheads
to changing code. In the above examples re-wording errors can lead to test
failures and also changing the type of error returned can lead to test
failures. All of which can be avoided by simply not checking for specific errors
and error strings.

### Code bloat

Sometimes extra error types or variables are added purely for the purpose of
testing. This adds to code bloat

# Resolution

Think carefully about the purpose of your tests and ensure that you only check
just what is required to ascertain success.

# Exceptions

There aren't any exceptions.

# Real life examples

[Checking error strings
](https://github.com/ethereum/go-ethereum/blob/1a55e20d3562b08a4cf31b5b6b66d39d9acc43df/p2p/enode/urlv4_test.go#L40) 
[Checking error
instances](https://github.com/ethereum/go-ethereum/blob/1a55e20d3562b08a4cf31b5b6b66d39d9acc43df/core/genesis_test.go#L61).
In this case the test framework mandates checking a specific error. This will
lead to creation of errors just to satisfy the test framework.  
