cmake_minimum_required(VERSION 3.8)

include(ExternalProject)
set(NAME "openssl_project")

EXECUTE_PROCESS( COMMAND uname -m COMMAND tr -d '\n' OUTPUT_VARIABLE ARCHITECTURE )
message( STATUS "Architecture: ${ANDROID_ABI}" )
message( STATUS "CMAKE_ANDROID_NDK: ${CMAKE_ANDROID_NDK}" )
message( STATUS "CMAKE_ANDROID_STANDALONE_TOOLCHAIN: ${CMAKE_ANDROID_STANDALONE_TOOLCHAIN}" )
message( STATUS "CMAKE_SYSROOT: ${CMAKE_SYSROOT}" )
message( STATUS "CMAKE_SYSTEM_NAME: ${CMAKE_SYSTEM_NAME}" )
message( STATUS "CMAKE_SYSTEM_VERSION: ${CMAKE_SYSTEM_VERSION}")
message( STATUS "CMAKE_ANDROID_API: ${CMAKE_ANDROID_API}" )
message( STATUS "CMAKE_ANDROID_ARCH_ABI: ${CMAKE_ANDROID_ARCH_ABI}" )
message( STATUS "CMAKE_ANDROID_ARCH: ${CMAKE_ANDROID_ARCH}" )
message( STATUS "CMAKE_ANDROID_ARM_MODE: ${CMAKE_ANDROID_ARM_MODE}" )
message( STATUS "ANDROID_NATIVE_API_LEVEL: ${ANDROID_NATIVE_API_LEVEL}" )
message( STATUS "ANDROID_PLATFORM: ${ANDROID_PLATFORM}" )
message( STATUS "CMAKE_<LANG>_COMPILER_EXTERNAL_TOOLCHAIN : ${CMAKE_COMPILER_EXTERNAL_TOOLCHAIN}" )
message( STATUS "CMAKE_ANDROID_ARCH_ABI: ${CMAKE_ANDROID_ARCH_ABI}" )
message( STATUS "CMAKE_ANDROID_NDK_TOOLCHAIN_VERSION: ${CMAKE_ANDROID_NDK_TOOLCHAIN_VERSION}" )
message( STATUS "ANDROID_TOOLCHAIN_PREFIX: ${ANDROID_TOOLCHAIN_PREFIX}" )
message( STATUS "CMAKE_<LANG>_ANDROID_TOOLCHAIN_SUFFIX: ${CMAKE_TOOLCHAIN_FILE}" )

get_cmake_property(_variableNames VARIABLES)
list (SORT _variableNames)
foreach (_variableName ${_variableNames})
    message(STATUS "${_variableName}=${${_variableName}}")
endforeach()

#set architecture with variable system_name
string(TOLOWER ${CMAKE_SYSTEM_NAME} system_name)
if(<${CMAKE_ANDROID_ARCH_ABI}|string> MATCHES "arm64")
    set(system_name "${system_name}-arm64")
elseif(<${CMAKE_ANDROID_ARCH_ABI}|string> MATCHES "arm")
    set(system_name "${system_name}-arm")
elseif(<${CMAKE_ANDROID_ARCH_ABI}|string> MATCHES "x86")
    set(system_name "${system_name}-x86")
endif()

message( STATUS "system_name: ${system_name}" )

set(build_dir "${CMAKE_CURRENT_BINARY_DIR}/${NAME}-prefix/src/${NAME}-build")
#message(STATUS "build_dir: ${build_dir}" )

set(openssl_source_dir "${CMAKE_CURRENT_BINARY_DIR}/${NAME}-prefix/src/${NAME}")
configure_file("configure_openssl.sh.in" "configure_openssl.sh" @ONLY)
configure_file("build_openssl.sh.in" "build_openssl.sh" @ONLY)
ExternalProject_Add(${NAME}
  SOURCE_DIR ${openssl_source_dir}
  GIT_REPOSITORY https://github.com/openssl/openssl
  GIT_TAG b2a00f62209add348deb8283c588ddbd572dc216
  CONFIGURE_COMMAND ${CMAKE_CURRENT_BINARY_DIR}/configure_openssl.sh
  BUILD_COMMAND ${CMAKE_CURRENT_BINARY_DIR}/build_openssl.sh
  #UPDATE_COMMAND "" # Skip annoying updates for every build
  #INSTALL_COMMAND ""
)

SET(OPENSSL_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/third_party/src/openssl_project/include)
SET(OPENSSL_LIB_DIR ${CMAKE_CURRENT_BINARY_DIR}/third_party/src/openssl_project/lib)

add_library(openssl SHARED IMPORTED)
