# Defensive programming

Defensive programming is the act of checking and putting right bad input values.

Example of defensive programming:
```go
func setHourlyWage(dollars int){
	if dollars == 0 {
		dollars = 1
	}	
...
}
```

Defensive programming is actually a violation of the [fail
fast](https://www.martinfowler.com/ieeeSoftware/failFast.pdf) principle. What
defensive programming does is allow a program to continue executing even after
we have encountered an error.

# Problems with defensive programming

### Unexpected behaviour

If we consider the example above, calling `setHourlyWage` with a value of 0 is
clearly a mistake, this is why the value is then updated to 1. It's likely that
in this case the caller forgot to set the value correctly, but instead of
alerting them to the failure, the defensive code silently changes the user
input, and if we imagine that this is a payroll system, that change could be
quite catastrophic for the person being paid.

Silently changing the user input will lead to unexpected behaviour and
therefore bugs, especially when we consider that the defensive code can be
hidden many levels deep.

### Harms readability

Because defensive programming converts certain values to other values "under
the hood" it allows there to be multiple ways to perform the same action. 

Consider these 2 calls to `setHourlyWage` that result in the same outcome:
```go
setHourlyWage(0)
setHourlyWage(1)
```

Allowing multiple differing ways of achieving the same thing harms readability
and understanding, since readers of this code will naturally assume that the
two calls to `setHourlyWage` have different effects.

### Duplication

Because defensive programming converts certain values to other values "under
the hood" it allows there to be multiple ways to perform the same action. 

This can lead to maintaining multiple code paths that are mistakenly believed
to be different, when in fact they both have the same effect, so one could be
removed.

### Resolution

Enforce valid values at the boundaries to the system and fail fast if invalid
values are encountered in the system.

Values either enter a system as input from some outside source or are defined
in the code. Invalid values defined in the code are programming errors and so
we should crash when those are encountered. That leaves the values that are
inputs to the system, these inputs should be validated at the first opportunity
and rejected unless correct. This allows all other code within the system to
know that they will receive valid values, which negates the need for any
defensive programming. 


# Exceptions

There are no exceptions, don't do it.
