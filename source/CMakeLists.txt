find_program(YACC_EXECUTABLE 
    NAMES yacc bison byacc
    DOC "Yacc compatible parser generator"
)

find_program(LEX_EXECUTABLE
    NAMES lex flex
    DOC "Lex compatible scanner generator"
)

if(NOT YACC_EXECUTABLE)
    message(FATAL_ERROR "yacc/bison not found! Please install yacc or bison.")
endif()

if(NOT LEX_EXECUTABLE)
    message(FATAL_ERROR "lex/flex not found! Please install lex or flex.")
endif()


set(GLBPARSER_SOURCES
    glb_minimize.c
    glb_error.c
    glb_path.c
    glb_multiex.c
    glb_version.c
    glb_rate_engine.c
    glb_min_sup.c
    glb_fluxes.c
    glb_smear.c
    glb_probability.c
    glb_profile.c
    glb_sys.c
    glb_wrapper.c
    glb_glb.c
    glb_prior.c
    glb_compat.c
    glb_lexer.c
    glb_parser.c
)

# 生成解析器文件
if(NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/glb_parser.c)
    message(STATUS "Generating parser files...")
    execute_process(
        COMMAND ${YACC_EXECUTABLE} -d -o ${CMAKE_CURRENT_BINARY_DIR}/glb_parser.c ${CMAKE_CURRENT_SOURCE_DIR}/glb_parser.y
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE yacc_result
    )
    if(NOT yacc_result EQUAL 0)
        message(FATAL_ERROR "Failed to generate parser with yacc")
    endif()
endif()

if(NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/glb_lexer.c)
    message(STATUS "Generating lexer files...")
    execute_process(
        COMMAND ${LEX_EXECUTABLE} -o${CMAKE_CURRENT_BINARY_DIR}/glb_lexer.c ${CMAKE_CURRENT_SOURCE_DIR}/glb_lexer.l
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE lex_result
    )
    if(NOT lex_result EQUAL 0)
        message(FATAL_ERROR "Failed to generate lexer with lex")
    endif()
endif()

# main lib
add_library(globeslib SHARED
    ${GLBPARSER_SOURCES}
    ${PARSER_SOURCES}
)

# add include for globeslib
target_include_directories(globeslib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(globeslib PUBLIC gnu ${GSL_LIBRARIES})
target_compile_definitions(globeslib PRIVATE 
    GLB_MODULE_DIR="${CMAKE_INSTALL_PREFIX}/lib/globes"
)

set_target_properties(globeslib PROPERTIES
    VERSION 3.2.18
    SOVERSION 0
)

# executable globes
add_executable(globes globes.c)
target_link_libraries(globes globeslib)

add_executable(globes-accuracy
    globes-accuracy.c
    glb_tools_parser.c
    glb_tools_header.c
    glb_tools_eightfold.c
)
target_link_libraries(globes-accuracy globeslib)

add_executable(globes-deltatheta
    globes-deltatheta.c
    glb_tools_parser.c
    glb_tools_header.c
)
target_link_libraries(globes-deltatheta globeslib)

add_executable(globes-discovery
    globes-discovery.c
    glb_tools_parser.c
    glb_tools_header.c
    glb_tools_eightfold.c
)
target_link_libraries(globes-discovery globeslib)

add_executable(globes-simple
    globes-simple.c
    glb_tools_parser.c
    glb_tools_header.c
    glb_tools_eightfold.c
)
target_link_libraries(globes-simple globeslib)

# install
install(TARGETS globeslib
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
)

install(TARGETS 
    globes globes-accuracy globes-deltatheta globes-discovery globes-simple
    DESTINATION bin
)

file(GLOB HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/*.h")
install(FILES ${HEADER_FILES}
    DESTINATION include/
)

# define macro
target_compile_definitions(globeslib PRIVATE 
    VERSION="${PROJECT_VERSION}"
    GLB_LIBVERSION="${GLB_LIBVERSION}"
    PACKAGE_NAME="${PACKAGE_NAME}" 
    PACKAGE_VERSION="${PACKAGE_VERSION}"
    GLB_MODULE_DIR="${CMAKE_INSTALL_PREFIX}/lib/globes"
    HAVE_CONFIG_H=1
)

