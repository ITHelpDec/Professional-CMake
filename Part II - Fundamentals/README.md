# Professional CMake

## Highlights from Part II - Fundamentals

### Variables

A normal variable can be defined with the following syntax:

```cmake
set(varName value... [PARENT_SCOPE])
```

`varName` can be defined using letters, numbers and underscores, and is case-sensitive (`.`, `/`, `-`, and `+` are uncommon).

Like other languages, CMake variables have scope, and cannot be read or modified when outside of that scope.

CMake treats all variables as strings.

Values need only be quoted if they contain a space.

If multiple values are given, CMake will implicitly add a semi-colon between each value (to respresent a list) e.g.

```cmake
set(myVar a b c)   # myVar = "a;b;c"
set(myVar a;b;c)   # myVar = "a;b;c"
set(myVar "a b c") # myVar = "a b c"
set(myVar a b;c)   # myVar = "a;b;c"
set(myVar a "b c") # myVar = "a;b c"
```

We obtain the value of our variable by calling `${myVar}` (similar to parameter expansion in bash).

Variables do not need to be defined.

An empty variable will result in an empty string (`""`) - no warnings are given by default for uninitialised variables, but we can enable warnings with the `--warn-uninitialised` option (although, it is uncommon practice).

```cmake
set(foo ab)               # foo   = "ab"
set(bar ${foo}cd)         # foo   = "abcd"
set(baz ${foo }cd)        # foo   = "ab;cd"
set(myVar ba)             # myVar = "ba"
set(big "${${myVar}r}ef") # big   = "abcdef"
set${foo} xyz)            # ab    = "xyz"
set(bar ${notSetVar})     # bar   = ""
```

We can also do multi-line strings:

```cmake
set(singleLine "First line")
set(multiLine  "First line ${singleLine}
Second line with a \"quoted\" word")
```

From CMake 3.0 and later, we can also use the Lua-inspired bracket syntax

```cmake
set(multiLine [[
First Line
Second Line
]])
```

Like with raw string literals in C++, this syntax helps prevent unwanted substitution:

```cmake
# Lua-inspired
set(shellScript [=[
!#/bin/bash

[[ -n "${USER}" ]] && echo "Have USER"
]=])

# Non-Lua
set(shellScript
"!#/bin/bash

[[ -n \"${USER}\" ]] && echo \"Have USER\"
")
```

Personally, I prefer the Lua-inspired syntax for these more complex strings - to me, they are more easily readable.

Variables can also be "unset":

```cmkae
set(myVar abc)
unset(myVar)
```

> _"In addition to variables defined by the project for its own use, the behavior of many of CMake’s commands can be influenced by the value of specific variables at the time the command is called. This is a common pattern used by CMake to tailor command behavior or to modify defaults so they don’t have to be repeated for every command, target definition, etc"_ – pg. 34

### Cache Variables
Variables also exist in `CMakeCache.txt` and persist between CMake runs (they can also be amended).

Setting a cache variable has slightly different syntax to setting a normal variable:

```cmake
set(varName value... CACHE type "docstring" [FORCE]) # get with ${varName}
```

One thing to bear in mind is that `set()` will only override a cache variable if `FORCE` is passed.

The `"docstring"` is only there to provide markup for documentation in things like GUI tools / tooltips - it can also be empty, and it is recommended to avoid HTML markup.

