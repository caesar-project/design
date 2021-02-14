## Error Handling

`ErrorCode` - A type-erased error code class. Allows interoperability between various error code enumeration types.

`Error` - Stores an `ErrorCode` along with source location info about where the error originated (filename and line #).

`Expected<T>` - A wrapper that may contain an object of type `T` or an `Error`. Intended to be used as a return type for operations that may fail.

- [v1](https://github.com/caesar-project/sandbox/pull/3)
  - Uses dynamic polymorphism to implement type erasure in `ErrorCode` (not fully CUDA compatible).
  - Uses [`boost::outcome`](https://ned14.github.io/outcome/) as the basis for `Expected<T>`.
- [v2](https://github.com/caesar-project/sandbox/pull/4)
  -  Uses a variant/visitor idiom to implement type erasure in `ErrorCode`.
  - Uses [`tl::expected`](https://github.com/TartanLlama/expected) as the basis for `Expected<T>`.



## Unit Testing

- Testing framework -- are we all happy with googletest?
	- Catch2 v3
	- doctest
- Common pattern in isce3 of bugs arising from the unit test suite -- Unit tests are run in parallel and all write their output to the same scratch directory. Output filename collisions may result in a data race. How to avoid this by design?
- It would be nice to have a library of common testing utils similar to [numpy.testing](https://numpy.org/doc/stable/reference/routines.testing.html) (e.g. `max_abs_error()`, `assert_all_equal()`)
- CMake googletest auto-discovery tool -- https://cmake.org/cmake/help/latest/module/GoogleTest.html


## Documentation

- [doxygen](https://www.doxygen.nl/index.html)
- [standardese](https://github.com/standardese/standardese)
	- Similar to doxygen with some more modern features
	- Inspired by cppreference documentation
	- No support for LaTeX equations AFAICT
- [pybind11_mkdoc](https://github.com/pybind/pybind11_mkdoc)
	- Automatically converts C++ comments into python docstrings
	- Useful to avoid copy&paste in pybind11 docstrings but not fully-featured for C++ documentation


## Array data structures

- `pyre::grid` walkthrough
- `mdspan` walkthrough
