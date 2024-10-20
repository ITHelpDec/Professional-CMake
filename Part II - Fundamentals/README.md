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

### Using Subdirectories

We use two commands to help bring content from another file or build into the build:

```cmake
add_subdirectory()
include()
```

Distributing the build logic across the directory hierarchy offers a few advantages:

* Build logic is localised
* Builds can be built up of subcomponents (especially helpful for projects that use submodules or embed third party source trees)
* Turning parts of a build on / off becomes more trivial

### `add_subdirectory()`
```cmake
add_subdirectory(sourceDir [binaryDir]
    [EXCLUDE_FROM_ALL]
    [SYSTEM] # from CMake 3.25
)
```
NB: An added directory must have its own `CMakeLists.txt` (both relative and absolute paths can be added).

CMake only requires `binaryDir` if the `sourceDir` is a path outside of the source tree.

### Source and Binary Directory Variables

CMake provides a number of variables to keep track of the source and binary directories for the `CMakeLists.txt` file currently being processed:

* `CMAKE_SOURCE_DIR` - where the top-most directory (i.e. `CMakeLists.txt`) of the source tree resides (never changes)
* `CMAKE_BINARY_DIR` - where the top-most direcotry of the build tree resides (never changes)

* `CMAKE_CURRENT_SOURCE_DIR` - the directory of the `CMakeLists.txt` currently being processed (changes for every call to `add_subdirectory()` and it restored once processing is complete)
* `CMAKE_CURRENT_BINARY_DIR` - the build directory of the `CMakeLists.txt` currently being processed (changes for every call to `add_subdirectory()` and is restored once processing is complete)

An example has been included in the [variables](variables) directory.

### Scope

Calling `add_subdirectory()` adds a new scope block - that scope includes copying variables from the parent scope, obscuring local variables from the parent scope (modifications and un-setting of variables only take place in the child scope).

I've attached an example in [scope](scope).

`PARENT_SCOPE` can be used to change or unset a variable named one level up, e.g.:

```cmake
set(myVar bar PARENT_SCOPE) # adjusts the variable named "myVar" in the parent scope, but not the local
```
It is considered a better practice to define a local variable and then pass it back to the parent scope if you want the changes to propagate, e.g.:
```cmake
set(localVar bar)
set(myVar${localVar}} PARENT_SCOPE)
```

### When to call `project()`
In the top-level `CMakeLists.txt` file (rarely in the subdirectory files), although you can do multiple `project()` calls if you're working with Visual Studio - this can be very helpful for loading up only the portion of a larger codebase that is immediateful useful.

Xcode behaves in a similar way, but does not include the logic for building in projects outside of the directory scope.

### `include()`

Include can also be used to pull in content from other directories:

```cmake
include(fileName [OPTIONAL] [RESULT_VARIABLE myVar] [NO_POLICY_SCOPE])
include(module   [OPTIONAL] [RESULT_VARIABLE myVar] [NO_POLICY_SCOPE])
```

..., although it is slightly different to `add_subdirectory()`:

* `include()` looks for a file; `add_subdirectory()` looks for a directory and the `CMakeLists.txt` inside.
* `include()` doesn't introduce new variable scope
* both introduce new poicy scope, but this can be suppressed in `include()` with `NO_POLICY_SCOPE`
* `CMAKE_CURRENT_SOURCE_DIR` and `CMAKE_CURRENT_BINARY_DIR` don't change in `include()`

It's worth noting that all, but the first point, hold true when including a module.

There are also a few useful CMake variables that help when including a file:

* `CMAKE_CURRENT_LIST_DIR`  - like `CMAKE_CURRENT_SOURCE_DIR`, but stores the directory of where your included file is
* `CMAKE_CURRENT_LIST_FILE` - gives the name of the file being worked on
* `CMAKE_CURRENT_LIST_LINE` - holds the line number of the file currently being processed (probably only useful for debugging)

### Project-relative Variables

