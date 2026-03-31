# Architecture

this is a high-level view of the architecture of moder.

you can imagine moder as 4 different main lines, each serves a specific type of use cases, here are the 4 lines:

- small slabs line: this is for 1-32 sized allocations.
- medium slabs line: this is for 33-256 sized allocations.
- big slabs line: this is for 257-1024 sized allocations.
- max slabs line: this is for 1025-4096 sized allocations.
(all sizes in bytes)

these are the building blocks of the moder. and its life cycle consists of 3 stages:

- initialization:
this stage is called once in the very start of your work with moder (it's unnecessary to be in the start of your app, as you may use moder in the middle of your appl), it initalizes the metadata of the allocator and reserves some RAM from the OS, but this RAM is ONLY reserved and will be commited on demand, this is to avoid unnecessary RAM usage.

- life
this stage starts after initaliation stage and ends when deinitialization stage start, it consist of multiple calls to different moder functions, such mdmalloc(), mdrealloc(), etc..
each function is responsible of changing the state and metadata of correponding to the operation its meant to perform, also to commit more RAM when needed.
but moder doesn't only contain those functions the user deal with, there are helper functions moder uses internally but the user doesn't use, this is for encapsulation-like behavior and abstraction of the implementation details of moder, and also to provide a unified, clean interface to deal with moder.

- deinitialization
this stage should clean all the data managed by moder and the data and metadata of moder itself.

initialization/deinitialization stages are pretty simple, but life stage is the hardest, because memory manegers include a lot of data and metadata that should be synced, and syncing them efficiently is the hardest challenge in memory managers (I'll complete this architecture tomorrow, I want to sleep).
