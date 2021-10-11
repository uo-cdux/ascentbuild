Building Ascent such that it allows concurrent development of VTK-m and VTK-h.

As a pre-requisite, checkout Ascent and AMR-Wind

```git
git clone --recursive https://github.com/Alpine-DAV/ascent.git
cd ascent
git submodule init
git submodule update
```

1. You'd perform a build of ascent as usual -- i.e.,
   ascent is going to use it's own VTK-m and VTK-h to start with.
   We're going to override these further.
   Just use the following command from your ascent repository.
   and to begin the build for ascent

```
python scripts/uberenv/uberenv.py --install --prefix install \
                                  --spec %gcc~openmp
```

   Over here you have a choice to specify certain software that you want to exclude from Spack.
   Eg. If you have a `CMake` installation you like and want Spack to use it.
   You'd specify this using a `packages.yaml` file. 

```yaml
packages:
  all:
    variants: ~openmp~python
    compiler: [gcc]
    providers:
      awk: [gawk]
      blas: [openblas]
      daal: [intel-daal]
      elf: [elfutils]
      golang: [gcc]
      java: [jdk]
      lapack: [openblas]
      mpe: [mpe2]
      mpi: [mvapich2]
      pil: [py-pillow]
      szip: [libszip, libaec]
      tbb: [intel-tbb]
      jpeg: [libjpeg-turbo, libjpeg]
      uuid: [util-linux-uuid, libuuid]
  cmake:
    buildable: false
    externals:
    - prefix: /home/users/ayenpure/tools/cmake-3.20.4-linux-x86_64
      spec: cmake@3.20.4
```
and invoke the build command like

```
python scripts/uberenv/uberenv.py --install --prefix install \
                                  --spec %gcc~openmp --spack-config-dir /research/Abhishek/2.0/spack_config/
```
Note: the last parameter here is the path to the directory that contains the `packages.yaml` file.


2. Checkout the version of VTK-m and VTK-h [here](https://ascent.readthedocs.io/en/latest/BuildingAscent.html)
   Build it such that it is not in any of the subdirectory inside your Ascent repository.
   Use the instructions that are mentioned on for manual build for Ascent [here](https://ascent.readthedocs.io/en/latest/BuildingAscent.html#vtk-m-optional-but-recommended)

3. Create a sandbox directory for VTK-m and VTK-h development with Ascent.
   This is a new directory where we are going to re-build Ascent and can be created inside the Ascent repository.
   We need to perform two tasks here.
	a. Create the sandbox directory
	b. Copy the `.cmake` file from your install directory to the sandbox directory
```
mkdir sandbox
cp ../install/alaska-linux-ubuntu18.04-haswell-gcc@7.4.0-ascent-dryv7jcuzehjhxsgqoory5yskw5jsqej.cmake .
```

4. Replace the VTK-m and VTK-h install paths in the `.cmake` file in your sandbox
   This will require changing two variables in this file to relevant locations
```cmake
# vtk-m from spack                                                              
set(VTKM_DIR "/research/Abhishek/2.0/vtk-m/install" CACHE PATH "")              
# vtk-h from spack                                                              
set(VTKH_DIR "/research/Abhishek/2.0/vtk-h/install" CACHE PATH "") 
```
   
5. Finally, we are ready to re-build Ascent in the sandbox directory
```
cmake -C alaska-linux-ubuntu18.04-haswell-gcc@7.4.0-ascent-dryv7jcuzehjhxsgqoory5yskw5jsqej.cmake \
      -DASCENT_INSTALL_PREFIX=/research/Abhishek/2.0/ascent/sandbox/install \
      -DCMAKE_INSTALL_PREFIX=/research/Abhishek/2.0/ascent/sandbox/install  \
      ../src
```
The above command assumes the sandbox directory was created under the Ascent repository.
If not, the path to Ascent source for the `cmake` command will need to be updated.
Once the configuration is successfully completed, you can execute `make -j 4 install` to finish the build and install
Note: We've created another install directory in our sandbox directory, you'll use the Ascent installation from this directory.

6. Checkout and build AMR-Wind with your new Ascent
```
git clone git@github.com:Exawind/amr-wind.git
cd amr-wind
mkdir build
cd build
cmake \
  -DAMR_WIND_ENABLE_TESTS:BOOL=ON  \
  -DAMR_WIND_ENABLE_ASCENT:BOOL=ON \
  -DAscent_DIR:PATH="/research/Abhishek/2.0/ascent/build/ascent-install/lib/cmake/ascent" \
  -DConduit_DIR:PATH="/research/Abhishek/2.0/ascent/build/spack/opt/spack/linux-ubuntu18.04-haswell/gcc-7.4.0/conduit-develop-rnqtrx4bajr4jsrmgqtb3z25ntdnakem" \
  ..
```
Note: Pick the correct Conduit and Ascent paths from your installation.  

7. Anytime you make any changes to VTK-m, you follow the following pipeline to see your changes propogate across the various tools
	a. `make -j 4 install` for VTK-m
	b. `make -j 4 install` for VTK-h
	c. `make -j 4 install` for Ascent
	d. `make` for the application using Ascent (AMR Wind, etc.)