It's worth being mindful of `CMAKE_CURRENT_DIR` and `CMAKE_CURRENT_DIR` when your project is incorporated into another project (e.g. when used as a git submodule, or as part of a `FetchContent()` call) - a few CMake variables exist to help with this:

* `PROJECT_SOURCE_DIR`     - source directory of most recent call to `project()` (uses name from argument)
* `projectName_SOURCE_DIR` - as above, but more explicit about the project
* `PROJECT_BINARY_DIR`     - build directory of most recent call to `project()` (uses name from argument)
* `projectName_BINARY_DIR` - as above, but more explicit about the project

So, for example, if we had `project(topLevel)` then `topLevel_SOURCE_DIR` would point to our top-level direcotry, regardless of whether the project is built stand-alone or being incorporated into a larger project.

An easy way to figure out if your project is the top-level project is to check if `CMAKE_SOURCE_DIR` and `CMAKE_CURRENT_SOURCE_DIR` are the same value:

```cmake
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    add_subdirectory(packaging)
endif()
```

In newer versions of CMake (3.21 or later), there is a dedicated variable for this exact purpose - `PROJECT_IS_TOP_LEVEL`:

```cmake
if(PROJECT_IS_TOP_LEVEL)
    ...
endif()
```

> _"A similar variable, `<projectName>_IS_TOP_LEVEL`, is also defined by CMake 3.21 or later for every call to `project()`"_ - pg. 75

### Returning Processing Early

Not called in a function? Ends processing of the current file via `include()` / `add_subdirectory()`
Called in a function? Covered in later chapter.

Before CMake 3.25, `return()` could not return values; from 3.25 onwards, we can now return values using the `PROPAGATE` keyword (like with `block()`) - it's a nice way of passing variables back up the chain

```cmake
# CMakeLists.txt

set(x 1)
set(y 2)
add_subdirectory(subdir)

# subdir/CMakeLists.txt

cmake_minimum_required(VERISON 3.25) # needed for CMP0140

set(x 3)
unset(y)
return(PROPAGATE x y)
```

Similar things can be done with `block()`, although having a `return(PROPAGATE myVar)` does not lose its effectiveness when inside of a `block()`.

It's suggested that this is not considered a CMake best practice - consider this technique more for returning values from functions.

There are some cool almost-"header guard" applications like from C/C++:

```cmake
if(DEFINED cool_stuff_include_guard)
    return()
endif()

set(cool_stuff_include_guard 1)
```

..., although this can used more succinctly in CMake 3.10 with `include_guard()` - it can accept an optional `GLOBAL` or `DIRECTORY` keyword; `GLOBAL` will end the command if the file has been processed across the entire project, whereas `DIRECTORY` does the same thing in a reduced scope (within the current directory).

### Recommended Practices

* Typically, use `add_subdirectory()`
* `CMAKE_CURRENT_LIST_DIR` is better than `CMAKE_CURRENT_SOURCE_DIR`
* Steer clear of `CMAKE_SOURCE_DIR` and `CMAKE_BINARY_DIR`; where possible, opt for the `PROJECT_<>_DIR` / `projectName_<>_DIR` in their place
* Avoid using `PROPAGATE` in `return()` statements (_"violates the best practice of preferring to attach info to targets, rather than passing details around in variables."_ - pg. 78)
* Use `include_guard()` is using CMake 3.10 or later
* Try avoid calling `project()` in every subdirectory (unless it can be considered its own standalone project)

## Functions and Macros

### The Basics

Like with C/C++, functions introduce new scope, and macros are like `#define` macros.

```cmake
function(name [arg1 [arg2 [...]]])
    # Function body
endfunction()

macro(name [arg1 [arg2 [...]]])
    # Macro body
endmacro()
```

NB: `name` should only consist of letters (case insensitive), numbers, and underscores; it is also no longer needed as an argument to close functions or macros.

