cmake_minimum_required(VERSION 3.13)

project(tree-sitter-conao3_clojure
        VERSION "0.1.0"
        DESCRIPTION "Conao3Clojure grammar for tree-sitter"
        HOMEPAGE_URL "https://github.com/conao3-playground/tree-sitter-conao3-clojure"
        LANGUAGES C)

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
option(TREE_SITTER_REUSE_ALLOCATOR "Reuse the library allocator" OFF)

set(TREE_SITTER_ABI_VERSION 14 CACHE STRING "Tree-sitter ABI version")
if(NOT ${TREE_SITTER_ABI_VERSION} MATCHES "^[0-9]+$")
    unset(TREE_SITTER_ABI_VERSION CACHE)
    message(FATAL_ERROR "TREE_SITTER_ABI_VERSION must be an integer")
endif()

find_program(TREE_SITTER_CLI tree-sitter DOC "Tree-sitter CLI")

add_custom_command(OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/src/parser.c"
                   DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/grammar.json"
                   COMMAND "${TREE_SITTER_CLI}" generate src/grammar.json
                            --abi=${TREE_SITTER_ABI_VERSION}
                   WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
                   COMMENT "Generating parser.c")

add_library(tree-sitter-conao3_clojure src/parser.c)
if(EXISTS src/scanner.c)
  target_sources(tree-sitter-conao3_clojure PRIVATE src/scanner.c)
endif()
target_include_directories(tree-sitter-conao3_clojure PRIVATE src)

target_compile_definitions(tree-sitter-conao3_clojure PRIVATE
                           $<$<BOOL:${TREE_SITTER_REUSE_ALLOCATOR}>:TREE_SITTER_REUSE_ALLOCATOR>
                           $<$<CONFIG:Debug>:TREE_SITTER_DEBUG>)

set_target_properties(tree-sitter-conao3_clojure
                      PROPERTIES
                      C_STANDARD 11
                      POSITION_INDEPENDENT_CODE ON
                      SOVERSION "${TREE_SITTER_ABI_VERSION}.${PROJECT_VERSION_MAJOR}"
                      DEFINE_SYMBOL "")

configure_file(bindings/c/tree-sitter-conao3_clojure.pc.in
               "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-conao3_clojure.pc" @ONLY)

include(GNUInstallDirs)

install(FILES bindings/c/tree-sitter-conao3_clojure.h
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/tree_sitter")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/tree-sitter-conao3_clojure.pc"
        DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig")
install(TARGETS tree-sitter-conao3_clojure
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}")

add_custom_target(test "${TREE_SITTER_CLI}" test
                  WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
                  COMMENT "tree-sitter test")

# vim:ft=cmake:
