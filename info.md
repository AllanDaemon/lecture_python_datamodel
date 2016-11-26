# Hash*

# Imgs
https://en.wikipedia.org/wiki/File:Uniform_discrete_pmf_svg.svg
http://www.panix.com/~murphy/bday.html
https://commons.wikimedia.org/wiki/File:Singly-linked-list.svg
https://en.wikipedia.org/wiki/File:Hash_table_3_1_1_0_1_0_0_SP.svg
https://en.wikipedia.org/wiki/File:Hash_table_5_0_1_1_1_1_1_LL.svg
https://en.wikipedia.org/wiki/File:Hash_table_5_0_1_1_1_1_0_LL.svg
https://commons.wikimedia.org/wiki/File:Hash_table_simple_999.svg

http://slideplayer.com/slide/9354004/
http://images.slideplayer.com/28/9354004/slides/slide_12.jpg

http://web.cs.ucla.edu/~forns/classes/winter-2014/cs-32/week-10.html

--
https://www.smartdraw.com/object-diagram/examples/object-diagram-employment-chart/w


# Objects

```python
class ClassName(object):

	def __init__(self, arg):
		self.arg = arg

	def __del__(self):
		print "good bye"

	def method_name(self):
		pass

	def method_with_arg(self, arg1, arg2):
		print arg1, arg2

o = ClassName(42)
print o.arg  #42
o.method_with_arg("Hi", "There")
del o
```

```python
>>> class ClassName(object):
...     def __init__(self, arg):
...             self.arg = arg
...     def __del__(self):
...             print "good bye"
...     def method_name(self):
...             pass
...     def method_with_arg(self, arg1, arg2):
...             print arg1, arg2
... 
>>> o = ClassName(42)
>>> print o.arg
42
>>> o.method_with_arg("Hi", "There")
Hi There
>>> del o
good bye

```


## CPython implementation


* cpython/Include/Python.h
* cpython/Include/object.h:77-116
* cpython/Include/typestruct.h:


```c
/* PyObject_HEAD defines the initial segment of every PyObject. */
#define PyObject_HEAD                   \
	Py_ssize_t ob_refcnt;               \
	struct _typeobject *ob_type;


#define PyObject_HEAD_INIT(type)        \
	1, type,

#define PyVarObject_HEAD_INIT(type, size)       \
	PyObject_HEAD_INIT(type) size,

#define PyObject_VAR_HEAD               \
	PyObject_HEAD                       \
	Py_ssize_t ob_size; /* Number of items in variable part */

typedef struct _object {
	PyObject_HEAD
} PyObject;

typedef struct {
	PyObject_VAR_HEAD
} PyVarObject;

#define Py_REFCNT(ob)           (((PyObject*)(ob))->ob_refcnt)
#define Py_TYPE(ob)             (((PyObject*)(ob))->ob_type)
#define Py_SIZE(ob)             (((PyVarObject*)(ob))->ob_size)
```

```c
typedef struct _object {
	i64 ob_refcnt;
	struct _typeobject *ob_type;
} PyObject;

typedef struct {
	i64 ob_refcnt;
	struct _typeobject *ob_type;
	i64 ob_size; /* Number of items in variable part */
} PyVarObject;

```
## References

https://www.safaribooksonline.com/library/view/learning-python-5th/9781449355722/ch06s02.html

## GC





# Slots

http://book.pythontips.com/en/latest/__slots__magic.html
https://stackoverflow.com/questions/472000/usage-of-slots
http://python-history.blogspot.com.br/2010/06/inside-story-on-new-style-classes.html
http://tech.oyster.com/save-ram-with-python-slots/
http://www.elfsternberg.com/2009/07/06/python-what-the-hell-is-a-slot/





# Data Model

## Attributes and Interfaces

https://docs.python.org/2/glossary.html
https://github.com/fluentpython

### (Im)Mutable:
	* **Immutable** may be reused (final on 3.1): a=1; b=1; # May be the same
	* **Mutable**: Always different instances: a=[]; b=[]; # a is not b
	but a = b = [] is

### Callable
	`callable(object)`

	`cpython/Objects/object.c:1655`

	```c
	/* Test whether an object can be called */

	int
	PyCallable_Check(PyObject *x)
	{
		if (x == NULL)
			return 0;
		if (PyInstance_Check(x)) {
			PyObject *call = PyObject_GetAttrString(x, "__call__");
			if (call == NULL) {
				PyErr_Clear();
				return 0;
			}
			/* Could test recursively but don't, for fear of endless
			   recursion if some joker sets self.__call__ = self */
			Py_DECREF(call);
			return 1;
		}
		else {
			return x->ob_type->tp_call != NULL;
		}
	}
	```

	```python
	def callable(obj):
		return obj and hasattr(obj, '__call__')
	```


### Hashable
* Immutable
* `__hash__()`
* `__eq__()` or `__cmp__()`
* object.__hash__ == id