### Argument Handling Essentials
In functions, arguments are like normal variables; in macros, arguments are string replacements e.g. this means that `DEFINED` will not return `true` for macro arguments:

```cmake
function(func arg)
    if(DEFINED arg) # true
...

macro(macr arg)
    if(DEFINED arg) # false
...
```

Like with declaring `main()` in C/C++, there are a few pre-defined variables:
 * `ARGC` - count of named and unnamed arguments
 * `ARGV` - list of both named and unnamed arguments
 * `ARGN` - like `ARGV`, but only the unnamed arguments

You can also supply `ARGV<n>` (e.g. `ARGV0`) to reference indexed named variables, although bear in mind `<n>` is greater than `ARGC` is considered undefined behaviour.

The author provides a good example of how we can use `ARGV` to create a generic function for assigning various files to their respective test executables:

```cmake
function(add_my_test targetName)
    add_executable(${targetName} ${ARGV})

    target_link_libraries({$targetName} private foobar)

    add_test(NAME    ${targetName}
             COMMAND ${targetName})
endfunction()

add_mytest(smallTest small.cpp)
add_mytest(bigTest   big.cpp algo.cpp net.cpp)
```

> _"It allows a function or macro to take a varying number of arguments, yet still specify a set of named arguments which must be provided"_ – pg. 81

The author provides a good example of where we might want to be minful when passing a function into a macro i.e. which `ARGN` will we be using?

> _"When the macro is called from inside another function, the macro ends up using the ARGN variable from that enclosing function, not the ARGN from the macro itself"_ – pg. 82

It's best to consider using a `function()` in place of a `macro()`, in situations like this.

### Keyword Arguments
CMake alows for the parsing of arguments with `cmake_parse_arguments()` - introduced in version 3.5, we can use the following syntax...

```cmake
cmake_parse_arguments(
    prefix
    valuelessKeywords singleValueKeywords nultiValueKeywords
    argsToParse...
)
```
..., and this addtional syntax in 3.7.

```cmake
# Do not use in macros
cmake_parse_arguments(
    PRASE_ARGV startIndex
    prefix
    valuelessKeywords singleValueKeywords nultiValueKeywords
)
```

A reminder that macros use string replacement rather than variables for its arguments.

The author provides a good example of this parser in action:

```cmake
function(func)
    # Define the supported set of keywords
    set(prefix       ARG)
    set(noValues     ENABLE_NET COOL_STUFF)
    set(singleValues TARGET)
    set(multiValues  SOURCES IMAGES)

    # Process the arguments passed in
    include(CMakePraseArguments)
    cmake_parse_arguments(
        ${prefix}
        "${noValues}" "${singleValues}" "${multiValues}"
        ${ARGN}
    )

    # Log details for each supported keyword
    message("Option summary:")

    for each(arg IN LISTS noValues)
        if(${prefix}_${arg})
            message("  ${arg} enabled")
        else()
            message("  ${arg} disabled")
        endif()
    endforeach()
endfunction()

# Examples of calling with different combinations of keyword arguments

func(SOURCES    foo.cpp bar.cpp
     TARGET     MyApp
     ENABLE_NET
)

func(COOL_STUFF
     TARGET     dummy
     IMAGES     here.png there.png gone.png
)
```

The output will be:

```
# 1st func call
Option summary:
    ENABLE_NET enabled
    COOL_STUFF disabled
    TARGET = MyApp
    SOURCES = foo.cpp;bar.cpp
    IMAGES =

# 2nd func call
Option summary:
    ENABLE_NET disabled
    COOL_STUFF enabled
    TARGET = dummy
    SOURCES =
    IMAGES = here.png;there.png;gone.png
```

This seems like it might be quite useful in a situation where you're trying to create a tuple-like structure.

Any arguments that aren't linked to / prefaced by their "keys" (e.g. `ENABLE_NET`) can be accssed using the `<arg_name>_UNPARSED_ARGUMENTS` list:

