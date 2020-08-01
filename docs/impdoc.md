# IMP: Documentation Format

> This document is a **work-in-progress** and still requires changes with respect to Minecraft 1.15+ and the [IMP datapack](https://github.com/Arcensoth/imp-datapack).

- [Function headers][function-headers]
- [Function annotations][function-annotations]
  - [`@user`][function-annotation-user]
  - [`@public`][function-annotation-public]
  - [`@api`][function-annotation-api]
  - [`@context`][function-annotation-context]
  - [`@within`][function-annotation-within]
  - [`@handles`][function-annotation-handles]
  - [`@patch`][function-annotation-patch]
  - [`@input` and `@output`][function-annotations-input-and-output]
    - [for scoreboard][function-annotations-input-and-output-for-scoreboard]
    - [for storage][function-annotations-input-and-output-for-storage]
  - [`@reads` and `@writes`][function-annotations-reads-and-writes]
- [Storage recommendations][storage-recommendations]
  - [Storage tag types][storage-tag-types]
  - [Separate namespace for transient storage][separate-namespace-for-transient-storage]
  - [Root tags for transient storage][root-tags-for-transient-storage]

The goal is to create a documentation format for functions that:

1. provides important and useful information; and
2. identifies and outlines common patterns; and
3. is easy to read and write.

These goals also have the intended side-effect of allowing external tools, such as syntax highlighters and documentation compilers, to quickly and reliably identify the different components of a function.

## Function headers

A function always begins with a **function header**, which is a series of comments that describe its purpose, context, and interface:

```mcfunction
#> imp:core/load
#
# Main loading procedure with the core entity as context.
#
# Anything branching from here can use `@s` instead of the core UUID.
#
# @context core
```

> Function headers do not contain information such as: who created the function, when it was last updated, etc. This information is expected to be handled automatically by a version control system such as Git. Contributors to the datapack should be listed in a centralized location easily accessible to the end-user, such as the datapack description and/or a user function that can be run in-game.

The first line of the function header is preceded by `>` and contains nothing but the function name:

```mcfunction
#> imp:core/load
```

This line requires adjustment when changing the name/location of the function, but makes it clear which function is being modified and easy to copy-paste into a `function` command.

> The first line of the function header is preceded by `>` to denote an imaginary block comment. This is an alternate, vanilla-compatible comment style that allows syntax highlighters and other external tools to quickly and reliably identify block comments.

The next section of the function header is a description of the function's purpose. This can be split into multiple paragraphs:

```mcfunction
# Main loading procedure with the core entity as context.
#
# Anything branching from here can use `@s` instead of the core UUID.
```

Following the description is a series of annotation tags that define the function's interface:

```mcfunction
# @context core
```

Similar to the reason why we use the block comment syntax, these annotations help external tools identify distinct components of the function. See the section on [annotations] for details regarding the purpose and capabilities of each individual annotation.

## Function annotations

### Function annotation `@user`

Declare that the function is a **user** function. User functions are intended to be run from chat by operators.

```mcfunction
#> imp:-user/menu
#
# @user
```

### Function annotation `@public`

Declare that the function is a **public** function. Public functions are designed to be called by other modules, and so they should be well-documented and tested for other developers.

```mcfunction
#> imp:utils/resolve_text
#
# Resolve a text component.
#
# @public
```

### Function annotation `@api`

Declare that the function is an **API** function. API functions are based on [public functions][function-annotation-public] but are typically exposed as part of a versioned API that attempts to abstract core logic in order to maintain backwards-compatible when possible. API functions are otherwise effectively identical to public functions.

```mcfunction
#> imp:api/v1/resolve_text
#
# Resolve a text component.
#
# @api
```

### Function annotation `@context`

Declare the entity execution context that is expected at runtime. Any context outside what is declared may lead to erroneous or undefined behaviour.

The context is an arbitrary string but should be minimally descriptive. For example, `root` is generally accepted to mean the server root context:

```mcfunction
#> imp:load
#
# Root entry point for the main loading procedure.
#
# @context root
```

The context does not need to be self-explanatory, but it is good practice to be unambiguous and consistent:

```mcfunction
#> imp:core/load
#
# Main loading procedure with the core entity as context.
#
# Anything branching from here can use `@s` instead of the core UUID.
#
# @context core
```

Sometimes it's necessary to be more verbose, especially if the context is being used in a very limited scope:

```mcfunction
#> imp:log/calc/coords/inner
#
# @context temporary entity used to extract coords from nbt
#
# @within imp:log/calc/coords
```

Public functions commonly use `@context any` which does not have any special meaning but serves to compliment `@public` by explicitly stating that the function will operate correctly under any context:

```mcfunction
#> imp:utils/resolve_text
#
# Resolve a text component.
#
# @public
#
# @context any
```

### Function annotation `@within`

Declare that the function is a **child** function and should only ever be called from _within_ one of its **parent** functions.

This pattern is commonly used when a function needs conditional logic but it would be suboptimal to include said logic in the same function. This is where a branch is used, and how function trees are constructed.

> Function trees are generally constructed from several layers of parent and child functions. It is unusual to call a function tree from somewhere in the middle, unless the caller is well-aware of how it is constructed and any assumptions that need to be fulfilled.

This annotation has two forms. The first is a short-form that should be used when there is only one parent function:

```mcfunction
#> namespace:path/to/function
#
# An imaginary example function.
#
# @within namespace:some/parent/function
```

The second form spans multiple lines and can be used for any number of parent functions:

```mcfunction
#> namespace:path/to/function
#
# An imaginary example function.
#
# @within
#   namespace:some/parent/function
#   namespace:another/parent/function
```

This pattern may also be used for API functions that require a certain context, and would otherwise place the burden of setting the context on the developer. This also helps to ensure that future versions of the function remain backwards-compatible.

For example, a public parent function is created that calls the child function with the correct context:

```mcfunction
#> namespace:my/parent/function
#
# An imaginary parent function.
#
# @public
#
# @context any

execute as @e[type=minecraft:area_effect_cloud, tag=namespace.marker] at @s run function namespace:my/child/function
```

And the child function documents this using `@within`:

```mcfunction
#> namespace:my/child/function
#
# @context marker entity
#
# @within namespace:my/parent/function
```

### Function annotation `@handles`

Declare that the function is an event handler for one or more function tags.

> Whether this should be used for all function tags is undecided, but the likely answer is: no. This annotation was primarily introduced to compliment `@context` in helping to remind the developer - at a glance - of the circumstances under which the function will be running.

Similar to `@within`, this annotation has two forms. The first form is used for a single event handler:

```mcfunction
#> imp:load
#
# Root entry point for the main loading procedure.
#
# @handles #minecraft:load
```

The second form is used for multiple event handlers:

```mcfunction
#> namespace:path/to/function
#
# An imaginary example function.
#
# @handles
#   #namespace:some/function/tag
#   #namespace:another/function/tag
```

### Function annotation `@patch`

Declare that the function is a **patch** function. Patch functions are intentional overrides.

```mcfunction
#> imp:utils/timestring/inner
#
# Extend IMP's timestring to include sub-tick resolution via Tickbuster.
#
# @patch
```

### Function annotations `@input` and `@output`

Declare the expected values to be defined before and/or after the function executes.

#### Function annotations `@input` and `@output` for scoreboard

For scores, we use the form `score <score_holder> <objective>` which makes it easy to copy-paste to/from `execute if` commands:

```mcfunction
# @input
#   score $base mypack.foo
#       The base number to multiply.
#   score $exponent mypack.foo
#       The exponent to use.
#
# @output
#   score $result mypack.foo
#       The resulting number.
```

#### Function annotations `@input` and `@output` for storage

For storage, we use the form `storage <storage_name> <root_tag>` for the same reason, respectively. Here are some additional key observations:

1. The `<root_tag>` here is the implied prefix/parent tag for each of the nested parameters.
2. The reason the `<root_tag>` is separate is because NBT and can be nested and repeating the root tag would be repetitive.
3. We include an additional line before the description to denote the tag's [expected type][storage-tag-types] in NBT.

There are some additional recommendations ahead, but here's what the basic form looks like:

```mcfunction
# @input
#   storage mypack:stuff person
#       first_name: string
#           The first name of the person.
#       second_name: string
#           The last name of the person.
#
# @output
#   storage mypack:stuff person
#       full_name: string
#           The resulting full name of the person.
```

However, there are some additional recommendations when using `storage` for this purpose:

1. The storage name should have a 1:1 correlation to the function name.
   - Example: `storage mypack:utils/get_full_name`
   - This helps to avoid ambiguity and makes it clear where each storage is being created.
   - It also helps to avoid one function from accidentally accessing the data of another.
2. Use a [separate namespace][separate-namespace-for-transient-storage] ending in `.__temp__` for all transient storage.
   - Example: `storage mypack.__temp__:utils/get_full_name`
3. Due to the limitations of selecting root NBT paths, it is also recommended to use separate [transient root tags][root-tags-for-transient-storage] for input vs output data.

Putting it all together, here's what the final form looks like:

```mcfunction
# @input
#   storage mypack.__temp__:utils/get_full_name __input__
#       first_name: string
#           The first name of the person.
#       second_name: string
#           The last name of the person.
#
# @output
#   storage mypack.__temp__:utils/get_full_name __output__
#       full_name: string
#           The resulting full name of the person.
```

### Function annotations `@reads` and `@writes`

Declare any additional values that are being read or written by the function.

Useful for declaring any other noteworthy scores or storage that are being accessed or modified during the execution of the function, which are not already defined as either `@input` or `@output`. Note that if a value is defined under `@writes` then it will be implicitly `@reads` as well.

The syntax here is identical to that of [`@input` and `@output`][function-annotations-input-and-output].

Here's a brief example:

```mcfunction
#> imp:core/registry/sync/process_registrants
#
# Do a second pass over compiled registrants, so that we can process them
# further compare them with one another in full.
#
# @reads
#   storage imp.__temp__:core/registry/sync __temp__
#       compiled_registrants: Module[]
#           The list of compiled modules.
#
# @writes
#   storage imp.__temp__:api/manage __temp__
#       install: ModuleID[]
#           IDs of modules that need to be (re-)installed.
```

Note that this example is making use of another [transient root tag][root-tags-for-transient-storage]: `__temp__`.

However, since not all storage is transient storage, it may be worthwhile to also list any persistent storage that's being accessed:

```mcfunction
#> imp:core/registry/forget
#
# Clears cached, inactive modules from the registry. This is equivalent to
# forgetting all disabled modules.
#
# @writes
#   storage imp:registry
#       modules: Module[]
#           The registry of all recorded modules.
```

## Storage recommendations

### Storage tag types

There are several standard atomic tag types defined. These all match their NBT equivalent:

- `byte`
- `short`
- `int`
- `long`
- `float`
- `double`
- `string`

There are some other names for commonly-used patterns:

- `bool` is an `int` that's either `true` (`1b`) or `false` (`0b`)
- `RawTextComponent` is a raw JSON text component `string`
- `Command` is a command `string`

Anything ending in `[]` is an array:

- `int[]` is an array of `int`s
- `Command[]` is an array of command `string`s

There is currently no standard format for specifying nested types via compound tags, however it is recommended to give them unique, identifiable names. For example, the `Module` compound is a common data structure used in IMP to record information about a datapack module.

### Separate namespace for transient storage

Transient storage includes anything involving argument passing, return values, and intermediate calculations, that need only be preserved in the current scope or tick.

Using a separate namespace for this type of storage allows us to separate important, persistent data from everything else. It also allows us to safely purge or delete any storage files following this naming scheme.

The recommended standard for naming transient storage is by appending `.__temp__` to the namespace of the storage. For example, the namespace `imp.__temp__` is used for all of IMP's argument passing, return values, and intermediate calculation storage.

### Root tags for transient storage

Transient root tags are used primarily to work-around certain limitations with selecting root NBT paths. However, they also double as an obvious visual indicator for common patterns when using storage.

These are the recommended standard root tag names for transient data:

- `__input__` for supplying arguments, such as with `@input`.
- `__output__` for returning values, such as with `@output`.
- `__temp__` for intermediate calculations involving NBT, such as resolving strings and iterating over NBT lists.

[function-headers]: #function-headers
[function-annotations]: #function-annotations
[function-annotation-user]: #function-annotation-user
[function-annotation-public]: #function-annotation-public
[function-annotation-api]: #function-annotation-api
[function-annotation-context]: #function-annotation-context
[function-annotation-within]: #function-annotation-within
[function-annotation-handles]: #function-annotation-handles
[function-annotation-patch]: #function-annotation-patch
[function-annotations-input-and-output]: #function-annotations-input-and-output
[function-annotations-input-and-output-for-scoreboard]: #function-annotations-input-and-output-for-scoreboard
[function-annotations-input-and-output-for-storage]: #function-annotations-input-and-output-for-storage
[function-annotations-reads-and-writes]: #function-annotations-reads-and-writes
[storage-recommendations]: #storage-recommendations
[storage-tag-types]: #storage-tag-types
[separate-namespace-for-transient-storage]: #separate-namespace-for-transient-storage
[root-tags-for-transient-storage]: #root-tags-for-transient-storage
