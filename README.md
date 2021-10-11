# Prerequisite
As a pre-requisite, checkout Ascent and AMR-Wind

```bat
[Ascent]
git clone --recursive https://github.com/Alpine-DAV/ascent.git
cd ascent
git submodule init
git submodule update


[AMR Wind]
git clone --recursive https://github.com/exawind/amr-wind.git
```

# Building Ascent w/ AMR Wind

The steps listed below are meant for building Ascent and all it's dependencies using Spack

## Reference Directory Structure
*Base Directory* </br>
|-*amr-wind* </br>
|&emsp;|-*build*</br>
|-ascent</br>
|&emsp;|-*install*</br>

## Steps

1. Perform a build of Ascent as usual

    ```bat
    [ascent]
    python scripts/uberenv/uberenv.py \
        --install --prefix "install" \
        --spec "%gcc~openmp"
    ```
    The spec parameter above asks Spack to use `gcc`as a compiler and not to build with `OpemMP` 

2. Perform a build of AMR-Wind, pay attention to the paths provided for Ascent and Conduit.</br>
    Conduit is built by Spack, and all depdenencies are stored in the paths following the </br>
    `<ascent install>/spack/opt/spack/<platform>/<compilter>/<package>` pattern

    ```bat
    [amr-wind/build]
    cmake \
      -DAMR_WIND_ENABLE_TESTS:BOOL=ON  \
      -DAMR_WIND_ENABLE_ASCENT:BOOL=ON \
      -DAscent_DIR:PATH="/research/Abhishek/2.0/ascent/install/ascent-install/lib/cmake/ascent" \
      -DConduit_DIR:PATH="/research/Abhishek/2.0/ascent/install/spack/opt/spack/linux-ubuntu18.04-haswell/gcc-7.4.0/conduit-develop-rnqtrx4bajr4jsrmgqtb3z25ntdnakem" \
      ..
    ```
    
    You might be required to use the same `cmake` that was used to build Ascent.</br>
    In that case find cmake in the same way how conduit was provided in the above command.
    
    ```bat
    [amr-wind/build]
    /research/Abhishek/2.0/ascent/install/spack/opt/spack/linux-ubuntu18.04-haswell/gcc-7.4.0/cmake-3.20.2-uinuzp7bedqoujagywqjdov5rzpu4uld/bin/cmake \
      -DAMR_WIND_ENABLE_TESTS:BOOL=ON  \
      -DAMR_WIND_ENABLE_ASCENT:BOOL=ON \
      -DAscent_DIR:PATH="/research/Abhishek/2.0/ascent/install/ascent-install/lib/cmake/ascent" \
      -DConduit_DIR:PATH="/research/Abhishek/2.0/ascent/install/spack/opt/spack/linux-ubuntu18.04-haswell/gcc-7.4.0/conduit-develop-rnqtrx4bajr4jsrmgqtb3z25ntdnakem" \
      ..
    ```


# Building Ascent : Concurrent development of VTK-m and VTK-h

## Reference Directory Structure
*Base Directory* </br>
|-*amr-wind* </br>
|&emsp;|-*build*</br>
|-ascent</br>
|&emsp;|-*install*</br>
|&emsp;|-*sandbox*</br>
|&emsp;|&emsp;|-*install*</br>
|-*vtk-h*</br>
|&emsp;|-*build*</br>
|&emsp;|-*install*</br>
|-*vtk-m*</br>
|&emsp;|-*build*</br>
|&emsp;|-*install*</br>

## Steps

1. You'd perform a build of ascent as usual -- i.e., ascent is going to use it's own VTK-m and VTK-h to start with.</br>
   We're going to override these further. 
   Just use the following command from your Ascent repository and to begin the build for ascent.

    ```bat
    [ascent]
    python scripts/uberenv/uberenv.py \
    --install --prefix "install" \
    --spec "%gcc~openmp"
    ```

   Over here you have a choice to specify certain software that you want to exclude from Spack.</br>
   Eg. If you have a `cmake` installation you like and want Spack to use it,
   you'd specify this using a `packages.yaml` file.</br>
   Following are the contents of a `packages.yaml` file that uses a user specified `cmake`

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
    
    ```bat
    [ascent]
    python scripts/uberenv/uberenv.py
        --install --prefix "install" \
        --spec "%gcc~openmp" \
        --spack-config-dir /research/Abhishek/2.0/spack_config/
    ```
    **Note: the last parameter here is the path to the directory that contains the `packages.yaml` file.**

