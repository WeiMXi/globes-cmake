find_package(Perl REQUIRED)

execute_process(
    COMMAND ${PERL_EXECUTABLE} -e "use Config; print \$Config{archlib} . '/CORE'"
    OUTPUT_VARIABLE PERL_CORE_DIR
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

message(STATUS "Using Perl core directory: ${PERL_CORE_DIR}")

add_library(globes_perl MODULE
    globes_perl.c
)

target_compile_definitions(globes_perl PRIVATE
    _GNU_SOURCE
    _FILE_OFFSET_BITS=64
    _LARGEFILE64_SOURCE
)

execute_process(
    COMMAND ${PERL_EXECUTABLE} -MConfig -e "print \$Config{ccflags}"
    OUTPUT_VARIABLE PERL_CCFLAGS
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

target_compile_options(globes_perl PRIVATE 
    ${PERL_CCFLAGS}
)

target_include_directories(globes_perl PRIVATE
    ${PERL_CORE_DIR}
    ${CMAKE_SOURCE_DIR}/source
    ${CMAKE_BINARY_DIR}
    ${CMAKE_SOURCE_DIR}
)

target_link_libraries(globes_perl globeslib)

set_target_properties(globes_perl PROPERTIES
    PREFIX ""
    SUFFIX ".so"
)

install(TARGETS globes_perl DESTINATION lib/globes/perl)
install(FILES globes.pm DESTINATION lib/globes/perl)

install(CODE "
    execute_process(
        COMMAND \${CMAKE_COMMAND} -E create_symlink
        globes_perl.so
        globes.so
        WORKING_DIRECTORY \${CMAKE_INSTALL_PREFIX}/lib/globes/perl
    )
")

install(FILES 
    globes.h
    glb-modules.h
    DESTINATION include/globes
)