The `type` is mostly used to improve the GUI tools experience, and consists of one of the following:
* `BOOL` (presents a tickbox in GUI tools)
* `FILEPATH`(presents a file dialog to hte user for modifying the variable's name)
* `PATH`(similar to above, but for the directory)
* `STRING`
* `INTERNAL` (do not show in GUI tools - also implies `FORCE`)

Setting a boolean cache variable is so common that CMake introduced a spcific `option` command:
```cmake
option(optVar helpString [initialValue]) # default is OFF
```
Can also be written as:
```cmake
set(optVar initialValue CACHE BOOL helpString)
```

Examples are given in later chapters where it explains how the behaviour of both commands are slightly different.

It is also worth bearing in mind that it is possible to have normal and cache variables with the same name, and CMake will give normal variables precedence.

The only expcetion is when using older CMake verisons, which may remove a normal variable from scope if a cache variable of the same name is being set.

An example of this kind of behaviour is shown below:

```cmake
set(myVar foo)                 # myVar  = "foo"
set(result ${myVar})           # result = "foo"
set(myVar bar CACHE STRING "") # Cache myVar

set(result ${myVar})           # First run:       result = bar
                               # Subsequent runs: result = foo

set(myVar fred)                # myVar  = fred
set(result ${myVar})           # result = fred
```

This behaviour changed in CMake 3.13 i.e. `option()` would not set a cache variable if that name already exists as a normal variable - a similar change was introduced for `set()` in CMake 3.21, but it is worth noting the subtle differences in behaviour:
* For `set()`, the cache variable is still set if it didn't exist previously (this is not the case for `option()`)
* If `INTERNAL` or `FORCE` is used with `set()`, the cache variable will always be set or updated

`$CACHE{myVar}` exists as a thing as of CMake 3.13, but it isn't encouraged to use it unless for temporary debugging purposes.

### Setting / Getting from the CLI

We can set cache variables from the CLI using the `-D` flag:
```cmake
cmake ... -D myVar:type=someValue ... # will replace any previous value (has an empty "docstring")
```

```cmake
cmake -D foo:BOOL=ON ...
cmake -D "bar:STRING=This contains spaces" ...
cmake -D hideMe=mysteryValue ... # similar to `INTERNAL` - CMake interprets as undefined
cmake -D helpers:FILEPATH=subdir/helpers.txt ...
cmake -D helpDir:PATH=/opt/helpThings ... # always specify a type with PATH to avoid relative path disasters
```

We can also remove variables from the cache usin the `-U` option:
```cmake
cmake -U "help*" -U foo ...
```

### CMake GUI Tools
CLI is great and all, but the GUI tools are said to provide a better user experience.

There are two tools worth noting:
* `cmake-gui`
* `ccmake` (good for ssh environments)

Interestingly, compared to the CLI, the GUI tool performs and "Configure" and "Generate" separately rather than together.

> _"Cache variables can be removed with the Remove Entry button, although CMake will most likely recreate that variable on the next run."_ – pg. 40


We can also pass a set of `STRING` types, which will show in a "combo box" - this can be represented in the command line with the following syntax:
```cmake
set(TRAFFIC_LIGHT Gren CACHE STRING "Status of something")
set_property(CACHE TRAFFIC_LIGHT PROPERTY STRINGS
    Red Orange Green
)
```

Sometimes, we might want to hide variables that aren't important to developers (like how we would with a .gitignore file) - we can do this by setting a cache variable as `ADVANCED`; `cmake-gui` only displays non-advanced variables by default.

Enabling the "Advanced" option in `cmake-gui` allows all cache variables (except `INTERNAL`) to be shown.

We can set this in our CMakeLists.txt file by using the following command:
```cmake
mark_as_advanced([CLEAR|FORCE] var1 [var2...])
# CLEAR = non-advanced
# FORCE = advanced
```

`ccmake` offers much of the same functinoality as the GUI, only in a TUI style of experience (similar to `gdb`) - one thing it does not offer is the ability to filter out variables.

### Block Scopes
> _"Cache variables have global scope, so they are always accessible...a non-cache variable's scope is the CMakeLists.txt file in which the variable is defined (directory scope)."_ – pg. 42

**NB: Subdirectories and functions inherit variables from their parent scope**

From CMake 3.25, we can use `block()` and `endblock()` to define local scope:
```cmake
set(x 1)

block()
    set(x 2) # overrides copy of x
    set(y 3) # creates new local variable
endblock()

# x is still 1 once the block closes, and y no longer exists
```

We can, however, use `PARENT_SCOPE` to alter the values of variables in enclosing scopes, but be warned - this is not quite the same as passing by reference in C++ (it's almost like passing by reference and by value at the same time):

```cmake
set(x 1)
set(y 3)

block()
    set(x 2 PARENT_SCOPE) # x still 1 inside of this scope
    unset(y PARENT_SCOPE) # y still 3 insode of this scope
endblock()

# x now 2 in this scope (parent scope), and y no longer defined
```

We can use the `PROPOGATE` keyword to induce something more similar to pass-by-reference from C++:
```cmake
set(x 1)
set(z 5)

block(PROPOGATE x z)
    set(x 2) # x in this scope and outer scope are now 2
    set(y 3) # local variable
    unset(x) # similar to z - unset in local and outer scope
endblock()

# Here, x = 2, and y and z are undefined
```

`block()`'s full command is outlined below...
```cmake
block([SCOPE_FOR [VARIABLES] [POLICIES]] [PROPOGATE var])
```
...and can be used for more than just variables - it can also adjust scope for policies as well e.g. if we were to pass the following `block()` in our previous example...

```cmake
...
block(SCOPE_FOR VARIABLES PROPOGATE x z)
...
```
..., then the behaviour would be similar for the variables, but the policy scope would remain unchanged.

If `SCOPE_FOR` is omitted, a new local scope is created for both `VARIABLES` and `POLICIES`.

### Choices

```cmake
block(SCOPE_FOR VARIABLES ...)
```
...will be slightly more efficient than...
```cmake
block()
```
..., but the latter is simpler.

### Printing Variable Values
We can print out the values of variables using the `message()` command:
```cmake
set(myVar HiThere)

message("The value of myVar = ${myVar}\nAnd this "
        "appears on the next line.")
```
...outputs...
```
The value of myVar = HiThere
And this appears on the next line
```

### String Handling

`string()` is a powerful command in CMake for string manipulation - the first argument we'll look at is `FIND`:

```cmake
string(FIND inputString subString outVar [REVERSE])
```
For example:
```cmake
string(FIND abcdefabcdef def fwdIndex)
string(FIND abcdefabcdef def revIndex REVERSE)

message([[
fwdIndex = ${fwdIndex}
revIndex = ${revIndex}
]])
```
...would output...
```
fwdIndex = 3
revIndex = 9
```

To replace strings, we use the `REPLACE` option:
```cmake
string(REPLACE matchString replaceWith outVar input...) # replaces every match, and multiple inputs are joined together
```

Regular expressions are also supported, and are available in `MATCH`, `MATCHALL`, and `REPLACE` variations:
```cmake
string(REGEX MATCH    regex outVar input...)             # only first match
string(REGEX MATCHALL regex outVar input...)             # all matches as a list
string(REGEX REPLACE  regex replaceWith outVar input...) # replaces all matches, unless specified with backtracking e.g. \1, \2
```
```cmake
string(REGEX MATCH    "[ace]"           matchOne abcdefabcdef)
string(REGEX MATCHALL "[ace]"           matchOne abcdefabcdef)
string(REGEX REPLACE  "([de])" "X\\1Y"  replVar1 abc def abcdef)
string(REGEX REPLACE  "([de])" [[X\1Y]] replVar1 abcdefabcdef)

message([=[
matchOne = ${matchOne}
matchAll = ${matchAll}
replVal1 = ${replVal1}
replVal2 = ${replVal2}
]=])
```
```
matchOne = a
matchTwo = a;c;e;a;c;e
replVal1 = abcXdYXeYfabcXdYXeYf
replVal1 = abcXdYXeYfabcXdYXeYf
```

### Substrings
We can extract substrings with hte following syntax:
```cmake
string(SUBSTRING input index length outVar)
```
e.g.
```cmake
string(SUBSTRING "chacha" 0  2 outVar1) # "ch"
string(SUBSTRING "chacha" 3 -1 outVar2) # "cha" (prints full word)
string(SUBSTRING "chacha" 0  8 outvar3) # "chacha" (would have errored in CMake 3.11, but no longer)
```

We have other typical string manipulation members, such as...
```cmake
string(LENGTH  "chachaslide" outVar1) # 8
string(TOLOWER "cHaChAsLiDe" outVar2) # "chachaslide"
string(TOUPPER "cHaChAsLiDe" outVar3) # "CHACHASLIDE"

string(STRIP   "   loads of whitespace    " outVar4) # "loads of whitespace"
```

### Lists
Two popular members for list are:
```cmake
list(LENGTH listVar outVar)
list(GET    listVar index  [index...] outVar)
```
e.g.
```cmake
set(myList a b c)

list(LENGTH myList len)
message("length = ${len}") # 3

list(GET myList 2 1 letters)
message("letters = ${letters}") # c;b
```

We can also `INVERT`, `APPEND`, and `PREPEND` to lists:
```cmake
list(INSERT  listVar index item [item ...])
list(APPEND  listVar item [item ...])
list(PREPEND listVar item [item ...]) # works from CMake 3.15
```
e.g.
```cmake
set(myList a b c)

list(INSERT myList 2 X Y Z)
message("myList = ${myList}") # a;b;X;Y;Z;c

list(APPEND myList d e f)
message("myList = ${myList}") # a;b;X;Y;Z;c;d;e;f

list(PREPEND myList P Q R)
message("myList = ${myList}") # P;Q;R;a;b;X;Y;Z;c;d;e;f
```

We can find items in a list with `FIND`...
```cmake
list(FIND myList value index)
```
e.g.
```cmake
set(myList a b c d e f)

list(FIND myList c index)
message("c is in position ${index}") # 5 (0-based indices)
```
..., and we can remove items with `REMOVE_ITEM`, `REMOVE_AT`, and `REMOVE_DUPLICATES`:
```cmake
list(REMOVE_ITEM       myList value [value...]) # not found?     no error
list(REMOVE_AT         myList index [index...]) # out of bounds? yes error
list(REMOVE_DUPLICATES myList)
```

From CMake 3.15, we can perform similar popping functions like in C++ with `POP_FRONT` and `POP_BACK`:
```cmake
list(POP_FRONT myList [outVar1 [outVar2...]])
list(POP_BACK  myList [outVar1 [outVar2...]])
```
e.g.
```cmake
set(myList a b c)

list(POP_FRONT myList)
message("myList = ${myList}") # b;c

list(POP_BACK myList backVar)
message("backVar = ${backVar}") # c
message("myList  = ${myList}")  # b
```

We can also `REVERSE` and `SORT` lists:
```cmake
list(REVERSE myList)
list(SORT    myList [COMPARE method] [CASE case] [ORDER order]) # all keywords available from CMake 3.13
```
For `SORT`, `COMPARE` must be one of the follwoing:
* `STRING` (alphabetical - the default when not supplied)
* `FILE_BASENAME`
* `NATURAL` (similar to [`strverscmp()` in C](https://man7.org/linux/man-pages/man3/strverscmp.3.html) - only available from CMake 3.18, and handy for sorting strings that also contain numbers)

`CASE` can be `SENSITIVE` or `INSENSITIVE`, and `ORDER` can be `ASCENDING` or `DESCENDING`.

**NB:** When you give a negative index, you're telling CMake to start from the end of the list and work backwards, i.e. `-1` means the last element, `-2` means the element before that.

e.g.
```cmake
set(myList a b c D e F)

list(REVERSE myList)
message("myList = ${myList}") # F;e;D;c;b;a

list(SORT myList CASE SENSITIVE)
message("myList = ${myList}")    # D;F;a;b;c;e

list(SORT myList CASE INSENSITIVE)
message("myList = ${myList}")    # a;b;c;D;e;F
```

### Problems with unbalanced square brackets
> _"...if a list item contains an opening square bracket `[`, it must also have a matching closing square bracket `]`"_ – pg. 49

An example of how things could go wrong is below:
```cmake
set(noBrackets   "a_a" "b_b")
set(withBrackets "a[a" "b]b")

list(LENGTH noBrackets   lenNo)
list(LENGTH withBrackets lenWith)

list(GET noBrackets   0 firstNo)
list(GET withBrackets 0 firstWith)

message("No brackets:   Length = ${lenNo}   --> First_element = ${firstNo}")   # "a_a"
message("With brackets: Length = ${lenWith} --> First_element = ${firstWith}") # "a[a;b]b
```

### Math
We can calculate mathematical expressions with the following syntax:
```cmake
math(EXPR outVar mathExpr [OUTPUT_FORMAT format])
```
e.g.
```cmake
set(x 3)
set(y 7)

math(EXPR zDec "(${x} + ${y}) * 2")

message("decimal = ${zDec}") # 20

# From CMake 3.13, we can work with hexidemical
math(EXPR zHex "(${x} + ${y}) * 2" OUTPUT_FORMAT HEXADECIMAL)

message("hexadecimal = ${zHex}") # 0x14
```

### Recommended Practices
Give CMake GUI a go.

Opt for cache variables instead of encoding the logic into build scripts outside of CMake (easy to switch on and off in the likes of CMake GUI).

Avoid relying on environment variables being defined in order for the project to work - prefer to pass information directly to CMake through cache variables instead wherever possible.

Consider using a prefix to distinguish cache variables from normal variables (e.g. `C_`) to help prevent the issues described earlier in the chapter around naming clashes.

And speaking of naming variables, as with any language, be mindful of pre-defined variables within CMake (e.g. `WIN32`, `APPLE`, etc., ...).

### Flow Control
In CMake, we have the classic `if()`, `foreach()`, and `while()` flow control elements:
```cmake
if(expression1)
    # commands ...
elseif(expression2)
    # commands ...
else()
    # commands ...
endif()
```
`TRUE` (quoted or unquoted)
* "ON"   / ON
* "YES"  / YES
* "TRUE" / TRUE
* "Y"    / Y

`FALSE` (quoted or unquoted)
* "OFF"      / OFF
* "NO"       / NO
* "FALSE"    / FALSE
* "N"        / N
* "IGNORE"   / IGNORE
* "NOTFOUND" / NOTFOUND
* "" (empty string)
* "<ANY_STRING>-NOTFOUND

Be mindful of floating-point numbers that will truncate and become an inplicit boolean.

We must also be mindful of the "fall-through" case:
```cmake
if(<variable|string>)
```
When unoquoted, the variable is compared to false constants - if there's no match, it returns true; an undefined variable returns an empty string, which we know now is a false constant, so returns false.

**NB:** Environment variables will always return false.

Quoted strings are a little trickier:
* Doesn't match any of the true constants? Returns false. (CMake 3.1 and later)
* Quoted string same name as existing variable? Quoted string substituted by value of simiarly-named unqouted string (before CMake 3.1)

As a result, it's generally advised to avoid using quoted arguments with the `if(...)` form.

### Logic Operators
We can use the typical boolean modifiers `AND`, `NOT` and `OR` as part of our expression:
```cmake
if(NOT expression)
if(expression1 AND expression2)
if(expression1 OR expression2)

if(NOT (expression1 AND (expression2 OR expression3)))
```

### Comparison Operators
| Numeric                  | String                      | Version numbers                  | Path                     |
| -------------            | ---------------             | --------------------             | -------------            |
| `LESS`                   | `STRLESS`                   | `VERSION_LESS`                   |                          |
| `GREATER`                | `STRGREATER`                | `VERSION_GREATER`                |                          |
| `EQUAL`                  | `STREQUAL`                  | `VERSION_EQUAL`                  | `PATH_EQUAL`<sup>2</sup> |
| `LESS_EQUAL`<sup>1</sup> | `STRLESS_EQUAL`<sup>1</sup> | `VERSION_LESS_EQUAL`<sup>1</sup> |                          |
| `LESS_EQUAL`<sup>1</sup> | `STRLESS_EQUAL`<sup>1</sup> | `VERSION_LESS_EQUAL`<sup>1</sup> |                          |

<sup>1</sup> - from CMake 3.7

<sup>2</sup> - from CMake 3.24

e.g.

```cmake
# TRUE
if(2 GREATER 1)
if("23" EQUAL 23)

set(val 42)
if(${val} EQUAL 42)
if("${val}" EQUAL 42)

# TRUE, but invalid (be mindful)
if("23a" EQUALS 23)
```
For version numbers:
```cmake
if(1.2         VERSION_EQUAL   1.2.0)   # TRUE
if(1.2         VERSION_LESS    1.2.3)   # TRUE
if(1.2.3       VERSION_GREATER 1.2)     # TRUE
if(2.0.1       VERSION_GREATER 1.9.7)   # TRUE
if(1.8.2       VERSION_LESS    2)       # TRUE
```
`PATH_EQUAL` is quite clever, as you might have an environment vairable for a directory that includes a `/` by mistake - with `STREQUAL`, this would result in an issue; with `PATH_EQUAL`, however, any duplicated directory separators (`/`) would be collapsed into one e.g.

```cmake
# comparison is TRUE
if("/a//////b/c" PATH_EQUAL "/a/b/c")
    ...
endif()

# comparison is FALSE
if("/a//////b/c" STREQUAL "/a/b/c")
    ...
endif()
```

CMake also support regular expressions:

```cmake
if(value MATCHES regex)
```
```cmake
if("Hi from ${who}" MATCHES "Hi from (Fred|Barney).*")
    message("${CMAKE_MATCH_1} says hello") # can be CMAKE_MATCH_<n>, depending on which match you want
endif()
```

### File System Tests
These are quite handy when flow control relies on the result of a file's / directory's state, e.g.:

```cmake
if(EXISTS pathToFileOrDir) # AND readable (bit of a gotcha)

if(IS_READABLE pathToFileOrDir)   # from CMake 3.29
if(IS_WRITABLE pathToFileOrDir)   # from CMake 3.29
if(IS_EXECUTABLE pathToFileOrDir) # from CMake 3.29

if(IS_DIRECTORY pathToDir)
if(IS_SYMLINK fileName)
if(IS_ABSOLUTE path)
if(file1 IS_NEWER_THAN file2)
```

**NB**
> _"Unlike most other if() expressions, none of the above operators perform any variable/string substitution without ${}, regardless of any quoting"_ – pg. 57

`IS_NEWER_THAN` is a bit of a gotcha - if both files have the same timestamp, then it will return `TRUE` (false positive?), and it will also return `TRUE` if ANY one of the files is missing.

Here's a good example from the book:

```cmake
set(firstFile  "/full/path/to/somewhere")
set(secondFile "/full/path/to/somewhere")

if(NOT EXISTS ${firstFile})
    message(FATAL_ERROR "${firstFile} is missing")
elseif(NOT EXISTS {$secondFile} OR
       NOT ${secondFile} IS_NEWER_THAN ${firstFile}) # it's a good idea to reverse negate the logic sometimes
    # ... commands to to recreate secondFile
endif()
```
...would be fine, but this...
```cmake
if(${firstFile} IS_NEWER_THAN ${secondFile})
    # ...
endif()
```
...might offer a false sense of security, e.g. if `${firstFile}` doesn't exist.

### Existence Tests
```cmake
if(DEFINED name)
if(COMMAND name)
if(POLICY name)
if(TARGET name)
if(TEST name)             # from CMake 3.4
if(value IN_LIST listVar) # from CMake 3.3
```

### `DEFINED`
Beyond just checking if any variable has been defined (regardless of value), we can also single out cache or environment variables:
```cmake
if(DEFINED SOMEVAR)
if(DEFINED CACHE{SOMEVAR})) # CMake 3.14
if(DEFINED ENV{SOMEVAR}))   # CMake 3.13
```

### `COMMAND`
Like `DEFINED`, but for CMake commands (ideally the one's created by the user) - it's usually a better idea to test for a specific CMake version, if you want to check for the existence of a certain CMake command.

### `POLICY`
Handy for checking the existence of policies like the `CMP0060` policy described earlier in the book ([here](https://cmake.org/cmake/help/latest/policy/CMP0060.html)).

### `TARGET`
Pretty self-explanatory - helpful in larger hierarchies.

### `TEST`
Again, self-explanatory.

### `IN_LIST`
The right-hand operand must not be a string literal - it must always be a variable name.
```cmake
# Correct
set(things A B C)
if("B" IN_LIST things)
    ...
endif()

# Incorrect
if("B" IN_LIST "A;B;C")
    ...
endif()
```

### Hall of Shame
```cmake
if(WIN32)
    set(platformImpl source_win.cpp)
else()
    set(platformImpl source_generic.cpp)
endif()
```
It would be more accurate to use the following, though, as it expresses more accurately the intent to use the MinGW compiler:
```cmake
if(MSVC)
    set(platformImpl source_win.cpp)
...
```
`if(APPLE)` is another gotcha - just because we're building for macOS doesn't mean we're intending to use the Xcode generator.

`if(CMAKE_GENERATOR STREQUAL "Xcode")` would express the intent more clearly.

### Hall of Fame
A good technique is the following for conditionally including targets given the state of a CMake option, e.g.
```cmake
option(BUILD_MYLIB "Enable building the MyLib target)
if(BUILD_MYLIB)
    add_library(MyLib src1.cpp src2.cpp)
endif()
```
It becomes especially useful when creating scripted builds for the likes of CI.

### Looping

As you'd expect, `foreach()` is included as part of flow control for individual items...

```cmake
foreach(loopVar arg1 arg2 ...)
    # loopVar takes the place of each argN
endforeach()
```
...and lists of items...
```cmake
foreach(loopVar IN [LISTS listVar1 ...] [ITEMS item1 ...])
    # ...
endforeach()
```
For example:
```cmake
set(list1 A B)
set(list2)
set(foo WillNotBeShown)

foreach(loopVar IN LISTS list1 list2 ITEMS foo bar)
    message("Iteration for: ${loopVar}")
endforeach()
```
...will print...
```
Iteration for: A
Iteration for: B
Iteration for: foo
Iteration for: bar
```
We can also work through lists in parallel, as of CMake 3.17, with `ZIP_LISTS`, using almost a C++ "structured bindings"-style approach...
```cmake
foreach(var0 var1 IN ZIP_LISTS list1 list1)
    message("Vars: ${var0} ${var1}")
endforeach()
```
...or assign a generic loop variable name and append an `_N` suffix during the variable calls:
```cmake
foreach(var IN ZIP_LISTS list1 list1)
    message("Vars: ${var_0} ${var_1}")
endforeach()
```

The lists also don't need to be the same length:
```cmake
foreach(varLong VarShort IN ZIP_LISTS long short)
    message("Vars: ${varLong} ${VarShort}")
endforeach()
```
...produces...
```
Vars: A justOne
Vars: B
Vars: C
```

You can also supply ranges with either...
```cmake
foreach(loopVar RANGE start stop [step])
```
...or...
```cmake
foreach(loopVar RANGE value) # equivalent of RANGE 0 value
```
### `while()`
`while()` follows a similar setup:
```cmake
while(condition)
    # ...
endwhile()
```

### Early exit
Like with C-style languages, we can exit both `while()` and `foreach()` using the `break()` command, and skip ahead to the next iteration using the `continue()` command:
```cmake
foreach(outerVar IN ITEMS a b c)
    unset(s)

    foreach(innerVar IN ITEMS 1 2 3)
        list(APPEND s ${outerVar}${innerVar})
        string(LENGTH "${s}" length)

        if(length GREATER 5)
            break()
        endif()

        if(outerVar STREQUAL "b")
            continue()
        endif()

        message("Processing ${outerVar}-${innerVar}")
    endforeach()

    message("Accumulated list: ${s}")
endforeach()
```
...produces...
```
Processing a-1
Processing a-2
Accumulated list: a1;a2;a3
Accumulated list: b1;b2;b3
Processing c-1
Processing c-2
Accumulated list: c1;c2;c3
```

You can also use `break()` and `continue()` within `block()`s - the book provides a good example of this flow:
```cmake
set(log "Value: ")
set(values one two skipMe three stopHere four)
set(didSkip FALSE)

while(NOT values STREQUAL "")
    list(POP_FRONT values next)

    block(PROPAGATE didSkip)
        string(APPEND log "${next}")

        if(next MATCHES "skip")
            set(didSkip TRUE)
            continue()
        elseif(next MATCHES "stop")
            break()
        elseif(next MATCHES "t")
            string(APPEND log ", has t")
        endif()

        message("${log}")
    endblock()
endwhile()

message("Did skip: ${didSkip}")
message("Remaining values: ${values}")
```
...produces...
```
Value: one
Value: two, has t
Value: three, has t
Did skip: TRUE
Remaining values: four
```

### Recommended Practices

* Try not to mix strings up with variable names
* Avoid unary expressions with quotes (opt for string comparison operations instead)
* Use at least CMake 3.1 to disable the old behaviour that allowed implicit conversion of quoted string values to variable names

* Capture regex results from `CMAKE_MATCH_<>` to variables as early as possible to prevent being overwritten

* Avoid ambiguous loops, e.g.
  * Give `RANGE` forms a definitive start and end
  * Consider whether `IN LISTS` or `IN ITEMS` is more expressive of your intent (in place of the following..)

```cmake
foreach(loopVar item1 item2 ...)
```
