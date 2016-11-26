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


## GC


# Slots

http://book.pythontips.com/en/latest/__slots__magic.html
https://stackoverflow.com/questions/472000/usage-of-slots
http://python-history.blogspot.com.br/2010/06/inside-story-on-new-style-classes.html
http://tech.oyster.com/save-ram-with-python-slots/
http://www.elfsternberg.com/2009/07/06/python-what-the-hell-is-a-slot/

# Data Model

## Attributes and Interfaces

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


* Hashable
* Iterable
* Container: Objects that references other objects

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


# Special Methods

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


## Attribute Access

x.name

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





