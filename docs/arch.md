# Architecture

this is a high-level view of the architecture of moder.

you can imagine moder as 7 different main lines, each serves a specific type of use cases, here are the 7 lines:

- tiny slabs line: this is for 1-32 sized allocations.
- small slabs line: this is for 33-128 sized allocations.
- medium slabs line: this is for 129-256 sized allocations.
- big slabs line: this is for 257-512 sized allocations.
- bigger slabs line: this is for 513-1024 sized allocations.
- semi max slabs line: this is for 1025-2048 sized allocations.
- max slabs line: this is for 2049-4096 sized allocations.
**allocations greater than 4096 bytes shouldn't be handled with the slab system.**
**all size in bytes.**


why we used slabs with fixed sizes? to avoid fragmentation, because if I used a giant memory pool for all sizes of object I'll have to compact the memory at some point which will introduce the dangling pointers problem.

these are the building blocks of the moder. and its life cycle consists of 3 stages:

- initialization:
this stage is called once in the very start of your work with moder (it's unnecessary to be in the start of your app, as you may use moder in the middle of your appl), it initalizes the metadata of the allocator and reserves some RAM from the OS, but this RAM is ONLY reserved and will be commited on demand, this is to avoid unnecessary RAM usage.

- life
this stage starts after initaliation stage and ends when deinitialization stage start, it consist of multiple calls to different moder functions, such mdmalloc(), mdrealloc(), etc..
each function is responsible of changing the state and metadata of moder correponding to the operation its meant to perform, also to commit more RAM when needed.
but moder doesn't only contain those functions the user deal with, there are helper functions moder uses internally but the user doesn't use, this is for encapsulation-like behavior and abstraction of the implementation details of moder, and also to provide a unified, clean interface to deal with moder.

- deinitialization
this stage should clean all the data managed by moder and the data and metadata of moder itself.

initialization/deinitialization stages are pretty simple, but life stage is the hardest, because memory manegers include a lot of data and metadata that should be synced, and syncing them efficiently is the hardest challenge in memory managers.


pehaps the most important metadata is the availability of slabs slots which is stored as a bitmap in the `slab_t` struct itself, bitmaps will reduce size and allow O(1) availability scanning using bit wise operations, but since we have different slab types, we'll need different struct for each type.

for example, the small slabs can store up to 128 allocations, so we need one single `uint128_t` variable as the bitmap of this type of slabs,
but medium slabs can store up to 32 allocations, so we'll need just one `uint32_t` for the whole bitmap and so on..

the prototype for small slabs struct is like:
```cpp
typedef struct {
    uint128_t bitmap;
    void* data;
    bool full, empty;
} small_slab_t;
```

and for other types just change the type of bitmap variable.

when calling mdmalloc, we'll see the next-perfect power of 2 for the size you requested, for example if you requested 15 bytes you'll actually get a slot that can store up to 32 bytes, this is memory waste but to avoid fragmentation..
after mdmalloc knows the proper size it'll call the corresponding function and return its reuslt back to the user.
but that result isn't actually a valid pointer, it's the pointer + its index in its slab line packed into it, the last 3 bits of the pointer are used to pack itd index.. this is to perform operations such as realloc and free on the pointer effeciently with the tradeoff that the user will not be able to directly dereference the pointer, in order to solve this problem there is a macro (GET_ADDRESS) that takes the packed pointer and returns the raw pointer that can be dereferenced.

I accepted this tradeoff because if I didn't, I'd need to add a header to each allocation which takes space and kills alignment (if I put the header right before the data the pointer points to) and I'd either need to give the user a header, which isn't conventional or give the user a pointer and fetch the header myself, which kills alignment as I said.

there is another tradeoff worth noting, pointer packing approach means moder will not work with 32-bit systems, because in 32-bit systems all the bits are used to address up to 4 GB so there is no free bits to use... in 64-bit system the last 3 bits aren't used as far as I know so we'll have some space to put the index.

I'm not sure if there is a solution to this tradeoff else than not supporting 32-bit systems (which isn't that bad, world is mostly running on 64-bit machines now).
