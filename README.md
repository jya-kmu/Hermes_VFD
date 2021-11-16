# Hermes_VFD
HDF5 Hermes VFD

## Dependencies
To build the HDF5 Hermes VFD, the following libraries are required:
* [Hermes](https://github.com/HDFGroup/hermes) - A heterogeneous aware, multi-tiered, dynamic, and distributed I/O buffering system that aims to significantly accelerate I/O performance. Make sure to build with HERMES_ENABLE_WRAPPER=ON.
* [HDF5](https://github.com/HDFGroup/hdf5) - HDF5 is built for fast I/O processing and storage. Make sure to build with HDF5_ENABLE_PARALLEL=ON.
* MPI

## Building

### CMake
Hermes VFD makes use of the CMake build system and requires an out of source build.
```
cd /path/to/Hermes_VFD
mkdir build
cd build
ccmake ..
```

Type 'c' to configure until there are no errors, then generate the makefile with 'g'. The default options should suffice for most use cases. In addition, we recommend the following options.

```
-DCMAKE_INSTALL_PREFIX=/installation/prefix
-DHDF5_DIR=/paht/to/hdf5
-DHERMES_DIR=/path/to/hermes
```
After the makefile has been generated, you can type `make -j 2` or `cmake --build . -- -j 2`. Add `VERBOSE=1` to see detailed compiler output.

Assuming that the `CMAKE_INSTALL_PREFIX` has been set and that you have write permissions to the destination directory, you can install the driver by simply doing:
```
make install
```

## Usage
To use the HDF5 Hermes VFD in an HDF5 application, the driver can either be linked into the application, or it can be dynamically loaded as a plugin. If dynamically loading the Hermes VFD, users should ensure that the HDF5_PLUGIN_PATH environment variable points to the directory containing the built VFD library if the VFD has been installed to a non-standard location.

### Method 1: Linked into application
To link the Hermes VFD into an HDF5 application, the application should include the H5FDhermes.h header that gets installed on the system and should link the installed VFD library (libhermes_vfd.so) into the application. Once this has been done, Hermes VFD access can be setup by calling H5Pset_fapl_hermes(...) on a FAPL within the HDF5 application.
