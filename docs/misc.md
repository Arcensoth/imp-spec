# IMP: Miscellaneous

> **This document is outdated and needs to be re-worked.** It is currently out-of-date and has not been adjusted to reflect the recent changes made to the IMP standards and other documents.

## Pack types

- [Modules](#modules)
- [Patches](#patches)
- [Bundles](#bundles)

### Modules

Modules are the building blocks of datapack development: single, self-contained, namespaced packages of data and logic that may or may not be dependent on other modules.

In general, modules should never overwrite anything from other modules, and all resources within a module should share the same namespace. Any other namespaces within the pack should contain only definitions for tags, such as the vanilla `minecraft:tick` and `minecraft:load` or those of other packs. The exception to this rule is for types of data that cannot be appended and instead must be overwritten entirely, such as the vanilla loot tables.

A module is not necessarily operational on its own and may be dependent on other modules. Because of this, it is not recommended to distribute lone modules to the end user. Module dependencies can be supplied by either:

1. providing the required modules separately to the world, for example during development with git and/or symlinks; or
2. distributing the module [bundled with its dependencies](#bundles), for something closer to drag-and-drop installation.

### Patches

If modules are the building blocks, patches are the glue that holds them together: special mini-packs designed specifically to merge and/or configure behaviour of one or more other modules. They can be used to bridge two or more modules and make them compatible, or to switch-out a particular component of one module with a different implementation.

It's not always easy to distinguish a patch from a module. In general, patches are small and meant specifically to bridge compatibility or configure behaviour, in cases where modifying the original pack is either not possible or not ideal.

Patches generally use the naming scheme `<module1>+<module2>`, or `<module>+<option>` for overwriting the behaviour of a single module, however these naming conventions are not required. If more than two modules are involved in the patch (as in [bundles](#bundles)) it may be more appropriate to simply name it something along the lines of `<bundlename>+patch` `<bundlename>+compat`.

Some examples of where to use a patch:

- For bridging compatibility. Two modules `foo` and `bar` want to overwrite the vanilla zombie loot table `minecraft:entities/zombie`. Without intervention, only one of these modules will claim the zombie loot table and the other will be left out entirely. We can create a patch pack `foo+bar` that redefines these loot tables to support both modules. To do this, we can either (1) merge both loot tables into a new `minecraft:entities/zombie` or (2) keep them separate and include both in `minecraft:entities/zombie`, like so:
  - Copy `minecraft:entities/zombie` from `foo` into `foo+bar:foo/zombie`
  - Copy `minecraft:entities/zombie` from `bar` into `foo+bar:bar/zombie`
  - Overwrite `minecraft:entities/zombie` to include the former two table as separate pools
- For configuring behaviour. The module `herobrine` implements a pathfinding system for a custom boss fight. By default the module uses an unoptimized pathfinding algorithm that causes noticeable lag on weaker machines. A patch `herobrine+betterpath` could be created, to directly replace the function files in the original `herobrine` module with an optimized version of the pathfinding algorithm.

### Bundles

A bundle is more like what you'd expect from a typical datapack: a bunch of folders and files that do stuff. Here, however, a bundle is specifically a collection of [modules](#modules) and [patches](#patches) all working together to create a cohesive, distributable, IMP-compliant datapack.

Since bundles are comprised of modules and all of the [standards and guidelines](#standards-and-guidelines) still apply, not all datapacks are IMP-compliant bundles.

Some examples of where to use a bundle:

- Distributing a [module](#modules) with its dependencies. Use a bundle to package the module along with all of its dependencies (and their dependencies, etc) so that the end user receives it in working order.
- Distributing a complete datapack comprised of several modules. Perhaps a curated collection of modules that work together to create a game-changing survival experience. Make liberal use of [patches](patches) when conflicting modules need to be told how to play nicely with one another. (This type of datapack is the closest thing we have to a modpack in vanilla.)
