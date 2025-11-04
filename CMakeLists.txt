cmake_minimum_required(VERSION 3.12)
project(globes VERSION 3.2.18 LANGUAGES C)

# set version information
set(PACKAGE_NAME "globes")
set(PACKAGE_VERSION "${PROJECT_VERSION}")
set(VERSION "${PROJECT_VERSION}")
set(GLB_LIBVERSION "11:2:3")

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
#     set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra")
# endif()

if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w")
elseif(MSVC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /w")
endif()

# find dependencies
find_package(PkgConfig)
if(PKG_CONFIG_FOUND)
    pkg_check_modules(GSL gsl)
endif()

if(NOT GSL_FOUND)
    find_library(GSL_LIBRARIES NAMES gsl)
    find_path(GSL_INCLUDE_DIRS NAMES gsl/gsl_version.h)
    if(GSL_LIBRARIES AND GSL_INCLUDE_DIRS)
        set(GSL_FOUND TRUE)
    else()
        message(FATAL_ERROR "GSL library not found")
    endif()
endif()

# generate config.h
configure_file(
    ${CMAKE_SOURCE_DIR}/config.h.cmake
    ${CMAKE_BINARY_DIR}/config.h
    @ONLY
)

include_directories(
    ${CMAKE_BINARY_DIR} # for config.h
    ${GSL_INCLUDE_DIRS}
)

# install config script
configure_file(
    ${CMAKE_SOURCE_DIR}/globes-config.in
    ${CMAKE_BINARY_DIR}/globes-config
    @ONLY
)

add_subdirectory(lib)
add_subdirectory(source)
add_subdirectory(data)
add_subdirectory(doc)
add_subdirectory(examples)

if(EXISTS ${CMAKE_SOURCE_DIR}/globes/CMakeLists.txt)
    add_subdirectory(globes)
endif()

install(PROGRAMS ${CMAKE_BINARY_DIR}/globes-config DESTINATION bin)

# install m4 files
install(FILES globes.m4 DESTINATION share/aclocal)

# set var for globes-config
set(prefix "${CMAKE_INSTALL_PREFIX}")
set(exec_prefix "\${prefix}")
set(includedir "\${prefix}/include")
set(libdir "\${prefix}/lib")
set(pkglibdir "\${libdir}/globes")
set(VERSION "${PROJECT_VERSION}")
set(GLBLIBVERSION "${GLB_LIBVERSION}")
set(CFLAGS "-I\${includedir}")
set(LIBS "${GSL_LDFLAGS} ${GSL_LIBS} -lm")
set(wl "-Wl,")
set(hardcode_libdir_flag_spec "\${wl}-rpath \${wl}\${libdir}")

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/globes-config.in"
    "${CMAKE_CURRENT_BINARY_DIR}/globes-config"
    @ONLY
)
