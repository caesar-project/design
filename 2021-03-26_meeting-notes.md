## Orbit

- Represents the position & velocity of the spacecraft (typically assumed to be the position of the antenna phase center) as a function of time over a closed, continuous interval `[tstart, tstop]`
- Commonly stores a list of position/velocity state vectors -- may be uniformly or non-uniformly sampled
- Some representations might also include acceleration vectors but this isn't used in isce3
- isce3 implements two different interpolation methods -- Third-order Hermite interpolation and Eigth-order Legendre interpolation
	- Hermite interpolation uses velocity info to estimate position. Legendre interpolates position & velocity vectors separately. In practice, Hermite interpolation is always used internally in isce3.
	- In general, there's some redundant computation that can be eliminated by interpolating both position & velocity in a single function call rather than as two separate methods.
- What are the requirements of Orbit?
	- Get start/stop time (and midpoint) of the valid domain
	- Get platform position & velocity for an input datetime `t` if `t` is in `[tstart, tstop]` (isce3 has an option to extrapolate outside this interval)
- Struct-of-Arrays vs Array-of-Structs
	- Store position & velocity together in Nx6 array

```c++
class Orbit {
public:
    // get start/stop/midpoint of Orbit domain
    DateTime start_time() const;
    DateTime end_time() const;
    DateTime mid_time() const;

    // estimate platform position & velocity at time `t`
    std::pair<Vec3, Vec3> operator()(const DateTime& t) const;

private:
    Linspace<DateTime> time_; // uniformly-sampled in time
    std::vector<Vec3> position_;
    std::vector<Vec3> velocity_;
};
```


## Doppler

- Represents the Doppler frequency centroid (first moment) of the raw or processed signal data as a function of azimuth time and range over a closed, continuous interval `{[tstart, tstop], [minrange, maxrange]}`
	- The "Doppler" we use in processing is typically some smooth parametric model which was fit to Doppler estimates from the measured phase history. It can also be inferred from antenna attitude.
- Doppler is most commonly represented by `LUT2d` in isce3. Some functions allow Doppler to be a `LUT1d` or `Poly2d` but this is mostly legacy code.
	- LUT is a bit of a misnomer -- the "lookup" involves interpolating between values stored in the table
	- isce3 supports various 2-D interpolation methods: {sinc, bilinear, bicubic, nearest neighbor, biquintic}. In practice, I think bilinear interpolation is nearly always used for `LUT2d`
- If we split up a long track into multiple Doppler segments, there should be a smooth transition between segments. 
	- Polynomials aren't really favorable for this reason
	- Seems like a 2-D spline would be a good representation -- it's globally smooth to Nth order and locally representable by a low-order polynomial.
- Doppler is used in geometrical calculations (e.g. geo2rdr & rdr2geo), and to represent the azimuth carrier frequency of signal data (e.g. to baseband the data for resampling)
	- The native Doppler depends on the collection geometry (squint angle, antenna steering, vertical velocity, variation in terrain elevation, etc)
	- During processing, the data may be re-skewed to a zero Doppler geometry. SLCs and downstream products commonly use this geometry. Note that this doesn't change the frequency content of the signal data, only the geometry of the grid.
- What are the requirements for Doppler?
	- Check whether a given (azimuth, range) pair is within the valid domain
	- Get Doppler centroid value for an input (azimuth, range) pair, if the input is within the valid domain
	- Get the Doppler gradient w.r.t azimuth & range at a given input (azimuth, range), if the input is within the valid domain. Some data structures may allow for analytical gradient calculation, otherwise default to numerical approximation.
- Since zero Doppler is commonly used, the library should have a convenient, efficient representation for this (e.g. a zeroth-order polynomial)
- There's probably no need for 1-D representations of Doppler
	- This could lead to confusion (is it a function of range or azimuth?)
	- A 2-D data structure with one of the dimensions fixed to 1 should work just as well, and is more explicit

```c++
class LUT2D {
public:
    // get valid azimuth extents of the LUT
    DateTime start_time() const;
    DateTime end_time() const;

    // get valid range extents of the LUT
    double near_range() const;
    double far_range() const;

    // check if the input azimuth,range pair is within the valid domain of the LUT
    bool contains(double azimuth, double range) const;

    // estimate Doppler centroid at the input azimuth & range
    double operator()(double azimuth, double range) const;

    // estimate Doppler gradient at the input azimuth & range
    std::pair<double, double> gradient(double azimuth, double range) const;

private:
	Linspace<DateTime> azimuth_;
	Linspace<double> range_;
	Array2D<double> doppler_;
};
```


## DEMInterpolator

- Represents surface topography as a function of *some parameters* over a closed, continuous 2-D interval
	- The parameters depend on the coordinate system: {(longitude, latitude), (easting, northing), (x, y), ...}
- `DEMInterpolator` in isce3 supports various 2-D interpolation methods: {sinc, bilinear, bicubic, nearest neighbor, biquintic}. Biquintic is the default
- What are the requirements of "Surface"?
	- For a given set of input parameters, check whether they are within the valid domain
	- For a given set of input parameters, if they are within the valid domain, get the location of that point on the surface in Cartesian space (in the spacecraft's coordinate system)
	- For a given set of input parameters, if they are within the valid domain, get the surface normal vector

```c++
class DEMInterpolator {
public:
	// Valid extents
    double xmin() const;
    double xmax() const;
    double ymin() const;
    double ymax() const;

	// Get position of a point on the surface
	Vec3 operator()(double x, double y) const;

	// Get surface normal vector at a point on the surface
	Vec3 normal(double x, double y) const;

private:
    int epsg_; // represents the coordinate system
    Ellipsoid ellipsoid_;
    InterpolationMethod interpmethod_;
    Linspace<double> x_; // or lon_ or east_ ...
    Linspace<double> y_; // or lat_ or north_ ...
    Array2D<double> height_;
};
```


## Vec3 & Quaternion

- Use Eigen or hand-roll our own implementations?
	- Hand-roll:
		- Eigen compile times are pretty slow -- `Vec3` in particular is included throughout most of the library
		- Usage in CUDA code obscures compilation output with a flood of warning messages
	- Eigen:
		- There is additional need for a linear algebra library besides these two classes
		- Potentially exploit SIMD operations for free
		- The best code is code that we don't have to write
- What functionality does the `Vec3` class need?
	- x,y,z coefficient access (get & set)
	- arithmetic (+,-,*,/)
	- norm, squared norm
	- normalize
	- dot product, cross product
	- structured bindings decomposition
- What functionality does the `Quaternion` class need?
	- convert to/from Euler angles & rotation matrix
	- w,x,y,z coefficient access (get & set)
	- scalar (real) & vector component access
	- arithmetic (+,-,*)
	- norm, squared norm
	- normalize
	- conjugate
	- slerp
	- structured bindings decomposition
- "Quaternion" or "UnitQuaternion"?

