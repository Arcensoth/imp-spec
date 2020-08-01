# IMP: Core Standards

These are the bare minimum standards for a datapack module to be considered IMP-compliant.

- [1. Leave vanilla alone](#1-leave-vanilla-alone)
- [2. Namespace everything](#2-namespace-everything)
  - [2a. Namespace entity tags](#2a-namespace-entity-tags)
  - [2b. Namespace custom item tags](#2b-namespace-custom-item-tags)
  - [2c. Namespace scoreboard objectives](#2c-namespace-scoreboard-objectives)
  - [2d. Namespace NBT storage locations](#2d-namespace-NBT-storage-locations)

## 1. Leave vanilla alone

Definition:

- Avoid changing anything under the vanilla `minecraft` namespace, where possible.

Rationale:

- Avoids causing confusion over what is provided by vanilla.
- Avoids the risk of unintentionally inheriting/overwriting future behaviour.

Caveats:

- Some data cannot be extended and must be overwritten to change functionality, such as the vanilla loot tables. In these unfortunate cases it may be necessary to overwrite data. This is acceptable so long as it is clearly documented.
- Tagging your functions/blocks/etc with vanilla tags is perfectly acceptable (and necessary for certain functionality), but replacing them (via `"replace": true`) is almost always a very bad thing to do.

Recommendations:

- In cases where vanilla data must be overwritten (such as loot tables), attempt to abstract the changes as clearly as possible.
  - For example, with loot tables: instead of writing your custom loot in the vanilla loot table directly, write a custom loot table under your namespace and include it as a single entry in the vanilla loot table. This makes it easier to adapt in the future and provides a clean way for other packs to account for.
- Overall, treat the vanilla namespace as you would any other pack. Be courteous of other pack authors, and keep in mind that your pack may not necessarily be the only one running.

## 2. Namespace everything

### 2a. Namespace entity tags

Definition:

- All entity tags are prefixed with and delimited by the module namespace.
- (Note: entity tags are those assigned to an entity with the `tag` command and stored inside its `Tags` NBT.)

Rationale:

- Effectively eliminates the chance of entity tag conflicts with other compliant modules.
- Drastically reduces the chance of entity tag conflicts with packs that do not follow any standards.

Caveats:

- The character `:` is not supported in all the same places tags are and thus cannot be used as a namespace delimiter.

Recommendations:

- Use `.` as the delimiter for both the namespace and different levels of tags, and use `_` for separating words (snake_case).
- Be verbose with your tags. Do not sacrifice your project's readability in the never-ending struggle for micro-optimization. If performance ever becomes a concern, run your pack through a minimizer and distribute the minimized version to your end users.

Examples:

- `playtracker.sneaking`
- `playtracker.sneaking.stopped`
- `playtracker.talked_to_villager`

### 2b. Namespace custom item tags

Definition:

- All custom item tags are contained inside an NBT compound that is named after and dedicated to the module.
- The dedicated NBT compound is prefixed or exists within another compound with a naming convention that does not conflict with vanilla.
- (Note: custom item tags are any non-vanilla tags stored within an item's `tag` NBT.)

Rationale:

- Effectively eliminates the chance of custom item tag conflicts with other compliant modules.
- Drastically reduces the chance of custom item tag conflicts with packs that do not follow any standards.
- Drastically reduces the chance of custom item tag conflicts with any current and/or future vanilla NBT.

Caveats:

- The `CustomModelData` tag is a vanilla tag and is not part of this specification.

Recommendations:

- Prefix your NBT compound with an underscore `_<namespace>` or keep it within another custom compound `__custom__.<namespace>` to help ensure that it remains separate from vanilla tags.

Examples:

- Bad: `{tag: {fire_wand: true}}`
- Bad: `{tag: {_fire_wand: true}}`
- Bad: `{tag: {mystuff: {fire_wand: true}}}`
- Good: `{tag: {_mystuff: {fire_wand: true}}}`
- Good: `{tag: {__custom__: {mystuff: {fire_wand: true}}}}`

### 2c. Namespace scoreboard objectives

Definition:

- All scoreboard objectives are prefixed with a _scorespace_: an abbreviated version of the module namespace that accounts for the 16-character limit on objective names.

Rationale:

- Effectively eliminates\* the chance of scoreboard objective conflicts with other compliant modules.
- Drastically reduces the chance of scoreboard objective conflicts with packs that do not follow any standards.

Caveats:

- \*Because the names of scoreboard objectives are limited in length, the number of possible scorespaces is much smaller than the number of possible namespaces. Therefore, the chance of a collision between two scorespaces is higher than it would be with the full namespaces.

Recommendations:

- Use a scorespace that is easily associated with the full namespace of the module. Avoid abbreviations that are too short or generic.

Examples:

- `ptrak.deathtime`
- `ptrak.sincerest`
- `ptrak.usebow`

### 2d. Namespace NBT storage locations

Definition:

- All NBT storage locations use the module namespace as part of their own namespace.

Rationale:

- Effectively eliminates the chance of NBT storage conflicts with other compliant modules.
- Drastically reduces the chance of NBT storage conflicts with packs that do not follow any standards.

Recommendations:

- Use the same namespace as your module for any persistent data, such as configuration options.
- Use a separate namespace based on the original namespace (such as `<namespace>.__temp__`) for any temporary/transient data.
  - Reduces the risk of bloating and potentially corrupting the main storage.
  - Makes it easy to recognize which `.nbt` files can be safely purged.
  - Simplifies the process of debugging each storage individually.

Examples:

- `playtracker:config`
- `playtracker.__temp__:player/on_sneak`
