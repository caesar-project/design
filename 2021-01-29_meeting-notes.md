## Array data structures

- `pyre::grid`
- `kokkos::mdspan`
- `Eigen::Tensor`
- `xtensor::xtensor`

## pyre::grid

- memory -- Heap, FileMap, View, ...
- pyre::view -- non-owning
- separate storage from layout

## kokkos::mdspan

- index/size type is `std::ptrdiff_t`
- templated on Extents, Layout, and Accessor
	- Extents describes the shape, independent of memory layout
	- Layout describes the mapping from multidimensional index to linear offset
	- Accessor allows for customization of how elements are accessed, allowing for pre-fetching, caching etc (see [Accessors](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0367r0.pdf))


## Eigen::Tensor

- We probably need an Eigen dependency anyway for linear algebra `¯\_(ツ)_/¯`