### Iterable
* `__iter__()` or `__getitem__()`
* `iter()`
* `next(iter)`
* `StopIteration`

### Descriptor
* `__get__()`
* `__set__()` 
* `__delete__()`

`class property([fget[, fset[, fdel[, doc]]]])`

```python
class C(object):
    def __init__(self):
        self._x = None

    def getx(self):
        print "get x"
        return self._x

    def setx(self, value):
    	print "set x", value
        self._x = value

    def delx(self):
        print "del x"
        del self._x

    x = property(getx, setx, delx, "I'm the 'x' property.")

class C(object):
    def __init__(self):
        self._x = None

    @property
    def x(self):
        """I'm the 'x' property."""
        print "get x"
        return self._x

    @x.setter
    def x(self, value):
    	print "set x", value
        self._x = value

    @x.deleter
    def x(self):
        print "del x"
        del self._x


c = C()
c.x
c.x = 10
c.x
del c.x
c.x
c.x = 20
c.x

>>> c = C()
>>> c.x
get x
>>> c.x = 10
set x 10
>>> c.x
get x
10
>>> del c.x
del x
>>> c.x
get x
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 9, in x
AttributeError: 'C' object has no attribute '_x'
>>> c.x = 20
set x 20
>>> c.x
get x
20

```


### Container

Objects that references other objects

#### Sequences
* `__getitem__()`
* `__len__()`

* tuple
* list
* str
* unicode
* bytes



#### Mapping
lookups use arbitrary immutable keys rather than integers

* dict
* collections.defaultdict
* collections.OrderedDict
* collections.Counter



### ABC & Others

* https://docs.python.org/2/library/collections.html
* https://docs.python.org/3.7/library/collections.abc.html

* https://docs.python.org/2/library/collections.html#collections-abstract-base-classes
* https://docs.python.org/3.7/library/collections.abc.html#collections-abstract-base-classes

* file: `read()` and `write()`
* byte arrays
* etc

## Type Hierarchy

* None
* NotImplemented
* Ellipsis

* numbers.Number
	* numbers.Integral
		* int
		* long
		* bool
	* numbers.Real: float
	* numbers.Complex: complex

* Sequences
	* Immutable
		* String: str | bytes
		* Unicode: unicode | str
		* tuple
		
	* Mutable
		* list
		* bytearray
		* array (module)

* Sets
	* set
	* frozenset

* Mappings
	* dict

* Callable
	* User-defined function
	* User-defined methods | Instance Methods
	* Generator functions
	* | Coroutine functions
	* Built-in functions
	* Built-in methods
	* Class types
	* Classic Classes |
	* Class instances

* Modules

* Classes | Custom Classes
* Class Intances

* Files | IO Objects

* Internal Types
	* Code objects
	* Frame Objects
	* Traceback Objects
	* Slice Objects
	* Static method objects
	* Class method objects

## New classes vs Old classes


## Special Method Names

__specialnames__

# Special Attributes

https://docs.python.org/2/library/stdtypes.html#special-attributes
https://docs.python.org/3.7/library/stdtypes.html#special-attributes

## object.__dict__
A dictionary or other mapping object used to store an object’s (writable) attributes.

`vars(obj)`
`dir(obj)`

## instance.__class__
The class to which a class instance belongs.

## class.__bases__
The tuple of base classes of a class object.

## definition.__name__
The name of the class, function, method, descriptor, or generator instance.

## definition.__qualname__
The qualified name of the class, function, method, descriptor, or generator instance.

New in version 3.3.

## class.__mro__
This attribute is a tuple of classes that are considered when looking for base classes during method resolution.

## class.mro()
This method can be overridden by a metaclass to customize the method resolution order for its instances. It is called at class instantiation, and its result is stored in __mro__.

## class.__subclasses__()
Each class keeps a list of weak references to its immediate subclasses. This method returns a list of all those references still alive. Example:

`>>> int.__subclasses__()`
`[<class 'bool'>]`

## object.__metaclass__

__prepare__


## docstring

object.__doc__

`help()`

## callable

#### __doc__
Writable
The function’s documentation string, or None if unavailable; not inherited by subclasses

#### __name__
Writable
The function’s name

#### __qualname__
Writable
The function’s qualified name

New in version 3.3.

#### __module__
Writable
The name of the module the function was defined in, or None if unavailable.

#### __defaults__
Writable
A tuple containing default argument values for those arguments that have defaults, or None if no arguments have a default value

#### __code__
Writable
The code object representing the compiled function body.

#### __globals__
Read-only
A reference to the dictionary that holds the function’s global variables — the global namespace of the module in which the function was defined.

#### __dict__
Writable
The namespace supporting arbitrary function attributes.

#### __closure__
Read-only
None or a tuple of cells that contain bindings for the function’s free variables.

