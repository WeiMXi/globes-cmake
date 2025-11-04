add_library(gnu STATIC
    obstack.c
    exitfail.c
)

target_include_directories(gnu PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

install(FILES
    gettext.h
    exit.h
    obstack.h
    exitfail.h
    DESTINATION include/globes/gnu
)
