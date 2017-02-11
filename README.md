# Type Fu
Author: Darren M. Struthers - <dstruthers@gmail.com>

This is a collection of utilities for manipulating and transforming Python
classes. Currently, the library consists of the following:

* [Derived Types](#derived-types)
* [Mimic Type](#mimic-type)

Much of the functionality provided by this module may be of particular use to
library authors.

## Derived Types

Derived types, for lack of a standardized or better term,  provide behavior
that is similar to class inheritance, albeit with some key deviations. Most
notably, if a derived type's inherited method returns a value of the base
class's type, the derived type's method will attempt to cast the value to
an instance of the derived type, rather than leave it as an instance of the
base class.

The following example illustrates this distinction:

```python
>>> from typefu import derived
>>> class MyStr(str): pass # inherit from str directly
...
>>> class MyStr2(derived(str)): pass # use a derived instance instead
...
>>> foo = MyStr('foo')
>>> foo.upper()
'FOO'
>>> type(foo.upper())
<class 'str'>
>>> bar = MyStr2('bar')
>>> bar.upper()
'BAR'
>>> type(bar.upper())
<class '__main__.MyStr2'>
```

Another important distinction is that derived types are not class descendents of
the types from which they are derived.

```python
>>> isinstance(foo, str)
True
>>> isinstance(bar, str)
False
```
### Why use Derived Types?

Continuing from the above example, consider situations like this:

```python
>>> type(foo)
<class '__main__.MyStr'>
>>>foo += 'baz'
>>> type(foo)
<class 'str'>
```
The fallback to the base class might be harmless in some cases, but problematic
in others. It's not difficult to imagine the inherited class containing
additional data that would be lost in this situation.

A single programmer can obviously account for, and work around, this situation,
but if a programmer is writing a class for a library or for others to use, it
may be advantageous to provide some protections in certain situations. A derived
type may be able to provide that protection.

```python
>>> type(bar)
<class '__main__.MyStr2'>
>>> bar += 'baz'
>>> type(bar)
<class '__main__.MyStr2'>
```

### Accessing the Underlying Value

To access the underlying value, use the `value` property.

```python
>>> bar.value
'barbaz'
>>> type(bar.value)
<class 'str'>
```

## Mimic Type

The `Mimic` type assumes the properties of the object passed to it during
initialization. A `Mimic` is recognizable as an instance of `Mimic` (or a
subclass) as well as an instance of the class of the argument it was passed
during initialization.

```python
>>> from typefu import Mimic
>>> class Arbitrary(Mimic): pass
...
>>> foo = Arbitrary(['foo', 'bar', 'baz'])
>>> foo
['foo', 'bar', 'baz']
>>> isinstance(foo, list)
True
>>> isinstance(foo, Arbitrary)
True
>>> len(foo)
3
```

A `Mimic` might be useful in such a case where you want the result of some
function to be an instance of an expected type, even when you otherwise
might not be able to predict what the type will be. Pardon this contrived
example using `eval`:

```python
from typefu import Mimic

class Untrusted(Mimic): pass

def eval_and_tag(code):
    return Untrusted(eval(code))
```
