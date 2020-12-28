## Directory layout

- To whatever extent possible, we should provide familiar entry points for each language we support
	- e.g. the python package should be built with `setup.py` and tested with `pytest`, python code should be in a familiar location
- Should be scalable to support bindings for multiple additional languages without adding undue complexity
- Deep, complicated directory layouts are cumbersome to navigate
- [llvm-project](https://github.com/llvm/llvm-project) - a good example of a single repository containing multiple independent sub-projects


## Naming conventions

- **UPPER_CASE**
	- Preprocessor macros (`#define CAESAR_VERSION_MAJOR 0`)
- **CamelCase**
	- Classes, Structs (`class BartlettKernel`)
	- Enums (`enum class LookSide`)
	- Enum constants (`LookSide::Left`)
	- Template parameters, Concepts (`template<class ForwardIterator>`)
- **lower_case**
	- Variables (including member variables) (`double speed_of_light = 3e8`)
	- Functions (including member functions) (`void calc_phase_gradient(...)`) <sup>[1]</sup>
	- Namespaces (`namespace caesar`)
	- Filenames (`bilinear_interpolator.cpp`) <sup>[2]</sup>
- **camelBack**
	- N/A
- What naming convention to use for private member variables?
	- Use underscore suffix (i.e. `{variable}_`)
- What about class names that include an acronym/initialism (e.g. LUT, SLC)
	- If it's an *acronym* (you pronounce it as a word), then use one uppercase letter followed by lowercase letters (e.g. `NasaEmployee`)
	- If it's an *initialism* (you pronounce it as individual letters), then each letter should be uppercase (e.g. `JPLEmployee`)

<sup>[1]</sup> For consistency w/ PEP 8 - allows for direct mapping between C++ names & Python names

<sup>[2]</sup> For consistency w/ PEP 8 and makes command line navigation more ergonomic


## Other conventions

### File extensions

- Distinguish between C++, CUDA header files
	- .hpp/.cpp <-- C++ code only, may *not* `#include` .cuh files
	- .cuh/.cu <-- C++/CUDA code, ~~may `#include` .hpp files~~ (this is potentially more complicated than anticipated - need to consider that valid C++ may not be supported by nvcc)
- No .icc files - put all inline code in the header file

### Header/source file pairs

- Each header file should have an associated source file that includes it in its first line, even if the source file is otherwise empty

### Namespaces

- There should be a single top-level namespace - no hierarchical nested namespaces <sup>[3]</sup> 

<sup>[3]</sup> Except for the `caesar::detail` namespace for inline code that should be private


## Compilers

- Maintain support for conda-forge's [compiler toolchain](https://conda-forge.org/docs/maintainer/infrastructure.html#compilers-and-runtimes) (and the associated standard library implementations)
	- MacOS - clang v11 (Q: which clang is this? AppleClang or open source clang?)
	- Linux - gcc v9
- Design for C++20
	- Use third-party implementations of C++20 library features (e.g. [fmt](https://github.com/fmtlib/fmt), [range-v3](https://github.com/ericniebler/range-v3), [date](https://github.com/HowardHinnant/date))
	- Use widely-supported non-standard language features (e.g. `std::experimental::source_location`)
- If possible, use a consistent C++ standard in both C++ & CUDA code
- Support Windows (MSVC)?
	- (Tentatively) yes, but unofficially. We'll try to avoid deliberately doing anything that would break Windows support, but won't provide CI and will drop support if it becomes too painful


## Error handling

- Avoid use of exceptions
	- Bad path is expensive
	- Not available in CUDA code, unhandled in multi-threaded code
- Error codes have issues too
	- Easy to ignore
	- Error handling is verbose, errors need to be propagated manually
	- All possible error codes from the program & third-party libraries must conform to a common type and not overlap in value

```c++
/** A type-erased error code */
class ErrorCode {
    const ErrorCategory* category_;
    int value_;

public:
    template<class EC>
    ErrorCode(EC ec);

    const ErrorCategory* category() const;
    
    int value() const;

    std::string_view description() const { return category()->describe(value()); }
};
```

```c++
class Error {
    ErrorCode errorcode_;
    int line_;
    const char* file_;

public:
    Error(const ErrorCode& errorcode, 
          const std::source_location& sl = std::source_location::current());

    const ErrorCode& errorcode() const;

    int line() const;

    std::string_view file() const;

    void throw_exception() const { errorcode().category()->throw_exception(*this); }
};
```

```c++
/** Stores an optional value of type T or an Error */
template<class T>
class Expected {
    union {
        T value_;
        Error error_;
    };
    bool has_value_;
    
public:
    Expected(T&& value);

    Expected(const Error& error);

    /** Calls error().throw_exception() if a value was not stored! */
    const T& value() const;

    const Error& error() const;

    bool has_value() const;
};
```


## Array data structures

- Requirements
	- N-dimensional arrays (where `N` is a template parameter)
	- Custom memory layouts
	- Custom allocators
	- Non-owning view/reference type for interoperability with numpy and use in CUDA kernels
	- (Expression templates?)
- Some possibilities
	- [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page)
		- Extremely slow compilation
		- Only supports up-to-2D
		- Doesn't support custom memory layouts
		- Doesn't support custom allocators
		- Not constexpr
		- Expression templates only support a limited subset of possible use cases
	- [xtensor](https://xtensor.readthedocs.io/en/latest/)
		- Shape uses std::vector
	- pyre::grid
		- Need to design/maintain it ourselves
	- [mdspan](https://github.com/kokkos/mdspan) / mdarray
