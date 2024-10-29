cmake_minimum_required(VERSION 3.25)

project(impeller_demo)

cmake_policy(SET CMP0135 NEW)
include(FetchContent)

# ------------------------------------------------------------------------------
# --- Setup Impeller -----------------------------------------------------------
# ------------------------------------------------------------------------------

set(IMPELLER_SDK_SHA b7c3af43f90ac9aaf66dbd10b922c25addd9c3db)
if(CMAKE_HOST_APPLE)
  set(IMPELLER_PLATFORM darwin)
elseif(CMAKE_HOST_LINUX)
  set(IMPELLER_PLATFORM linux)
elseif(CMAKE_HOST_WIN32)
  set(IMPELLER_PLATFORM windows)
else()
message(FATAL_ERROR "Unsupported platform.")
endif()

if (CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "arm64")
  set(IMPELLER_ARCH arm64)
elseif(CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "x86_64")
  set(IMPELLER_ARCH x64)
else()
message(FATAL_ERROR "Unsupported architecture.")
endif()

message(NOTICE "Impeller SDK ${IMPELLER_SDK_SHA} for ${IMPELLER_PLATFORM}-${IMPELLER_ARCH} selected.")

FetchContent_Declare(
  impeller_sdk
  URL https://storage.googleapis.com/flutter_infra_release/flutter/${IMPELLER_SDK_SHA}/${IMPELLER_PLATFORM}-${IMPELLER_ARCH}/impeller_sdk.zip
)
FetchContent_MakeAvailable(impeller_sdk)
add_library(impeller SHARED IMPORTED)
set_target_properties(impeller
  PROPERTIES
  IMPORTED_LOCATION ${impeller_sdk_SOURCE_DIR}/lib/libimpeller.dylib
)
target_include_directories(impeller INTERFACE ${impeller_sdk_SOURCE_DIR}/include)
list(APPEND CMAKE_INSTALL_RPATH ${impeller_sdk_SOURCE_DIR}/lib)
list(APPEND CMAKE_BUILD_RPATH ${impeller_sdk_SOURCE_DIR}/lib)

if(APPLE)
# The ID in the dylib may need to be tinkered with depending on how you are
# distributing the library.
# Since we setup the RPath above, we'll just try to find the library relative to
# it.
execute_process(COMMAND install_name_tool -id "@rpath/libimpeller.dylib" libimpeller.dylib
                WORKING_DIRECTORY ${impeller_sdk_SOURCE_DIR}/lib)
endif(APPLE)

# ------------------------------------------------------------------------------
# --- Setup GLFW ---------------------------------------------------------------
# ------------------------------------------------------------------------------

FetchContent_Declare(
  glfw
  URL https://github.com/glfw/glfw/archive/refs/tags/3.4.tar.gz
  EXCLUDE_FROM_ALL
)
FetchContent_MakeAvailable(glfw)

# ------------------------------------------------------------------------------
# --- Setup Project ------------------------------------------------------------
# ------------------------------------------------------------------------------

add_executable(impeller_demo demo.c)
target_link_libraries(impeller_demo glfw impeller)
