## `std::mdspan`

- `mdspan` represents a non-owning reference to a multidimensional array
	- It is templated on `ElementType`, `Extents`, `LayoutPolicy`, and `AccessorPolicy`
- `Extents` describes the shape of a multidimensional index space, but not its layout in memory
	- Extents classes could have static or dynamic dimensions, or a mix thereof
- `LayoutPolicy` describes the storage ordering of a multidimensional array in memory. It provides a mapping from a multidimensional index space to a linear offset
	- It is a nested class template -- the top-level class is independent of extents. It provides a member type `mapping`, which is templated on `Extents`
	- This allows you to treat `Extents` and `LayoutPolicy` as orthogonal concepts when specializing `mdspan`
	- 3 layouts are provided: `layout_left` (a generalization of row-major), `layout_right` (a generalization of column-major), and `layout_stride` (which allows for user-defined strides)
- `AccessorPolicy` allows for customization of how elements are accessed, allowing for pre-fetching, caching, etc (see [Accessors](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0367r0.pdf))
- Some issues with `mdspan`
	- The API is a bit clunky and verbose in order to allow for some esoteric array layouts (e.g. mix of static & dynamic dimensions, aliasing of elements, etc)
	- Missing a few modern C++ features such as structured bindings and iterators (though you could easily create your own)
	- No "owning" multidimensional array counterpart -- `std::mdarray` is not expected in c++23
- Refs:
	- [mdspan proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0009r10.html)
	- [mdspan paper](https://arxiv.org/pdf/2010.06474.pdf)
	- [kokkos/mdspan](https://github.com/kokkos/mdspan)
	- [mdarray proposal](https://isocpp.org/files/papers/D1684R0.html)


## Multidimensional Arrays

- Reference vs. value semantics
	- What does it mean to copy/assign an ndarray? What does it mean to compare equality between two ndarrays?
	- What if you copy/assign a block from an ndarray?
- Shallow const vs. deep const for reference types
	- Does a *const* reference to *non-const* data have mutable access?
- What size type to use?
	- Use `std::ptrdiff_t` everywhere
- Storage vs Allocator (vs [MemoryResource](https://en.cppreference.com/w/cpp/memory/memory_resource)?)
	- How do nested allocations work? e.g. `pyre::grid<std::string>`


## DateTimes & TimeDeltas

- `DateTime` is a concept, not a class
	- Other time standards besides UTC are used in practice
	- `UTCTime`, `UT1Time`, `GPSTime`, etc.
- On the other hand, `TimeDelta` feels like a concrete type
- Should datetime components (year, month, day, ...) be stored as separate members or combined into a single data member?
	- `DateTime` classes are wrappers around `std::chrono::timepoint`
	- `TimeDelta` is a wrapper around `std::chrono::duration`
- Fixed-point vs floating point?
	- Fixed point -- don't want loss of precision over long intervals
- What resolution is required?
	- Sub-nanosecond?
- A 128-bit int can represent time w/ picosecond resolution over a range of trillions of years (a 64-bit int can represent a range of ~100 days with the same resolution)
	- `absl::int128`
		- Automatically uses a builtin 128-bit int type if the compiler provides one
		- `__device__`-code compatible
