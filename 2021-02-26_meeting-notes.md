## geo2rdr/rdr2geo

- Computes the intersection between the [range sphere, Doppler cone](https://www.researchgate.net/figure/Description-of-the-distance-sphere-Doppler-cone-and-Doppler-circle_fig5_221906313), and a scattering surface (i.e. the ground) by solving the [Range-Doppler Equation](https://isce-framework.github.io/isce3/overview_geometry.html)
- In `geo2rdr`, you know the target position in map (geodetic or projected + height) coordinates, and want to find the target position in "radar" coordinates (azimuth time & slant range)
- In `rdr2geo`, you know the sensor's position & velocity (via azimuth time), slant range to the target, and Doppler of the target. You wish to find the target's position in map coordinates.
- In both cases, some iterative root-finding algorithm is applied (Newton's method is used in isce3) to approximately solve the Range-Doppler equation.
- [`geo2rdr` in isce3](https://github.com/isce-framework/isce3/blob/0eb175f86f6c74d86c19ba9623f23054513fc657/cxx/isce3/geometry/detail/Geo2Rdr.h)
- [`rdr2geo` in isce3](https://github.com/isce-framework/isce3/blob/0eb175f86f6c74d86c19ba9623f23054513fc657/cxx/isce3/geometry/detail/Rdr2Geo.h)
- Conceptually, geo2rdr & rdr2geo require 5 arguments:
	- **Sensor** - some data structure that encapsulates {Orbit, Doppler, wavelength, LookSide(?)}
	- **Surface** - one of {Ellipsoid, Geoid, "DEMInterpolator", Tangent plane, etc.}
	- **Solver** - some root finding algorithm (Newton's method, Bisection, TOMS 748, Brent's method, ...). The solver typically requires some configuration parameters.
	- **Target position** - Either in radar coordinates (time & range) or in the coordinates which parameterize the surface (lat & lon, easting & northing, x & y, etc.)
	- **(optional) Initial guess**


## Memory resource

- Introduced in C++17 (supported by libstdc++ (gcc >= 9.1), not yet supported by libc++)
- Uses dynamic polymorphism as opposed to templating containers on allocator type
	- Abstract base class [`std::pmr::memory_resouce`](https://en.cppreference.com/w/cpp/memory/memory_resource) defines the interface
	- `std::pmr::polymorphic_allocator` acts as a handle to a memory resource (just a wrapper around a pointer to the memory resource)
- Implementing a memory resource is much easier than writing an allocator
- Implementing a pmr container is much easier than writing an allocator-aware container
- Multi-level containers share the same allocator
	- `std::vector<std::list<std::string>>>`
- Allows for some powerful memory optimizations
- Allows for chaining memory resources
	- For example, you could implement Small String Optimization (SSO) for any object by chaining a stack buffer resource with an upstream heap resource


## DateTime/TimeDelta

- https://github.com/gmgunter/sandbox/tree/datetime-v1/datetime/v1/caesar/datetime

## Documentation

- [standardese](https://github.com/standardese/standardese)
	- Tool for generating C++ API reference documentation automatically from inline comments
	- Goal: nextgen Doxygen with the ability to generate documentation similar to the C++ standard
	- Parses source code using libclang
	- Cons: doesn't have support for LaTeX equations or citations

Possible strategy:
1. Use standardese as a frontend to parse the code and generate *markdown* from the inline comments
2. Use a static site generator (Jekyll, Hugo, etc.) (or Sphinx) to parse the markdown and generate nice-looking HTML and link in MathJax & bibtex support