```cmake
# similar to before...

message("Leftover args: ${ARG_UNPARSED_ARGUMENTS}")
foreach(arg IN LISTS ARG_UNPARSED_ARGUMENTS)
    message("${arg}")
endforeach()
```

The same goes for keywords that receive more arguments than they are expecting.

If a non-`multiValue` keyword is missing a value, then they will be added to a `ARG_KEYWORDS_MISSING_VALUES` list.

The author alos gives a good example of combining `group`s with multiple levels of `cmake_parse_arguments()` on page 87.

One gotcha to be mindful of is that `cmake_parse_arguments()` doesn't function like `target_link_libraries()`, which allows for its keywords (`PUBLIC` / `PRIVATE` / `INTERFACE`) to be called multiple times:

```cmake
target_link_libraries(
    PRIVATE lib1
    PRIVATE lib2
    PRIVATE lib3
)
```

`cmake_parse_arguments()` would only parse the last `PRIVATE` keyword (`lib3`), discarding all the previous ones (`lib1` and `lib2`).

In general, it's a good idea to opt for `cmake_parse_arguments()` than for an ad-hoc, manually-coded parser.

### Returning Values

> _"A fundamental difference between functions and macros is that functions introduce a new variable scope, whereas macros do not."_ – pg. 88

Functions also receive a copy of all variables from the calling scope, whereas macros do not, which makes me wonder if, like in C++ value semantics, there are performance benefits to using macros over functions, or if they're equally as frowned upon in CMake, as they are in modern C++...

We've already touched on the `PROPAGATE` keyword in earlier chapters (which will the nominated variables in the calling scope).

Again, this takes effect from CMake 3.25 - in 3.24 and earlier you would have had to use `PARENT_SCOPE` within `set()` to get a similar result:

```cmake
function(func resultVar1 resultVar2)
    set(${resultVar1} "First result" PARENT_SCOPE)
    set(${resultVar2} "Second result" PARENT_SCOPE)
endfunction()

func(myVar otherVar)

message("myVar:     ${myVar}")
message("otherVar:  ${otherVar}")
```
...will produce...
```
myVar:    First result
otherVar: Second result
```

### Returning Values from Macros

Macros can follow a similar set pattern, but because, unlike functions, they do not have their own scope, we do not need to pass the `PARENT_SCOPE` keyword), so the above example can be changed to the following:

```cmake
- set(${resultVar1} "First result" PARENT_SCOPE)
+ set(${resultVar1} "First result")
```

**N.B. Be mindful of using `return()` inside a macro - it may not produce the effect you're looking for!**

You have to imagine the contents of your macro are directly pasted into the place where you have called it.

### Overriding Commands
Interestingly, when `function()` or `macro()` is called to define a name that already exists, CMake will add a prefix to the existing function name.

```cmake
function(func)
endfunction()
...

# Later in the file
function(func) # new "func" function; previous "func" is now "_func"
endfunction()
```

The general recommendation is to treat overloads like "Highlander" - "there can be only one".

### Special Variables for Functions

In CMake 3.17, there are some useful variables (similar to the reserved macros in C/C++ e.g. `__FUNCTION__`), that are useful for debugging:

* `CMAKE_CURRENT_FUNCTION` (self-explanatory)
* `CMAKE_CURRENT_FUNCTION_LIST_FILE` - file path to file that defined the currently-executing function
* `CMAKE_CURRENT_FUNCTION_LIST_DIR` - absolute directory of the file that defines the currently-executing function
* `CMAKE_CURRENT_FUNCTION_LIST_LINE` - line number of the function's definition

It's worth noting that one variable will show where the function is _called_ (`CMAKE_CURRENT_LIST_DIR`), whereas the one described above (`CMAKE_CURRENT_FUNCTION_LIST_DIR`) will show where the called function is _defined_.

The other gives a good example of how this might be used when copying files (this was a little less idiomatic to implement in versions earlier than 3.17).

