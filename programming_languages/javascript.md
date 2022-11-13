---
    title: JavaScript Notes
    author: shawshary
    date: 2022-11-06
---

\newpage

# Reversed Identifiers #
`arguments break case catch class const continue debugger default delete do
else enum eval export extends false finally for function if implements import
in instanceof interface let new null package private protected public return
static super switch this throw true try typeof var void while with yield`

Technically, the following tree identifiers are not reversed words, but
shouldn't be used as variable names, either: `Infinity NaN undefined`



# Values #

## Primitive values versus objects ##

- The *primitive values* are: booleans, numbers, strings, null, undefined.
- All other values are *objects*. That is actually how objects are defined.

The main difference between the two is how they are **compared**: each object
has a unique identity and is only equal to itself. In contrast, all primitive
values encoding the same value are considered the same.


## undefined and null ##

- undefined means "no value". Uninitialized variables are undefined.
- null means "no object". It is used as a non-value when an object is expected
  (parameters, last in a chain of objects, etc.).


## Categorizing values via typeof and instanceof ##
There are two operators for categorizing values: *typeof* is mainly used for
primitive values, *instanceof* is used for objects.



# Numbers #
All numbers in JavaScript are floating point (although most JavaScript engines
internally also sue integers).
