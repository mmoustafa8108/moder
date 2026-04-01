# Moder -- managed C

introducing moder, the tool that speeds up your development and takes care of the hard stuff.

moder is a slab-based memory allocator with different slab types for different allocations (e.g. small/medium sized allocations).

in case you're working on a professional-grade performance-critical software system (such as a game engine or web browser), you'll probably spend half of your development time working on boilerplate code, one of the important parts of any professional project is memory managers, as relying on manual memory management is a sure way to memory leaks!
but why writing the same memory manager every new project?
why it's not as easy as `#include "moder.h"`?
well, it's now as easy as it!

with moder, you'll only include 1 file, and get a function rich engine that's not just easy to use, but easy to change, let's be honest.. 'one size fits all' is a lie, and moder doesn't fit all!
but you can fit it to your size easily!
it's designed to be easily changed so it fits 99.99% of use cases, for the reamining 0.01%, it's probably better to make their own memory manager!

## how to build

building moder is as hard as running:
```bash
make static_lib
```
or:
```bash
make dynamic_lib
```

and you get a static/dynamic library ready to use!

## how to use

as noted above, using moder is just a one include statement:

```cpp
#include "moder.h"
```

## functionality

what do you get with that file?
you get a lot:

```cpp
mdmalloc() or malloc()
mdrealloc() or realloc()
mdfree() or free()
mdcalloc() or calloc()
```

so why use moder if it gives me the same functionality as stdlib?
because in stdlib, you deal with the OS, each time you can stdlib version of malloc() there is a lot of layers of overhead added, moder simply reserves a HUGE amount of memory when it's initialized, then dealing with moder's functions is direct roughly no overhead!

## architecture

to understand the core of moder, its architecture, read the [docs](./docs/) prefixed with arch_ (e.g. arch_workflow for high-level workflow view of moder).

## contributions

moder is nothing without its contributors, you're totally welcome to contribute and become part of this project, just read [contributing](./CONTRIBUTING.md) before you make a new issue or pull request to ensure a consistent environment.

## TODOs

these are the features we currently plan to add/refactor in moder:
- actually finish it then speak like a corporation CEO.
