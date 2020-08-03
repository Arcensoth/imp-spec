# IMP Conventions & Recommendations (IMP-Con)

- [Documentation format][documentation-format]
- [Storage][storage]
  - [Storage tag types][storage-tag-types]
  - [Separate namespace for transient storage][separate-namespace-for-transient-storage]
  - [Root tags for transient storage][root-tags-for-transient-storage]

## Documentation format

The [IMP Documentation Format](./docs/imp_doc.md) is recommended for maintaining well-documented functions.

## Storage

The following recommendations apply when using storage for temporary/transient data:

1. The storage name should have a 1:1 correlation to the function name.
   - Example: `storage mypack:utils/get_full_name`
   - This helps to avoid ambiguity and makes it clear where each storage is being created.
   - It also helps to avoid one function from accidentally accessing the data of another.
2. Use a [separate namespace][separate-namespace-for-transient-storage] ending in `.__temp__` for all transient storage.
   - Example: `storage mypack.__temp__:utils/get_full_name`
3. Due to the limitations of selecting root NBT paths, it is also recommended to use separate [transient root tags][root-tags-for-transient-storage] for input vs output data.

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

[documentation-format]: #documentation-format
[storage]: #storage
[storage-tag-types]: #storage-tag-types
[separate-namespace-for-transient-storage]: #separate-namespace-for-transient-storage
[root-tags-for-transient-storage]: #root-tags-for-transient-storage
