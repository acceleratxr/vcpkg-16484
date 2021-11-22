This repo reproduces vcpkg issue 16484.

Steps to Repro
==============

1. Clone this repository
2. Open a **Developer PowerShell for VS 2019** prompt.
3. Run the command.
   `cmake -G "Visual Studio 16 2019"`
   
Output
======

```
The package openssl is compatible with built-in CMake targets:

    find_package(OpenSSL REQUIRED)
    target_link_libraries(main PRIVATE OpenSSL::SSL OpenSSL::Crypto)

The package poco provides CMake targets:

    find_package(Poco CONFIG REQUIRED)
    # Note: 10 target(s) were omitted.
    target_link_libraries(main PRIVATE Poco::Net Poco::XML Poco::Zip Poco::Data)

The package zlib is compatible with built-in CMake targets:

    find_package(ZLIB REQUIRED)
    target_link_libraries(main PRIVATE ZLIB::ZLIB)

CMake Error at C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/CMake/share/cmake-3.20/Modules/FindPackageHandleStandardArgs.cmake:230 (message):
  Could NOT find ZLIB (missing: ZLIB_LIBRARY ZLIB_INCLUDE_DIR)
Call Stack (most recent call first):
  C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/CMake/share/cmake-3.20/Modules/FindPackageHandleStandardArgs.cmake:594 (_FPHSA_FAILURE_MESSAGE)
  C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/CMake/share/cmake-3.20/Modules/FindZLIB.cmake:120 (FIND_PACKAGE_HANDLE_STANDARD_ARGS)
  CMakeLists.txt:86 (find_package)


-- Configuring incomplete, errors occurred!
```