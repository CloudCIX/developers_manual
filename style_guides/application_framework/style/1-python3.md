# Python3 Style Guide

Trying to write down a definitive style guide for Python3 code so that it's not just in my head.

Will require a lot of revisions as we go through the creation of Membership in Python3, but I can at least write down what we have so far.

## Table of Contents
- [Linting](#linting)
    - [Current Packages](#current-packages)
- [Rules](#rules)
    - [Quotes](#quotes)
    - [Names](#names)
    - [Multi-line stuff](#multi-line-functions--lists--dicts--etc)
    - [`__all__`](#__all__)
    - [Type Hinting](#type-hinting)
    - [Import Order](#import-order)
    - [Collection Literals](#collection-literals)

# Linting
We have a [lint image](https://hub.docker.com/r/cloudcix/lint) that is used for linting Python3 repositories.

This image is to make it so that we have a single point of recording what lint packages are currently being used

## Current packages
- `flake8-quotes`: Checks that the [correct quotes](#quotes) are being used in all files
- `pep8_naming`: Checks that [names](#names) of classes/variables/functions/etc are following the [PEP8](https://www.python.org/dev/peps/pep-0008/) scheme
- `flake8-commas`: Checks that trailing commas are being used wherever necessary

# Rules
We follow the PEP8 style guide, but we also have a few extra styling rules on top of that, as seen below.

## Quotes
* Double (") quotes are to be used only for multi-line strings
* Single (') quotes are to be used for all other strings

## Names
- Variable names should follow the pep8 naming scheme of using `underscore_case` and not `camelCase`
- Function names should also be `underscore_case`
- Class names should be `CapCase`

## Multi-line functions / lists / dicts / etc.
If you have to split a function call or the definition of a list or dict over multiple lines (i.e. when a line gets too long), there are a couple of things to note;

- The first line should be empty
- The closing bracket should be on it's own line
- Every parameter / item should be on their own line
- There should be a comma after every parameter / item, including the last one (`flake8-commas`)

### Example
```python
# Like this
print(
    'Hello',
    'World',
)
d = {
    'a': 1,
    'b': 2,
}

# Not like this
print('Hello',
      'World')
d = {
    'a': 1,
    'b': 2
}
```

The reason for trailing commas on every line is so that when new items get added, the diff looks cleaner;

Without trailing commas;

d = {  
    'a': 1,  
{-    'b': 2-}  
{+    'b': 2,+}  
{+    'c': 3+}  
}  

With trailing commas;

d = {  
    'a': 1,  
    'b': 2,   
{+    'c': 3,+}  
}  

## `__all__`
Any file that creates classes or functions for importing should define a variable called `__all__` after its imports and before the rest of the code.

The `__all__` variable is a list of strings where the strings are the names of all of the members of the file that are importable elsewhere.

This list should be multi-line regardless of whether or not it is short enough to fit on one line

### Example
```python
import os

__all__ = [
    'delete_folder',
    'create_folder',
]


def delete_folder(folder_name: str):
    ...


def create_folder(folder_name: str):
    ...

def _check_folder_exists(folder_name: str):
    ...
```

## Type Hinting
In Python 3.6 we can add type hinting to function definitions.

For all the code being made in Python3, type hinting is now a requirement for all function definitions.

See the [typing guide](https://docs.python.org/3.6/library/typing.html) for more information.

This decision was made after we discovered how insufficiently the old code base was commented.

## Import Order
Imports should be grouped under the following sections;
1. stdlib
2. Third party libraries
3. Local application / library imports

Imports should be alphabetically ordered within these groups, with `import foo` style statements going before `from foo import bar` style statements.

Additionally, `import foo` style statements should only import one module each;
```python
# Bad
import os, sys

# Good
import os
import sys
```

## Collection Literals
When defining collections, (i.e `list`, `dict`, `set`, etc), use literals or comprehensions as much as possible.

```python
# Bad
l = list()
for x in other_list:
    l.append(x)

d = dict()
d['x'] = 1
d['y'] = 2


# Good
l = [x for x in other_list]
d = {
    'x': 1,
    'y': 2,
}
```