2. Create a sandbox directory for VTK-m and VTK-h development with Ascent.</br>
    This is a new directory where we are going to re-build Ascent and can be created inside the Ascent repository.<br>
    We need to perform two tasks here:</br>
    1. Create the sandbox directory</br>
    2. Copy the `.cmake` file from your install directory to the sandbox directory</br>
    ```bat
    [ascent]
    mkdir sandbox
    cp ../install/alaska-linux-ubuntu18.04-haswell-gcc@7.4.0-ascent-dryv7jcuzehjhxsgqoory5yskw5jsqej.cmake .
    ```
    
3. Checkout the version of VTK-m and VTK-h listed [here](https://ascent.readthedocs.io/en/latest/BuildingAscent.html)</br>
    Use the instructions that are mentioned on for manual build for Ascent [here](https://ascent.readthedocs.io/en/latest/BuildingAscent.html#vtk-m-optional-but-recommended)

    If you end up chosing incompatible versions of VTK-m or VTK-h, it'll upset Ascent.</br>
    The git hashes provided on the webpage are meant to work together.</br>
    ```bat
    [vtk-m/build]
    cmake -DCMAKE_INSTALL_PREFIX=/research/Abhishek/2.0/vtk-m/install \
      -DCMAKE_BUILD_TYPE=Release \
      -DVTKm_USE_64BIT_IDS=OFF \
      -DVTKm_USE_DOUBLE_PRECISION=ON \
      -DVTKm_USE_DEFAULT_TYPES_FOR_ASCENT=ON \
      -DVTKm_NO_DEPRECATED_VIRTUAL=ON \
      -DVTKm_ENABLE_TESTING=OFF \
      -DBUILD_TESTING=OFF \
      ..
      
      make -j 4 install
    ```
    
    VTK-h requires other dependencies too, not just VTK-m, some of them are already built by ascent.</br>
    To make VTK-m re-use all the packages that have been previously built by ascent, </br>
    we point it to the `.cmake` file we copied to our sandbox earlier. </br>
    But first we need to update the VTK-m pointed by this file by editing the relevant line </br>
    ```
    set(VTKM_DIR "/research/Abhishek/2.0/vtk-m/install" CACHE PATH "") 
    ```
    Now, VTK-h can be built using the following commang

    ```bat
    [vtk-h/build]
    cmake -C /research/Abhishek/2.0/ascent/sandbox/alaska-linux-ubuntu18.04-haswell-gcc@7.4.0-ascent-dryv7jcuzehjhxsgqoory5yskw5jsqej.cmake  \
        -DCMAKE_INSTALL_PREFIX=/research/Abhishek/2.0/vtk-h/install \
     ../src
     
     make -j 4 install
    ```
    After VTK-h has finished installing, update it's path in the `.cmake` file in the sandbox as well.
    ```
    set(VTKH_DIR "/research/Abhishek/2.0/vtk-h/install" CACHE PATH "") 
    ```
    
4. Finally, we are ready to re-build Ascent in the sandbox directory</br>
    Once again, make sure the path to VTK-m and VTK-h are updated in the `.cmake` file that we copied to the sandbox
	```bat
	[ascent/sandbox]
	cmake -C alaska-linux-ubuntu18.04-haswell-gcc@7.4.0-ascent-dryv7jcuzehjhxsgqoory5yskw5jsqej.cmake \
	      -DASCENT_INSTALL_PREFIX=/research/Abhishek/2.0/ascent/sandbox/install \
	      -DCMAKE_INSTALL_PREFIX=/research/Abhishek/2.0/ascent/sandbox/install  \
	      ../src
	```
	Once the configuration is successfully completed, you can execute `make -j 4 install` to finish the build and install</br>
	**Note: We've created another install directory in our sandbox directory.</br> For AMR-Wind you'll use the Ascent installation from this directory.**

5. Checkout and build AMR-Wind with your new Ascent
	```bat
	[amr-wind/build]
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

6. Anytime you make any changes to VTK-m,</br> you follow the following pipeline to see your changes propogate across the various tools
	1. `make -j 4 install` for VTK-m
	2. `make -j 4 install` for VTK-h
	3. `make -j 4 install` for Ascent
	4. `make` for the application using Ascent (AMR Wind, etc.)
