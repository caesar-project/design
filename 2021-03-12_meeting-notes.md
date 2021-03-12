## Vec3

- Do we want to depend on Eigen, or hand-roll this?
- Add `.x()`, `.y()`, `.z()` setters/getters for individual components in addition to (instead of?) array-based access
- `Vec3` should only be used for vectors in Cartesian space (e.g. ECEF, ENU coordinates -- not LLH)


## Orbit

- Need to be able to represent both uniformly-sampled and non-uniformly-sampled orbits
	- Should probably be a template type that stores either a `Linspace` or `vector` of datetimes
- Maybe we should also add classes for simple orbit representations like `LinearOrbit`, `PolynomialOrbit`, `CircularOrbit`?


## Polynomial2D

- Could potentially have a fixed number of coefficients and use the stack, or a dynamic number of coefficients stored on the heap

## DateTimes/TimeDeltas

- Dependencies
	- Howard Hinnant's [`date`](https://github.com/HowardHinnant/date) library for calendar date manipulation handling UTC leap seconds (added to STL in C++20) (currently available through spack/nix, but not conda)
	- Google's [`abseil-cpp`](https://github.com/abseil/abseil-cpp) library for 128-bit integer type (we will vendor the necessary source code under Apache-2.0 license to avoid a dependency on abseil)
	- Uses [`fmt`](https://github.com/fmtlib/fmt) for some string formatting (also added to STL in C++20)

- `TimeDelta`
	- Represents the signed duration between two timepoints w/ fixed-point picosecond resolution
	- Internally, it stores a `std::chrono::duration` -- a 128-bit integer tick count of picoseconds
	- Can be converted to/from any `std::chrono::duration` (no implicit conversion, though). Conversion to lower precision causes truncation.
	- Supports the same (integer-like) arithmetic operations as `std::chrono::duration`
	- How should `operator<<(std::ostream&, TimeDelta)` work?
		- "123456ps" *(integer count of picoseconds)*
		- "1234.56789s" *(total number of decimals seconds)*
		- "1d23h45m67.89s" *(split into components w/ lowercase letter separators)*
		- "P1DT23H45M67.89S" *(ISO 8601 duration format)*
	- The main way of constructing a `TimeDelta` is using its static-method factory functions:
```c++
namespace cs = caesar;

// A TimeDelta representing approximately 1.5 days
const auto dt1 = cs::TimeDelta::days(1.5);

// A TimeDelta representing exactly 1.5 seconds
const auto dt2 = cs::TimeDelta::seconds(1) + cs::TimeDelta::milliseconds(500);

// same as dt2
const auto dt3 = cs::TimeDelta::seconds(1) + 500 * cs::TimeDelta::milliseconds(1);
```
- `GPSTime`
	- 	Represents a time-point in GPS time w/ fixed-point picosecond resolution
	- Internally, it stores a `std::chrono::time_point` -- a 128-bit integer timestamp of picoseconds since some epoch
	- Represents time in a linear, continuous time scale -- unlike `UTCTime`, nearly the entire interface is `constexpr`
	- Can be constructed from date/time components and has getters for individual components
	- Can be converted to/from `std::chrono::time_point` (but not implicitly)
	- Can be converted to/from `std::string` in ISO 8601 format (but not implicitly)
		- TODO: string formatting & parsing similar to [`strftime()` and `strptime()`](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)
	- Should datetimes before year 1 or after year 9999 be valid?
		- If all datetimes that can be represented by `int128` are valid, we would need a wider type than `int` to represent the year component. However, we rely on `date::year` for calendar date conversions. It is always an `int`. 
		- Years below this range are weird -- year 0 is supposed to be 1 BC in the Gregorian calendar.
		- ISO 8601 allows for formatting datetimes outside this range (the number of digits in the year must be >= 4)
		- Protecting this invariant requires a lot more error-handling than would otherwise be the case. This seems particularly burdensome for datetime arithmetic.
		- My current approach is to say that datetimes outside this range are not valid, but it's up to the programmer to ensure that they don't result from datetime arithmetic.

- `UTCTime`
	- Not yet added to the [PR](https://github.com/caesar-project/sandbox/pull/6)
	- Nearly identical to `GPSTime`, except it needs to account for leap seconds
		- Uses the `date` library's functionality to access the system's [IANA timezone database](https://www.iana.org/time-zones) to extract leap second information
		- Constructing a `UTCTime` or breaking it down into components requires a database lookup (the first one is slow, subsequent lookups are fast)
		- However, the representation in memory & doing datetime arithmetic are just as efficient as `GPSTime`
		- If a new leap second is inserted/removed, users will need the most up-to-date IANA database for accurate datetime processing
	- Can be converted to/from `GPSTime` (explicitly)


## Product serialization

- Supporting a wide spectrum of sensors/product types would potentially require bringing in a large number of heavyweight dependencies. How are we going to manage this?
	- NISAR uses hdf5
		- We could add our own bindings for libhdf5 instead of using h5py
	- DopplerScatt uses netcdf
	- Lots of geocoded products use GeoTIFF, VRT, etc (i.e. gdal)
	- Sentinel-1, ALOS use custom, domain-specific file formats
	- NGA uses SICD format
	- Should also look into what private companies like ICEEYE, Capella use
- Bring these in as optional dependencies
	- Separate package for libhdf5 bindings?
	- Sub-packages for NISAR support (which brings in hdf5), sub-package for DopplerScatt (which brings in netcdf5)
	- Use careful import to avoid cryptic runtime link error messages
	- How to package this?
		- All subpackage depencies are required dependencies
			- conda can resolve dependencies for you -- no runtime surprises
			- including the world
		- All subpackage dependencies are optional -- user's responsibility to resolve
		- Subpackages are specified as variants
			- Users must explicitly tell conda what they want
			- conda can resolve the dependencies for you
		- Super-packages (separate packages that depend on caesar) for NISAR, DopplerScatt, etc
			- One github repo -- multiple "pseudo-packages" which are extensions to caesar
			- Is it `import caesar-nisar` or `from caesar import nisar`?


## Future meeting topics?
