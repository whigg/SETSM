cmake_minimum_required (VERSION 3.0.0)
set (GNU_VERSION 4.0.0)
set (INTEL_VERSION 14.0.0)
set (PGI_VERSION 13.0.0)
set (CRAY_VERSION 0.0.0)
set (TIFF_VERSION 4.0.0)
set (GEOTIFF_VERSION 1.4.0)

set (CC_LIST GNU INTEL PGI CRAY)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/modules")

# Define compiler IDs
set (GNU_ID "GNU")
set (INTEL_ID "Intel")
set (PGI_ID "PGI")
set (CRAY_ID "Cray")

# Compiler flags
if ($ENV{MPI} MATCHES "on")
MESSAGE("Building with MPI enabled")
set (GNU_FLAGS -O3 -march=native -ffast-math -DBUILDMPI)
set (INTEL_FLAGS -O3 -xHost  -fp-model precise -DBUILDMPI)
set (PGI_FLAGS -O3 -fast -DBUILDMPI)
set (CRAY_FLAGS -DBUILDMPI)
else()
MESSAGE("Building with MPI disabled")
set (GNU_FLAGS -O3 -march=native -ffast-math)
set (INTEL_FLAGS -O3 -xHost -fp-model precise)
set (PGI_FLAGS -O3 -fast)
set (CRAY_FLAGS )
endif()

# Other options
if (NOT DEFINED OPENMP_OPTION)
	set (OPENMP_OPTION )
endif (NOT DEFINED OPENMP_OPTION)

# Create target
project (setsm C)
MESSAGE("CMAKE_C_COMPILER_ID is ${CMAKE_C_COMPILER_ID}")
add_executable (setsm setsm_code.c voronoi_setsm.c setsmgeo.c)
set_target_properties (setsm PROPERTIES C_STANDARD 99)

# Check compiler
foreach (CPL ${CC_LIST})
	if (CMAKE_C_COMPILER_ID MATCHES ${${CPL}_ID})
		set (COMPILER ${CPL})
	endif ()
endforeach (CPL)
if (NOT DEFINED COMPILER)
	message (FATAL_ERROR "Unknown compiler ${CMAKE_C_COMPILER_ID}.")
endif (NOT DEFINED COMPILER)
if (CMAKE_C_COMPILER_VERSION LESS ${${COMPILER}_VERSION})
	message (FATAL_ERROR "${CMAKE_C_COMPILER_ID} minimum version \"${${COMPILER}_VERSION}\" required.")
endif ()

# Find tiff
find_package (TIFF ${TIFF_VERSION})
if (NOT TIFF_FOUND)
	message (FATAL_ERROR "Cannot find tiff/${TIFF_VERSION} or above.")
endif (NOT TIFF_FOUND)

# Find geotiff
find_package (GeoTIFF)# ${GEOTIFF_VERSION})
if (NOT GEOTIFF_FOUND)
	message (FATAL_ERROR "Cannot find geotiff/${GEOTIFF_VERSION} or above.")
endif (NOT GEOTIFF_FOUND)

# Find OpenMP
find_package (OpenMP)
if (NOT OPENMP_FOUND)
	message (FATAL_ERROR "Cannot find OpenMP.") 
endif (NOT OPENMP_FOUND)

# Build setsm target
target_compile_options (setsm PUBLIC ${${COMPILER}_FLAGS})
target_include_directories (setsm PUBLIC ${TIFF_INCLUDE_DIR})
target_include_directories (setsm PUBLIC ${GEOTIFF_INCLUDE_DIR})
target_link_libraries (setsm m ${TIFF_LIBRARIES})
target_link_libraries (setsm m ${GEOTIFF_LIBRARY})

list (APPEND CMAKE_C_FLAGS ${OpenMP_C_FLAGS}${OPENMP_OPTION})

# Make install
install(TARGETS setsm DESTINATION .)
