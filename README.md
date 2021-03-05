# PHP RFC: Optionally capture exceptions as second return values
  * Version: 0.1
  * Date: 2020-03-05
  * Author: Mike Schinkel, <mike@newclarity.net>
  * Status: ROUGH Draft

## Introduction

_"Error"_ handling has evolved over PHP's lifetime, from:

1. Returning flag values such as `false` when calling `strpos()` where the substring is not found, 
2. Outputting diagnostic messages such a `E_WARNING` or `E_NOTICE` to,
3. Throwing Exceptions that need to be caught. And PHP is still littered with this collection of inconsistency which most PHP developers who have expressed an oppinion seem to be interested in seeing be made more consistent.

In general the majority of influential PHP developers seem intereste in seeing Exceptions overtaking most if not all error handling in PHP whereas a non-insignificant contingent would like to see more GoLang-style immediate handling of errors be enabled. Ideally both approaches would be supported if it were easy and elegant to do so.

And one issue to always be sure of is that whatever changes need to support backward compatibility whenever possible, and where not to provide as smooth as transition as possible.

## Proposal

Allow the addition of an optional second variable on the lefthand-side of an `=` assignment and when used that it would capture any exception that might be thrown instead of requiring a `try {} catch {}`.

### Optionally capturing Exceptions as secondary return values

So instead of always having to write code like the following when a function potentially throws an exception:

```php
function GetValueButMaybeThrowException() {
    $date = date('m/d/Y');
    if ( intval( date('H') ) >= 18 ) { 
        throw new Exception('Sorry, it is too late in the day now');
    return $date
}
try {
    $value = GetValueButMaybeThrowException()
} catch ($e Exception) {
    echo $e->getMessage()
    return
}
echo $value
```

This would be an optional approach that would not throw the exception but instead assign it to the variable after the comma on the left hand side. Or in the example's case, the exception would be assigned to `$e`:

```php
$value, $e = GetValueButMaybeThrowException()
if !is_null($e) {
   echo $e->getMessage()
   return
}
echo $value
```
### Optionally ignoring Exceptions

And if you do not care about the exception but want to keep it from being thrown you could do either of these:

```php
$value, void = GetValueButMaybeThrowException()
echo $value
```

This assumes George P. Banyard's RFC[[1]](https://github.com/Girgias/error-control-operator-exceptions-rfc) to extend error control operator to suppress exceptions.

```php
echo @GetValueButMaybeThrowException()
```

### Explicitly returning Exceptions
In addition this proposal suggests a developer should be able to write a function that explicitly returns both a value and an exception — where the exception can be ignored by the caller — and in this case the function never throws an exception.

```php
function GetValueButMaybeReturnException() {
    $date = date('m/d/Y');
    $e = intval( date('H') ) >= 18 
        ? new Exception('Sorry, it is too late in the day now')
        : null;
    return $date, $e
}
```
### Type-hinting when Exceptions are used
In either the throw or the return case, type hinting could be handled by using a comma to separate the two types, e.g.

```
function GetValueButMaybeThrowException(): string,Exception? {...}
function GetValueButMaybeReturnException(): string,Exception? {...}
```


## Prior Art

This approach is similar but not identical to Go-style[[2]](https://blog.golang.org/error-handling-and-go) returns.

Go functions can return one or more values to the caller, and for the use-case a function needs to return an error object it should be the last return value, but only _**by convention**_.

Further in Go, _**all**_ values returned by a function need to be accepted by the caller.

### Deviation from Prior Art
In this proposal however HP would not be supporting multiple return values, _**per se**_.

Instead PHP would be supporting one (1) return value, as it currently does, and one (1) optional Exception object, or `null`.

In addition, the optional Exception object would _**only optionally**_ need to be captured into a variable. It no second variable were assigned then the function would throw the Exception as it always has in the past.

This takes the best part of Go — being able to return an error flag from a function in addition to the value — but leaves behind the overuse of returning multiple parameters that you sometimes see in Go and provides structure for multiple parameters that Go does not have.

In PHP, the first return value would be a return value whereas the second would be an optional flag that indicate an issue occurred if it is not `null`.

## Rationale

Since most influential developers in the PHP world appear to want to move to exceptions, this approach would allow for a move to exceptions for all functions, even ones that currently return a flag value or output `E_WARNING` or `E_NOTICE` diagnostic messages.

And it would still allow developers who want to handle errors immediately do so in a more streamlined manner.

This would have a backward compatible syntax and leverage the existing `Exception` classes and their related concepts. IOW, no new concept of an `Error` class would be required.

However, changes to existing functions in PHP would not be possible in a backward compatible manner without George P. Banyard's `throw_on_error` RFC[[3]](https://github.com/Girgias/php-rfc-throw-on-error-declare). Ideally that RFC would accompany this one in being implemented.

## Backward Incompatible Changes

No envisioned BC yet.

## Proposed PHP Version

Next minor version, i.e. PHP 8.1.

## RFC Impact 

### To Opcache

TBD

## Open Issues

None currently.

## Future Scope

No envisioned future scope yet.

## Proposed Voting Choices

As per the voting RFC a yes/no vote with a 2/3 majority is needed for this proposal to be accepted.

## Patches and Tests

No patches yet.

### Current implementation detail

No implementation yet.

#### Known issues

No known issues yet.

## Implementation 
After the project is implemented, this section should contain
  - the version(s) it was merged into
  - a link to the git commit(s)
  - a link to the PHP manual entry for the feature
  - a link to the language specification section (if any)

## References

[1]:
[2]:
[3]:
[4]:


## Rejected Features 
Keep this updated with features that were discussed on the mail lists.
