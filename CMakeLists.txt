cmake_minimum_required(VERSION 3.10)

# Project definition
project(Issue16484 VERSION 1.0.0
				   DESCRIPTION "vcpkg issue 16484 repro"
				   LANGUAGES C CXX)

# Make sure we're using C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# The version of vcpkg to checkout for our third party deps
set(VCPKG_VERSION HEAD)

set(DEPENDENCIES poco openssl zlib)

# Setup vcpkg
if(NOT DEFINED ENV{VCPKG_ROOT})
  set(VCPKG_ROOT ${PROJECT_SOURCE_DIR}/.vcpkg)
else()
  set(VCPKG_ROOT $ENV{VCPKG_ROOT})
endif()

if(NOT EXISTS ${VCPKG_ROOT}/.git)
  message("Cloning vcpkg in ${VCPKG_ROOT}")
  execute_process(COMMAND git clone -n https://github.com/Microsoft/vcpkg.git ${VCPKG_ROOT})
endif()

# The following command has no effect if the vcpkg repository is in a detached head state.
message("Checking out vcpkg version ${VCPKG_VERSION} in ${VCPKG_ROOT}")
#execute_process(COMMAND git checkout -f ${VCPKG_VERSION} WORKING_DIRECTORY ${VCPKG_ROOT})
execute_process(COMMAND git checkout -f ${VCPKG_VERSION} WORKING_DIRECTORY ${VCPKG_ROOT})

if(NOT EXISTS ${VCPKG_ROOT}/README.md)
  message(FATAL_ERROR "***** FATAL ERROR: Could not clone vcpkg *****")
endif()

if(WIN32)
  set(VCPKG_EXEC ${VCPKG_ROOT}/vcpkg.exe)
  set(VCPKG_BOOTSTRAP ${VCPKG_ROOT}/bootstrap-vcpkg.bat)
else()
  set(VCPKG_EXEC ${VCPKG_ROOT}/vcpkg)
  set(VCPKG_BOOTSTRAP ${VCPKG_ROOT}/bootstrap-vcpkg.sh)
endif()

if(NOT EXISTS ${VCPKG_EXEC})
  message("Bootstrapping vcpkg in ${VCPKG_ROOT}")
  execute_process(COMMAND ${VCPKG_BOOTSTRAP} WORKING_DIRECTORY ${VCPKG_ROOT})
endif()

if(NOT EXISTS ${VCPKG_EXEC})
  message(FATAL_ERROR "***** FATAL ERROR: Could not bootstrap vcpkg *****")
endif()

set(CMAKE_TOOLCHAIN_FILE ${VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake CACHE STRING "")
 
# Determine what triplet to use
if (WIN32)
  if (CMAKE_GENERATOR MATCHES ".*Win64" OR CMAKE_GENERATOR_PLATFORM MATCHES "x64")
    set(ARCH x64)
  else()
    set(ARCH x86)
  endif()
  set(TRIPLET ${ARCH}-windows-static)

  if (CMAKE_GENERATOR MATCHES ".*2017.*")
    set(TOOLCHAIN v141)
  else()
    set(TOOLCHAIN v142)
  endif()
elseif (UNIX AND NOT APPLE)
  # Only 64-bit is supported
  set(ARCH x64)
  set(TRIPLET x64-linux)
else()
  message(FATAL_ERROR "***** FATAL ERROR: Unsupported generator specified. *****")
endif()

# Set the following now so that vcpkg doesn't fail (see https://github.com/microsoft/vcpkg/issues/13614)
set(VCPKG_TARGET_TRIPLET ${TRIPLET})

message(STATUS "***** Checking project third party dependencies in ${VCPKG_ROOT}")
execute_process(COMMAND ${VCPKG_EXEC} install ${DEPENDENCIES} --recurse --overlay-triplets=${PROJECT_SOURCE_DIR}/triplets/${TOOLCHAIN} --triplet ${TRIPLET} WORKING_DIRECTORY ${VCPKG_ROOT})

# Load the required dependencies
find_package(ZLIB REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Poco CONFIG REQUIRED COMPONENTS Foundation JSON NetSSL Util)

string(REGEX REPLACE "(^.*/|\\.[^.]*$)" "" file_without_ext src/main.cpp)
add_executable(${file_without_ext} src/main.cpp)