```cmake
function(writeSomeFile toWhere)
    configure_files(${CMAKE_CURRENT_FUNCTION_LIST_DIR}/template.cpp in ${toWhere} @ONLY)
endfunction()
```

### Other Ways of Invoking CMake Code

The author provides a good example of how `cmake_language()` (introduced from CMake 3.18) allows for useful generics, that would, otherwise, rely on multiple `if()` statements:

```cmake
function(qt_generate_moc)
    set(cmd qt${QT_DEFAULT_MAJOR_VERSION}_generate_moc)

    cmake_language(CALL ${cmd} ${ARGV})
endfunction()
```

The author also provides a really good use of `EVAL` over `CALL` in an example for delaying evaluation of variables to create a function that helps provide a backtrace (although, bear in mind this won't work in macros - only functions):

```cmake
set(myProjTraceCall) [=[
    message("Called ${CMAKE_CURRENT_FUNCTION}")

    set(__x 0)

    while(__x LESS ${ARGC})
        message("  ARGV{__x} = ${ARGV${__x}}")
        math(EXPR __x "${__x} + 1")
    endwhile()

    unset(__x)
]=])

functino(func)
    cmake_language(EVAL CODE "${myProjTraceCall}")
    # ...
endfunction()

func(one two three)
```

This is really quite an impressive way of allowing developers to express intent.

Speaking of deferral, you can even defer the execution of your command to the end of your directory (or another already-known directory's) scope).

```cmake
cmake_language(DEFER
    CALL message "End of current scope processing"
)

cmake_language(DEFER
    DIRECTORY ${CMAKE_SOURCE_DIR}
    CALL message "End of top-level processing"
)
```

You can also use IDs to help identify (or even group) deferrals for easier manipulation...

```cmake
cmake_language(DEFER
    ID_VAR deferredId
    CALL message "First deferred command"
)

cmake_language(DEFER
    ID_VAR ${deferredId}
    CALL message "Second deferred command"
)
```

...and `DEFER` sub-commands can be used to interact with these IDs (e.g. querying, cancelling, etc.):

```cmake
cmake_language(DEFER [DIRECTORY dir] GET_CALL_IDS    outVar) # returns list of currently-queued IDs
cmake_language(...                   GET_CALL     id outVar) # returns first command and its args for the ID passed
cmake_language(...                   CANCEL_CALL  ids...   ) # discards all deferred commands with provided IDs
```

Whilst deferring seems cool, following code execution, as a reader, can be less intuitive than non-deferred commands, and could potentially lead to variable pollution, so it may be worth considering a refactor at that stage, instead of opting for the likes of `DEFER`.

### Problems With Argument Handling

Consecutive spaces are treated as one single argument separator, i.e. ...

```cmake
someCommand(a b c)
someCommand(a     b     c)
```
...are the same.

The same goes for multiple `;` separators (many equals one).

```cmake
someCommand(a b;c)
someCommand(a;;;;b;c)
```

> _"Much of the confusion can be avoided by quoting arguments if they contain any variable evaluations. This eliminates any special interpretation of embedded spaces or semicolons."_ – pg. 98

The author shares an interesting example of where a slight modification of `cmake_parse_argument()` can help stop `${ARGV}` from flatting two lists provided to a function:

```cmake
func("a;b;c" "d;e;f") # would otherwise be flattened to "a;b;c;d;e;f" without `PARSE_ARGV 0`

cmake_parse_arguments(
    PARSE_ARGV 0 ARG
    "${noValues}" "${singleValues}" "${multiValues}"
)
```

### Forwarding Command Arguments

Another example of flattening is the following:

```cmake
function(inner)
    message("inner:\n"
            "ARGC = ${ARGC}\n"
            "ARGN = ${ARGN}"
    )
endfunction()

printArgs("a;b;c" "d;e;f")
```
```
ARGC = 2
ARGN = a;b;c;d;e;f
```

We can avoid this, by using a special technique from `cmake_parse_arguments()`:

```cmake
function(outer)
    cmake_parse_arguments(PARSE_ARGV 0 FWD "" "" "")
    inner(${FWD_UNPARSED_ARGUMENTS})
endfunction()
```
...instead of this...
```cmake
function(outer)
    message("outer:\n"
            "ARGC = ${ARGC}\n"
            "ARGN = ${ARGN}"
    )
    inner(${ARGN}) # naive forwarding
endfunction()
```

..., or the following, if we want to account for and preserve empty arguments (requires CMake 3.18 or above):

```cmake
function(outer)
    cmake_parse_arguments(PARSE_ARGV 0 FWD "" "" "")

    set(quotedArgs "")
    foreach(arg IN LISTS FWD_UNPARSED_ARGUMENTS)
        string(APPEND quotedArgs " [===[${arg}]===]")
    endforeach()

    cmake_language(EVAL CODE "inner(${quotedArgs})")
endfunction()
```

> _"Note the use of the bracket form for quoting. This ensures that any arguments with embedded quotes will be handled robustly too."_ – pg. 101

This technique doesn't carry over to macros - flattening cannot be prevented, but, if you're interested in preserving empty strings, then you can use the following template:

```cmake
macro(outer)
    string(REPLACE ";" "]===] [===[" args "[===[${ARGV}]===]")
    cmake_language(EVAL CODE "inner(${args})")
endmacro()
```

### Special Cases For Argument Expansion
The author provides a good example of forwarding a command with arguments, so, instead of this...

```cmake
set(cmdWithArgs message STATUS "Hello, world!")
cmake_language(CALL ${cmdWithArgs})
```
..., we call _this_:
```cmake
set(cmd message)
set(args STATUS "Hello, world!")
cmake_language(CALL ${cmd} ${args})
```

Other intricacies to be aware of is that `cmake_language(DEFER CALL)` defers the evaluation of the variables in the command's arguments, rather than the command itself.

```cmake
set(cmd message)
set(args "before deferral")
cmake_language(DEFER CALL ${cmd} ${args})

set(cmd somethingElse)
set(args "before deferral") # these arguments appear when the deferred call is made
```

If we want to evaluate both at the same time, then we need to wrap the `DEFER CALL` in an `EAVL CODE`:

```cmake
function(endOfScopeMessage msg)
    # [[]] quoting is used to ensure spaces are handled properly
    cmake_language(EVAL CODE "cmake_language(DEFER CALL message [[${msg}]])")
endfunction()
```

### Boolean gotchas
Be mindful of not using quotes on something you intend to be a string - this will be intepreted as a variable.

> _"There is no way to get an argument to be treated as quoted when providing the argument to `if()` (or `while()`) through an expanded variable evaluation..."_ – pg. 104

```cmake
set(noQuotes   someVar STREQUAL xxxx)
set(withQuotes someVar STREQUAL [["xxxx"]])

if(${noQuotes})
    message(YES)
else()
    message(NO)
endif()

if(${withQuotes})
    message(YES)
else()
    message(NO)
endif()
```

The author also gives a good example of how unevenly-matched square brackets can lead to strange behaviour when being evaluated, although this is quite a rare occurrence.

### Recommended Practices

* Prefer to opt for functions over macros
* Avoid calling `return()` in a macro
* Make use of `cmake_parse_arguments()` (improved robustness and scalability)
* Be mindful of dropping empty arguments and list flattening when forwarding command arguments
* Avoid using `cmake_language(DEFER)`
* Have a central source for functions (typically, in `xxx.cmake` files in a direcotry just below the top level)
* Avoid naming functions with a leading underscore (remember - it's for previous implementations that have been overloaded)
* * Also don't override any built-in CMake command

> _"Where the project's minimum CMake version is set to 3.17 or later, prefer to use `CMAKE_CURRENT_FUNCTION_LIST_DIR` to refer to any file or directory expected to exist at a location relative to the file in which a function is defined"_ – pg. 106
