# Architecture

This is a high-level view of the architecture of Moder.

You can imagine Moder as seven distinct main lines, each serving a specific type of use case. Here are the seven lines:

- Tiny Slabs Line: For 1–32 byte allocations.
- Small Slabs Line: For 33–128 byte allocations.
- Medium Slabs Line: For 129–256 byte allocations.
- Big Slabs Line: For 257–512 byte allocations.
- Bigger Slabs Line: For 513–1024 byte allocations.
- Semi-Max Slabs Line: For 1025–2048 byte allocations.
- Max Slabs Line: For 2049–4096 byte allocations.

All sizes are in bytes.

Note: Allocations larger than 4096 bytes are requested from the OS directly (e.g., via VirtualAlloc() on Windows or mmap() on Linux).

Why use slabs with fixed sizes? This approach avoids fragmentation. If a single, giant memory pool were used for all object sizes, the allocator would eventually need to compact the memory, which introduces the problem of dangling pointers.

These are the building blocks of Moder. Its lifecycle consists of three stages:

## Initialization
This stage is called once when you begin working with Moder. It does not necessarily have to occur at the start of your application, as you may initialize Moder mid-execution. It initializes the allocator's metadata and reserves RAM from the OS. This RAM is only reserved and is committed on demand to avoid unnecessary memory usage.

## Life
This stage begins after initialization and ends when deinitialization starts. It consists of multiple calls to various Moder functions, such as mdmalloc(), mdrealloc(), etc. Each function updates Moder’s state and metadata according to the operation performed and commits more RAM when necessary.

Moder also includes internal helper functions that are not exposed to the user. This ensures encapsulation and abstracts implementation details, providing a unified, clean interface for the user.

## Deinitialization
This stage cleans up all data managed by Moder, including the allocator's own internal data and metadata.

While the initialization and deinitialization stages are straightforward, the "Life" stage is the most complex. Memory managers involve a significant amount of metadata that must remain synchronized; doing so efficiently is the primary challenge.

The most critical piece of metadata is the availability of slab slots, which is stored as a bitmap within the slab_t struct. Bitmaps reduce memory overhead and allow for O(1) availability scanning using bitwise operations. Since there are different slab types, a unique struct is required for each.

For example, small slabs can store up to 128 allocations, requiring a single uint128_t variable for the bitmap. Medium slabs store up to 32 allocations, requiring only a uint32_t, and so on.


## overview
The prototype for the small slab struct is as follows:

```cpp
typedef struct {
    uint128_t bitmap;
    void* data;
    bool full, empty;
} small_slab_t;
```
For other types, only the bitmap variable type changes.

When calling mdmalloc(), the allocator finds the next perfect power of two for the requested size. For example, a request for 15 bytes will be assigned a slot that can store up to 32 bytes. While this results in some memory waste, it effectively prevents fragmentation. Once mdmalloc() determines the proper size, it calls the corresponding internal function and returns the result.

However, the returned result is not a standard pointer. It is a "packed" pointer containing the memory address and the slot index within its slab line. Specifically, the last 16 bits of the pointer are used to store the index. This allows operations like realloc and free to be performed efficiently. The tradeoff is that the user cannot dereference this pointer directly. To solve this, a GET_ADDRESS macro is provided to extract the raw, dereferenceable pointer.

This tradeoff was chosen because the alternative—adding a header to each allocation—consumes extra space and breaks memory alignment. Placing a header right before the data would either force the user to manage the header manually (which is unconventional) or require the allocator to fetch it, which compromises alignment.

Another notable tradeoff is that this pointer-packing approach means Moder is incompatible with 32-bit systems. In a 32-bit architecture, all bits are required to address the 4 GB address space, leaving no room for index packing. In 64-bit systems, several bits are typically unused in the virtual address space, providing the necessary room to store the index. While this limits compatibility, it is a reasonable compromise given that modern computing is predominantly 64-bit.
