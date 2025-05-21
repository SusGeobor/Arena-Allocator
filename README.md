# Arena-Allocator
C++ Arena allocator which should be pretty fast

Stores 4 x size_t metadata before the allocated block. Allocates in blocks which the user usually manages.

Usage:
```
struct ArenaHeader // metadata struct placed right before pointer returned by createArena
{
    size_t alignment;
    size_t totalSize;
    size_t userSize;
    size_t offset;
};

void* arena = gbr::arena::createArena(400, alignof(int)); // size in bytes and alignment, second parameter is optional, defaults to alignof(size_t)

gbr::arena::destroyArena(arena); // deallocate

gbr::arena::clearArena(arena); // sets arena offset to 0

gbr::arena::stepBackwards<int>(arena, 10); // sets offset backwards sizeof(<int>) x 11

gbr::arena::unsafeStepBackwards<int>(arena, 10); // same as above, but omits out of bounds check

size_t slots = gbr::arena::getFreeSlots<int>(arena); // returns number of <int> sized objects that can fit inside the arena

size_t slots = gbr::arena::multiTypeGetFreeSlots<int>(arena); // returns number of <int> sized objects that can fit inside the arena, considering the alignment of the pointer if the arena holds multiple types

int* ptr = gbr::arena::allocate<int>(arena, 10); // allocates space for 10 <int>, returns nullptr if there is not enough space

int* ptr = gbr::arena::unsafeAllocate<int>(arena, 10); // same as above but omits bounds check

int* ptr = gbr::arena::multiTypeAllocate<int>(arena, 10); // allocates space for 10 <int>, considering the alignment of the pointer if the arena holds multiple types, returns nullptr if there is not enough space

int* ptr = gbr::arena::unsafeMultiTypeAllocate<int>(arena, 10); // same as above but omits bounds check

int* ptr = gbr::arena::construct<int>(arena, 10, 12); // allocates and constructs 10 <int> with the value 12 forwarded to the constructor, returns nullptr if there is not enough space

int* ptr = gbr::arena::unsafeConstruct<int>(arena, 10, 12); // same as above but omits bounds check

int* ptr = gbr::arena::multiTypeConstruct<int>(arena, 10, 12); // allocates and constructs 10 <int> with the value 12 forwarded to the constructor, considering the alignment of the pointer if the arena holds multiple types, returns nullptr if there is not enough space

int* ptr = gbr::arena::unsafeMultiTypeConstruct<int>(arena, 10, 12); // same as above but omits bounds check

gbr::arena::destroy<int>(ptr, 10); // calls dectructors for 10 <int> starting at ptr, useful for non trivial types


// STL Adaptor 1: Stateful Adaptor:

template<class T>
class UnsafeArena : public gbr::arena::stdAllocator<T, no_safety, use_multi_type> {}; // firstly inherit to create your own instance, no_safety and use_multi_type are flags which select from the above functions using constexpr if at compile time

void* arena = gbr::arena::createArena(400, alignof(int)); // create an arena

// create instance of UnsafeArena to allocate integers, use_multi_type allows the STL to convert your allocator to use it with internal datastructures, like linked list nodes
// each stateful allocator holds a void* to the arena
UnsafeArena<int> arenaAllocator{ arena };

{
  std::list<int, UnsafeArena<int>> list(arenaAllocator); // create STL container, pass type into template parameters, and pass instance to constructor
}

gbr::arena::destroyArena(arena); // after STL container exits scope, deallocate arena

// Notes:
// member functions work on internal arena*, deallocate is a no op


// STL Adaptor 1: Static Adaptor:

template<class T>
class UnsafeArena : public gbr::arena::staticAllocator<T, 0, no_safety, use_multi_type> {}; // firstly inherit like before, exactly the same as the previous STL adaptor except the second parameter UID allows you to create multiple instances with the same template parameters

UnsafeArena<int>::createArena(400); // does not return arena, instead sets a static void*, takes count of <int> to allocate rather than number of bytes

{
  std::list<int, UnsafeArena<int>> list; // pass the type into the template parameters, does not require instance in constructor as the allocator has static state and a trivial default constructor
}

UnsafeArena<int>::destroyArena(); // deallocate

size_t UID = UnsafeArena<int>::getUID(); // returns UID

// Notes:
// member functions are static and work on static arena*, deallocate is a no op
```