#### __annotations__
Writable
A dict containing annotations of parameters. The keys of the dict are the parameter names, and 'return' for the return annotation, if provided.

#### __kwdefaults__
Writable
A dict containing defaults for keyword-only parameters.


# Special Methods

in class `__dict__`
not int instance `__dict__`.

## Basic

### new
object.__new__(cls[, ...])

### init
object.__init__(self[, ...])

### del
object.__del__(self)

### repr
object.__repr__(self)

### str unicode bytes format
object.__str__(self)
object.__unicode__(self)
object.__bytes__(self)
object.__format__(self, format_spec)	# py3

### comparations
object.__lt__(self, other)
object.__le__(self, other)
object.__eq__(self, other)
object.__ne__(self, other)
object.__gt__(self, other)
object.__ge__(self, other)

object.__cmp__(self, other)		# py2	# C cmp semmantics

### bool
object.__nonzero__(self)	# py2
object.__bool__(self)		# py3

### hash
object.__hash__(self)

https://docs.python.org/3.7/library/stdtypes.html#hashing-of-numeric-types


## Attribute Access

`x.name`

object.__getattr__(self, name)		# Not always called (just when not found in ...)
object.__getattribute__(self, name)		# New style class. Always called
object.__setattr__(self, name, value)
object.__delattr__(self, name)
object.__dir__(self)

https://docs.python.org/2/reference/datamodel.html#new-style-special-lookup

### Descriptors
object.__get__(self, instance, owner)
object.__set__(self, instance, value)
object.__delete__(self, instance)
object.__set_name__(self, owner, name)		# py3.6

*** PYTHON WAY OF LOOKUP *** (invoking descriptors)

### __slots__


## Class Creation

### __metaclass__
### __prepare__		# py3
### __class__

### Checking
class.__instancecheck__(self, instance)
class.__subclasscheck__(self, subclass)

isinstance(object, classinfo)
issubclass(class, classinfo)


## Callable Objects
object.__call__(self[, args...])
x(arg1, arg2, ...) is a shorthand for x.__call__(arg1, arg2, ...)


## Container Types

object.__len__(self)
object.__getitem__(self, key)	# IndexError, KeyError
object.__missing__(self, key)
object.__setitem__(self, key, value)
object.__delitem__(self, key)
object.__iter__(self)
object.__reversed__(self)
object.__contains__(self, item)

object.__length_hint__(self) 	# py3.4

* Note: getslice, setslice, delslice deprecaed since py2.0


## Numeric


### Normal

object.__add__(self, other)
object.__sub__(self, other)
object.__mul__(self, other)
object.__matmul__(self, other)		# py3.5
object.__div__(self, other)			# py2: /
object.__truediv__(self, other)		# py2: future, py3: /
object.__floordiv__(self, other)	# //
object.__mod__(self, other)
object.__divmod__(self, other)
object.__pow__(self, other[, modulo])
object.__lshift__(self, other)
object.__rshift__(self, other)
object.__and__(self, other)
object.__xor__(self, other)
object.__or__(self, other)

### Reflected
object.__radd__(self, other)
object.__rsub__(self, other)
object.__rmul__(self, other)
object.__rmatmul__(self, other)
object.__rdiv__(self, other)
object.__rtruediv__(self, other)
object.__rfloordiv__(self, other)
object.__rmod__(self, other)
object.__rdivmod__(self, other)
object.__rpow__(self, other)
object.__rlshift__(self, other)
object.__rrshift__(self, other)
object.__rand__(self, other)
object.__rxor__(self, other)
object.__ror__(self, other)

### In-place
object.__iadd__(self, other)
object.__isub__(self, other)
object.__imul__(self, other)
object.__imatmul__(self, other)
object.__itruediv__(self, other)
object.__ifloordiv__(self, other)
object.__imod__(self, other)
object.__ipow__(self, other[, modulo])
object.__ilshift__(self, other)
object.__irshift__(self, other)
object.__iand__(self, other)
object.__ixor__(self, other)
object.__ior__(self, other)

### Unary
object.__neg__(self)
object.__pos__(self)
object.__abs__(self)
object.__invert__(self)

### Builtins
object.__complex__(self)
object.__int__(self)
object.__long__(self)	# py2
object.__float__(self)
object.__round__(self[, n])	# py3?

### Convertion
object.__oct__(self)	# 2
object.__hex__(self)	# 2
object.__coerce__(self, other)

#### Coercion Rules
No. Really.

### Index
object.__index__(self)

## With Statement Context Managers
object.__enter__(self)
object.__exit__(self, exc_type, exc_value, traceback)

## Py 3 Async
### Coroutines
object.__await__(self)

### Asynchronous Iterators
object.__aiter__(self)
object.__anext__(self)

### Asynchronous Context Managers
object.__aenter__(self)
object.__aexit__(self, exc_type, exc_value, traceback)

