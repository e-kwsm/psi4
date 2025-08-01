    if(${ENABLE_PCMSolver})
    find_package(PCMSolver 1.2.3 CONFIG QUIET)  # edit in codedeps

    if(${PCMSolver_FOUND})
        get_property(_loc TARGET PCMSolver::pcm PROPERTY LOCATION)
        message(STATUS "${Cyan}Found PCMSolver${ColourReset}: ${_loc} (found version ${PCMSolver_VERSION})")
        add_library(pcmsolver_external INTERFACE)  # dummy

        if(${CMAKE_INSIST_FIND_PACKAGE_PCMSolver})
            message(VERBOSE "Suitable PCMSolver located externally as user insists")
        endif()

    else()
        if(${CMAKE_INSIST_FIND_PACKAGE_PCMSolver})
            message(FATAL_ERROR "Suitable PCMSolver could not be externally located as user insists")
        endif()

        include(ExternalProject)
        message(STATUS "Suitable PCMSolver could not be located, ${Magenta}Building PCMSolver${ColourReset} instead.")

        if(${BUILD_SHARED_LIBS})
            set(_a_only  OFF)
            set(_so_only ON)
        else()
            set(_a_only  ON)
            set(_so_only OFF)
        endif()

        set(_pcmsolver_dir "share/cmake/PCMSolver")

        ExternalProject_Add(pcmsolver_external
            URL https://github.com/loriab/pcmsolver/archive/v123_plus_ming.tar.gz  # edit in codedeps
            UPDATE_COMMAND ""
            CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${STAGED_INSTALL_PREFIX}
                       -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
                       -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
                       -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
                       -DCMAKE_Fortran_COMPILER=${CMAKE_Fortran_COMPILER}
                       -DCMAKE_INSTALL_LIBDIR=${CMAKE_INSTALL_LIBDIR}
                       -DCMAKE_INSTALL_BINDIR=${CMAKE_INSTALL_BINDIR}
                       -DPYMOD_INSTALL_LIBDIR=${PYMOD_INSTALL_LIBDIR}
                       -DCMAKE_INSTALL_DATADIR=${CMAKE_INSTALL_DATADIR}
                       -DCMAKE_INSTALL_INCLUDEDIR=${CMAKE_INSTALL_INCLUDEDIR}
                       -DPCMSolver_INSTALL_CMAKEDIR=${_pcmsolver_dir}
                       -DPython_EXECUTABLE=${Python_EXECUTABLE}
                       -DSTATIC_LIBRARY_ONLY=${_a_only}
                       -DSHARED_LIBRARY_ONLY=${_so_only}
                       -DENABLE_OPENMP=OFF
                       #-DENABLE_XHOST=${ENABLE_XHOST}  # option missing
                       # always fpic'd
                       -DENABLE_GENERIC=${ENABLE_GENERIC}
                       -DENABLE_TESTS=OFF
                       -DENABLE_TIMER=OFF
                       -DENABLE_LOGGER=OFF
                       -DBUILD_STANDALONE=OFF
                       -DTEST_Fortran_API=OFF
                       -DENABLE_CXX11_SUPPORT=ON
                       -DENABLE_64BIT_INTEGERS=OFF
                       -DCMAKE_OSX_DEPLOYMENT_TARGET=${CMAKE_OSX_DEPLOYMENT_TARGET}
                       -DCMAKE_OSX_SYSROOT=${CMAKE_OSX_SYSROOT}
            CMAKE_CACHE_ARGS -DCMAKE_C_FLAGS:STRING=${CMAKE_C_FLAGS}
                             -DCMAKE_CXX_FLAGS:STRING=${CMAKE_CXX_FLAGS}
                             -DCMAKE_Fortran_FLAGS:STRING=${CMAKE_Fortran_FLAGS}
                             -DCMAKE_C_COMPILER_ARG1:STRING=${CMAKE_C_COMPILER_ARG1}
                             -DCMAKE_CXX_COMPILER_ARG1:STRING=${CMAKE_CXX_COMPILER_ARG1}
                             -DCMAKE_Fortran_COMPILER_ARG1:STRING=${CMAKE_Fortran_COMPILER_ARG1})

        set(PCMSolver_DIR ${STAGED_INSTALL_PREFIX}/${_pcmsolver_dir} CACHE PATH "path to internally built PCMSolverConfig.cmake" FORCE)
    endif()
else()
    add_library(pcmsolver_external INTERFACE)  # dummy
endif()
